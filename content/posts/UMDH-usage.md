+++
date = '2025-12-15T15:33:10+08:00'
draft = false
title = 'UMDH Usage'
+++
# UMDH（User Mode Dump Heap）内存泄露排查笔记

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

- **分配次数的变化**
- **同一调用栈的累计增长**
- 而不是“某一块具体地址的内存”

------

## 前置条件：开启 UST

UMDH 要显示分配调用栈，必须开启 **UST（User Stack Trace Database）**。

### 使用 gflags 开启 UST

```

gflags.exe -i LogisticsPlatformApp.exe +ust
```

说明：

- `-i LogisticsPlatformApp.exe`
   针对指定进程名生效
- `+ust`
   启用用户态堆分配的调用栈记录
- 修改的是注册表，需要 **重新启动进程**

注意事项：

- 会增加一定内存和性能开销
- 只建议在测试 / 调试环境启用
- 关闭方式：

```

gflags.exe -i LogisticsPlatformApp.exe -ust
```

------

## UMDH 的基本用法

### 1. 获取进程 PID

可以通过：

- 任务管理器
- `tasklist`
- Process Explorer

------

### 2. 生成第一次堆快照

示例：

```

umdh -p:12345 -f:snap1.txt
```

------

### 3. 生成第二次堆快照

在程序运行一段时间后：

```

umdh -p:12345 -f:snap2.txt
```

------

### 4. 对比两个快照

```

umdh snap1.txt snap2.txt > diff.txt
```

`diff.txt` 即为分析结果。

------

## UMDH 输出内容说明

UMDH diff 中常见的条目格式如下：

```

+   3c270 ( 3d624 -  13b4)   10f6 allocs BackTrace4BD0
+    10a1 (  10f6 -    55)   BackTrace4BD0 allocations
```

字段含义：

- `+ 3c270`
   净增长内存（十六进制，单位字节）
- `(3d624 - 13b4)`
   后一次快照 - 前一次快照
- `10f6 allocs`
   分配次数
- `10a1`
   分配次数减去释放次数
- `BackTrace4BD0`
   调用栈 ID

一般关注点：

- **allocs 是否持续增加**
- **allocations（净增长）是否为正数**
- 同一 `BackTraceXXX` 是否反复出现

------

## 关于分配大小的理解

UMDH 中看到的大小通常是：

- **请求大小（requested size）**
- 不包含堆管理开销

例如：

```

34 bytes + 2C at 1524B8BBCD0 by BackTrace4BD0
```

含义：

- 请求了 `0x34`（52）字节
- 堆管理额外开销 `0x2C`（44）字节
- 实际堆占用约 96 字节
- 分配来源为 `BackTrace4BD0`

在 LFH 下，常见现象是：

- 请求大小：`0x34`
- 堆统计中看到：`0x40`（64 字节 bucket）

这是正常的对齐行为。

------

## 符号（PDB）说明

- **UMDH 本身不强制依赖 PDB**
- 没有 PDB 也可以看到模块名和函数名（系统模块）
- 有 PDB 可以：
  - 显示更完整的函数名
  - 定位到自定义模块中的具体函数

建议：

- 系统模块：使用微软符号服务器即可
- 自己的程序：尽量保留 PDB

------

## 常见注意事项

- UMDH 适合 **趋势分析**，不适合看“单个对象”
- LFH (Low Fragmentation Heap) 场景下，`!heap -flt` 往往不可用
- 十六进制数值需要注意换算
- 关注 **分配次数**，而不只是总字节数