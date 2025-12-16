+++
date = '2025-12-16T14:45:35+08:00'
draft = true
title = '一次 WinDbg + P/Invoke 内存暴涨问题的排查'
+++
## 背景

在运行 **LogisticsPlatformApp.exe** 的过程中，发现进程内存**持续增长**：

- 启动时约 **300MB**
- 运行一段时间后增长到 **500MB**
- GC 正常触发，但内存无法回落

这类问题第一反应是 **native / PInvoke / 非托管内存泄漏**，于是开始使用 **WinDbg** 进行系统级排查。

------

## 一、开启 Heap 分配调用栈（UST）

第一步是让系统记录 **Heap 分配调用栈**，否则后续很难定位来源。

```

gflags.exe -i LogisticsPlatformApp.exe +ust
```

说明：

- `+ust`：User Stack Trace
- 作用：记录每一次 Heap 分配的调用栈
- ⚠️ **必须在进程启动前设置**

设置完成后，重新启动程序并复现内存增长问题。

------

## 二、定位占用最大的 Heap

进入 WinDbg，附加进程后执行：

```

!heap -stat
```

输出中可以看到多个 Heap，这里重点关注 **Commit / Busy 最大的 Heap**，例如：

```

0000029955a70000  ...  Commit 190264k
```

👉 记录下这个 Heap 地址：

```

Heap = 0000029955a70000
```

------

## 三、分析 Heap 内存分布

对目标 Heap 进行统计分析：

```

!heap -stat -h 0000029955a70000
```

输出中出现了非常关键的一行：

```

size     #blocks     total     (%)
40       9a731       269cc40   (29.33)
```

含义解释：

- `40` → **0x40 = 64 bytes**
- `9a731` → 实际统计后约 **60 多万个 block**
- 占整个 Heap **接近 30%**

📌 **结论**：

> Heap 中存在大量 **64 字节的小对象**，这是异常信号。

------

## 四、过滤指定大小的 Heap Block

为了进一步确认这些 64 byte 对象的来源，对 Heap 进行 size 过滤：

```

!heap -flt s 0x40 0000029955a70000
```

（部分 WinDbg 版本中 size 需要略微对齐，实际可用 `0x4e`）

这一步的目的：

- 只列出 **大小为 64 byte** 的分配
- 验证这些 block 是否来自同一类分配行为

#### 补充说明：`!heap -flt` 可能报错的问题

在实际排查过程中，需要特别注意一点：
 **`!heap -flt` 在某些 WinDbg 版本 / 进程状态下可能会直接报错**。

例如执行：

```

!heap -flt s 0x40 0000029955a70000
```

可能会出现类似错误提示（或直接无输出）。

### 原因

这是因为：

- 新版本 WinDbg 中，Heap 扩展已经逐步迁移到 **ext.heap**
- 有些 heap 分析命令在当前会话中 **默认未启用 page heap / heap parsing 支持**
- 导致 `!heap -flt` 无法正常工作

------

## 正确做法：先初始化 heap 扩展

在执行 `!heap -flt` 之前，**先执行一次**：

```

!ext.heap -p
```

作用：

- 初始化 heap 解析上下文
- 启用对 NT Heap / Segment Heap 的完整支持
- 之后 `!heap -flt` 才能正常过滤

然后再执行：

```

!heap -flt s 0x40 0000029955a70000
```

即可正确列出 **指定 Heap 中所有大小为 64 byte 的 block**。

------

## 五、查看单个对象的分配调用栈

随便选取一个 64 byte block 的地址，执行：

```

!heap -p -a <address>
```

得到关键调用栈：

```

ntdll!RtlpAllocateHeapInternal+0xa7d
clr!FieldMarshaler_StringAnsi::UpdateNativeImpl
clr!LayoutUpdateNative
clr!FmtClassUpdateNative
clr!MarshalNative::StructureToPtr
```

📌 **这条调用栈信息量非常大：**

- 分配来自 **CLR**
- 与 **P/Invoke / Marshal.StructureToPtr**
- 明确指向 **ANSI string marshaling**

------

## 六、回到代码：问题根源

对应的 C 结构体定义如下：

```

typedef struct tagVideoRenderText
{
    char* text;
    uint32_t textLen;
    VR_Point2D_S textPos;
    uint32_t fontSize;
    uint32_t fontColor;
} VR_Text_S;
```

而 C# 侧最初的 P/Invoke 定义是：

```

[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi)]
public struct VR_Text_S
{
    [MarshalAs(UnmanagedType.LPStr)]
    public string text;
    public uint textLen;
    public VR_Point2D_S textPos;
    public uint fontSize;
    public uint fontColor;
}
```

### ⚠️ 关键误区在这里

- native 中的 `char*`
  - **只是一个指针**
  - 不代表定长数组
  - 生命周期由调用方 / 被调用方约定
- C# 中使用 `string + LPStr`
  - CLR 会 **每次调用 StructureToPtr 时**
    - 自动 `AllocHGlobal`
    - 拷贝字符串
    - 分配一个 **小的 unmanaged buffer**
  - **该 buffer 并不会随着结构体释放自动回收**

📌 **结果**：

- 每次调用都会产生一个 ~64 byte 的 unmanaged 分配
- 频繁调用后 → **60 多万个 64 byte LFH block**
- 即使 `Marshal.FreeHGlobal` 释放了外层结构体
- **字符串对应的内存仍然驻留在 LFH 中**

------

## 七、正确的 P/Invoke 写法

对于 `char*`，**正确的 C# 映射方式是 `IntPtr`**：

```

[StructLayout(LayoutKind.Sequential)]
public struct VR_Text_S
{
    public IntPtr text;   // char*
    public uint textLen;
    public VR_Point2D_S textPos;
    public uint fontSize;
    public uint fontColor;
}
```

使用时显式管理内存：

```

vr.text = Marshal.StringToHGlobalAnsi(str);
vr.textLen = (uint)str.Length;

// 调用 native 方法

Marshal.FreeHGlobal(vr.text);
```

✔ 内存可控
 ✔ 生命周期清晰
 ✔ 不再产生 LFH 小对象堆积

------

## 八、结论

这次问题的本质是：

> **将 native 的 `char*` 错误地当成了“托管 string”使用**

而实际上：

- `char*` ≠ `char[]`
- `char*` 在 P/Invoke 中 **永远不应该直接用 `string`**
- 否则就会引入 **隐式 unmanaged 分配 + LFH 堆积**

------

## 九、经验总结

1. **Heap 内存暴涨，优先 WinDbg**
2. `gflags +ust` 是排查 native 泄漏的前提
3. `!heap -stat` → 找最大 Heap
4. `!heap -flt s` → 锁定异常 size
5. `!heap -p -a` → 看调用栈
6. **看到 `FieldMarshaler_StringAnsi`，第一时间回查 P/Invoke 定义**