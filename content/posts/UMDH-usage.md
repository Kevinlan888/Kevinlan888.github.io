+++
date = '2025-12-15T15:33:10+08:00'
draft = false
title = 'UMDH（User Mode Dump Heap）内存泄露排查笔记'
tags = ['Debug']
+++

## 简介

**UMDH（User Mode Dump Heap）** 是 Windows 提供的一个用户态内存分析工具，主要用于：

- 对比两个时间点的堆快照
- 找出**哪些调用栈的内存分配在增长**
- 常用于排查 **内存泄露 / 长时间内存增长**

UMDH 适合的场景包括：

- 小块内存（几十字节）持续增长
- 普通 dump 中 `!heap -stat` 无法定位来源

------

## 基本原理

UMDH 并不分析单次 dump，而是：

1. 在某一时刻生成 **堆快照 A**
2. 程序继续运行
3. 再生成 **堆快照 B**
4. 对比 A 和 B，输出 **内存增长的调用栈**

因此，UMDH 的关注点是：

- **存活分配数的变化**
- **同一调用栈的累计增长**
- 而不是“某一块具体地址的内存”

------

## 前置条件

UMDH 随 **Debugging Tools for Windows**（Windows SDK / WDK）一起安装，gflags 也在同一目录下。

- umdh 的位数需要与目标进程一致（x64 进程使用 x64 版本）
- 一般需要以**管理员权限**运行

### 1. 开启 UST

UMDH 要显示分配调用栈，必须开启 **UST（User Stack Trace Database）**。

```cmd
gflags.exe -i YourApp.exe +ust
```

说明：

- `-i YourApp.exe`：针对指定进程名生效
- `+ust`：启用用户态堆分配的调用栈记录
- 修改的是注册表，需要 **重新启动进程**

注意事项：

- 会增加一定内存和性能开销
- 只建议在测试 / 调试环境启用
- 关闭方式：

```cmd
gflags.exe -i YourApp.exe -ust
```

### 2. 配置符号路径

UMDH 依靠符号把调用栈地址解析成函数名。没有配置符号时，diff 结果里只有地址，基本无法分析。

```cmd
set _NT_SYMBOL_PATH=srv*C:\symbols*https://msdl.microsoft.com/download/symbols;D:\YourApp\pdb
```

- `srv*C:\symbols*...`：从微软符号服务器下载系统模块符号并缓存到本地
- `D:\YourApp\pdb`：自己程序的 pdb 所在目录，需要与运行的二进制版本匹配

生成快照和对比快照时都需要设置该环境变量。

------

## UMDH 的基本用法

### 1. 确定目标进程

可以通过任务管理器、`tasklist` 或 Process Explorer 获取 PID，也可以直接使用进程名（`-pn:`）。

------

### 2. 生成第一次堆快照

```cmd
umdh -p:12345 -f:snap1.txt
```

或按进程名：

```cmd
umdh -pn:YourApp.exe -f:snap1.txt
```

------

### 3. 生成第二次堆快照

在程序运行一段时间后：

```cmd
umdh -p:12345 -f:snap2.txt
```

------

### 4. 对比两个快照

```cmd
umdh snap1.txt snap2.txt > diff.txt
```

`diff.txt` 即为分析结果。如果不想手动换算十六进制，可以加 `-d` 以十进制输出：

```cmd
umdh -d snap1.txt snap2.txt > diff.txt
```

------

## UMDH 输出内容说明

UMDH diff 中每个调用栈通常对应两行：

```text
+   3c270 ( 3d624 -  13b4)   10f6 allocs BackTrace4BD0
+    10a1 (  10f6 -    55)   BackTrace4BD0 allocations
```

第一行描述的是**字节数**：

- `3d624`：第二次快照中，该调用栈分配且仍未释放的总字节数
- `13b4`：第一次快照中对应的总字节数
- `+ 3c270`：两者之差，即净增长字节数（十六进制）
- `10f6 allocs`：第二次快照中，该调用栈仍存活的分配个数

第二行描述的是**分配个数**：

- `10f6`：第二次快照中的存活分配个数
- `55`：第一次快照中的存活分配个数
- `+ 10a1`：两者之差，即存活分配个数的净增长

- `BackTrace4BD0`：调用栈 ID，diff 中紧跟其后的就是解析出的调用栈

需要注意：UMDH 只比较两个时间点的**存活**分配，并不统计中间发生了多少次分配和释放。

一般关注点：

- **存活分配个数（第二行）是否为正并持续增长**
- 多次抓取快照时，同一 `BackTraceXXX` 是否反复出现在增长列表前列

------

## 关于分配大小的理解

UMDH diff 中统计的大小是：

- **请求大小（requested size）**
- 不包含堆管理开销

在单个快照文件中，可以看到每一块分配的明细，例如：

```text
34 bytes + 2C at 1524B8BBCD0 by BackTrace4BD0
```

含义：

- 请求了 `0x34`（52）字节
- 额外占用 `0x2C`（44）字节（块头 + 对齐产生的未使用字节）
- 该块在堆上共占 `0x60`（96）字节
- 分配来源为 `BackTrace4BD0`

由于堆会按粒度（x64 下为 16 字节）对齐，并且每个块带有块头，`!heap` 等工具中看到的块大小一般会大于请求大小。如需确认某一块的实际情况，可以在 WinDbg 中查看：

```text
!heap -p -a 1524B8BBCD0
```

------

## 常见注意事项

- UMDH 适合 **趋势分析**，不适合看“单个对象”
- 没有正确的符号，调用栈无法解析
- 十六进制数值需要注意换算，或使用 `-d` 参数
- 关注 **存活分配个数**，而不只是总字节数
