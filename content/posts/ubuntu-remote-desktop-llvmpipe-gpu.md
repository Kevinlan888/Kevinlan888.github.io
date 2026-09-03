+++
date = '2026-09-03T09:00:00+08:00'
draft = false
title = 'Ubuntu 24.04 远程桌面使用 llvmpipe 的排查与修复'
tags = ['Linux', 'Ubuntu', 'Remote Desktop', 'Debug']
+++

> 远程连接 Ubuntu 后，如果发现桌面使用的是 llvmpipe 而不是显卡，CPU 占用通常会明显升高。本文记录一次从现象、日志到最终修复的完整排查过程。

## 一、问题现象

通过远程桌面连接 Ubuntu 24.04 后，执行下面的命令检查 OpenGL：

```bash
DISPLAY=:0 glxinfo -B
```

输出中出现：

```text
Accelerated: no
OpenGL renderer string: llvmpipe (LLVM 20.1.2, 256 bits)
```

`llvmpipe` 是 Mesa 提供的软件渲染器，主要依赖 CPU 完成 OpenGL 运算。当 GNOME Shell、浏览器或其他图形程序频繁刷新画面时，CPU 占用就会明显升高。

## 二、先确认显卡驱动是否正常

首先检查显卡和内核驱动：

```bash
lspci -nnk | grep -A4 -B2 -E 'VGA|3D|Display'
```

本文案例中的显卡是 Intel UHD 630，内核驱动为 `i915`。系统中还存在 Intel Mesa 驱动文件：

```text
iris_dri.so
```

这说明显卡驱动和 Mesa 并没有缺失，问题更可能发生在远程图形会话层面。

## 三、确认当前使用的是哪一种远程桌面

Ubuntu 上可能同时安装 GNOME Remote Desktop 和 xrdp，但“安装了 xrdp”并不代表当前连接一定使用 xrdp。

检查 GNOME Remote Desktop：

```bash
grdctl status
systemctl --user status gnome-remote-desktop
```

如果看到：

```text
RDP:
    Status: enabled
    Port: 3389
```

并且服务处于运行状态，就说明当前连接使用的是 GNOME Remote Desktop。

继续检查登录会话：

```bash
loginctl list-sessions
loginctl show-session <会话编号> \
  -p User -p Type -p Remote -p Seat -p State
```

典型的无头远程会话可能显示：

```text
User=1000
Type=wayland
Remote=yes
Seat=
State=active
```

这说明当前是一个远程 Wayland 会话，而不是连接到本地物理显示器上的桌面会话。GNOME Remote Desktop 同时支持共享现有桌面和无头远程登录两种模式，二者的 GPU 设备访问方式并不完全相同。[GNOME Remote Desktop 官方 README](https://github.com/GNOME/gnome-remote-desktop/blob/main/README.md)

## 四、从日志定位根因

检查 GNOME Shell 启动时的 GPU 相关日志：

```bash
journalctl -b _COMM=gnome-shell --no-pager \
  | grep -Ei 'renderD|card[0-9]|renderer|llvmpipe|surfaceless|permission denied'
```

如果看到类似内容：

```text
failed to open /dev/dri/renderD128: Permission denied
failed to open /dev/dri/card1: Permission denied
Created surfaceless renderer without GPU
```

就可以确定：GNOME Shell 无法访问 DRM 设备，因此退回到了没有 GPU 的 surfaceless 软件渲染。

查看设备权限：

```bash
ls -la /dev/dri
getfacl /dev/dri/renderD128 /dev/dri/card1
```

通常可以看到：

```text
/dev/dri/renderD128  group: render
/dev/dri/card1       group: video
```

如果当前用户不在这些组中，或者正在运行的 GNOME Shell 进程没有继承这些组，就会出现上述问题。

## 五、修复 GPU 设备访问权限

如果用户还没有加入相关组，可以执行：

```bash
sudo usermod -aG render,video <用户名>
```

例如：

```bash
sudo usermod -aG render,video kevin
```

检查组配置：

```bash
getent group render
getent group video
```

需要特别注意：加入用户组只会影响新创建的登录会话。已经运行的 GNOME Shell 和远程桌面服务不会自动获得新的 supplementary groups。

因此，修改组配置后必须彻底退出当前图形会话，然后重新登录。

如果有独立的 SSH 或 TTY 会话，可以先查看会话编号：

```bash
loginctl list-sessions
```

确认目标会话后终止：

```bash
sudo loginctl terminate-session <会话编号>
```

执行前请保存所有工作，因为该命令会断开当前远程桌面。如果远程桌面是唯一的控制方式，可以在维护窗口重启电脑。

## 六、重新登录后验证

重新连接远程桌面后，先确认当前会话已经继承了用户组：

```bash
id
```

输出中应包含：

```text
video
render
```

然后检查 OpenGL：

```bash
DISPLAY=:0 glxinfo -B
```

正确结果应类似：

```text
Accelerated: yes
OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) UHD Graphics 630 (CFL GT2)
```

也可以检查 Vulkan：

```bash
vulkaninfo --summary | grep -E 'deviceName|driverName|driverInfo'
```

预期看到：

```text
deviceName = Intel(R) UHD Graphics 630 (CFL GT2)
driverName = Intel open-source Mesa driver
```

如果 Vulkan 输出中仍然列出了 llvmpipe 作为第二个设备，不一定是问题。只要 Intel GPU 是主要设备，并且 OpenGL 显示 `Accelerated: yes`，就说明硬件渲染已经生效。

## 七、几个常见误区

### 1. 设置 `MESA_LOADER_DRIVER_OVERRIDE=iris`

```bash
export MESA_LOADER_DRIVER_OVERRIDE=iris
```

这个变量只能选择 Mesa 驱动，不能解决没有权限访问 `/dev/dri/renderD128` 的问题。

### 2. 设置 `LIBGL_ALWAYS_SOFTWARE=0`

```bash
export LIBGL_ALWAYS_SOFTWARE=0
```

这也不能强制 GNOME Shell 使用 GPU。真正的问题是 GPU 设备访问权限，以及会话启动时的渲染器选择。

### 3. 只重启 GNOME Remote Desktop

只重启远程桌面服务通常不够，因为已经运行的 GNOME Shell、Mutter 和 Xwayland 可能在启动时就选择了 llvmpipe。最可靠的方法是完整退出并重新建立图形会话。

## 八、GNOME Remote Desktop 与 xrdp

如果 3389 端口已经由 GNOME Remote Desktop 占用，那么修改 xrdp 配置不会影响当前连接。

xrdp 使用独立的 Xorg 会话，通常需要配合 `xorgxrdp` 和 Glamor/DRI3 才能获得更好的图形性能。[xorgxrdp 官方说明](https://github.com/neutrinolabs/xorgxrdp)

如果希望连接本地已经登录的桌面，可以考虑 GNOME 的 Desktop Sharing 模式；如果希望创建独立的远程桌面会话，则可以单独配置 xrdp，并使用不同端口。

## 九、总结

远程桌面出现 llvmpipe，并不一定意味着显卡驱动损坏。更常见的原因是：

1. 远程会话是无头 Wayland 会话；
2. GNOME Shell 没有权限访问 `/dev/dri/renderD128` 或 `/dev/dri/card1`；
3. 用户组配置已经修改，但旧的图形会话没有重新加载；
4. Mutter 启动时因此选择了软件渲染。

解决的关键是让用户拥有正确的 GPU 设备权限，并彻底重新登录图形会话。最终应重点确认：

```text
Accelerated: yes
OpenGL renderer string: Mesa Intel(R) UHD Graphics 630
```

另外，GPU 图形渲染和 RDP 视频编码是两个不同环节。即使 OpenGL 已经使用 GPU，远程桌面的图像编码仍可能使用部分 CPU，具体取决于远程桌面服务、客户端和编码器支持情况。
