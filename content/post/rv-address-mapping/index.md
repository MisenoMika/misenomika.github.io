---
title: "Rv Address Mapping"
description: 
date: 2026-06-17T16:21:17+08:00
image: 
math: 
license: 
comments: true
draft: false
categories:
    - Risc-V
build:
    list: always    # Change to "never" to hide the page from the list
---

# RISC-V RV32 的物理地址空间与内存映射

初次接触PA实验RTFSC时，我一直很疑惑为什么客户内存是 `0x80000000` 开始的。
因此和gpt老师探讨了一下，讨论内容也顺便记录下来，权当个学习笔记

我们都知道
CPU 访问的只是一个地址：

```c
*(uint32_t *)addr
```

至于这个地址最终对应：

* RAM
* 串口
* 定时器
* 显存
* 不存在的区域

CPU 本身并不知道。

真正的处理方式由地址译码（Address Decode）决定。

可以抽象成：

```c
if (addr 在 RAM 范围内)
    访问内存
else if (addr 在设备范围内)
    访问设备
else
    非法访问
```

---

## NEMU 中的地址布局

在 NEMU 的默认配置下，我们设置成：

```c
MBASE = 0x80000000
```

这意味着客户的物理内存（pmem）从 `0x80000000` 开始。

典型布局如下：

```text
            高地址
┌──────────────────────────┐
│      MMIO 设备区域       │
├──────────────────────────┤
│                          │
│        RAM (pmem)        │
│                          │
└──────────────────────────┘
            低地址
```

即

```text
0xFFFFFFFF
    │
    │  MMIO
    │
0x90000000
    │
    │
0x80000000 ────────────────
    │
    │  DRAM / pmem
    │
    │  程序代码段
    │  全局数据
    │  堆(heap)
    │  栈(stack)
    │
0x80000000 + CONFIG_MSIZE
```

程序真正运行的地方是这一段 RAM。

---

## 什么是 MMIO

MMIO（Memory-Mapped I/O）即：

> 将设备映射到地址空间中。

这样 CPU 不需要额外指令访问设备。

访问设备和访问内存的写法完全一致：

```c
*(volatile uint32_t *)0x10000000 = 'A';
```

看起来是在写内存。

实际上可能是在向串口发送字符。

---

## Hole：未实现区域

地址空间中并不是所有地址都有意义。

例如：

```text
0x00000000 ~ 0x7FFFFFFF
```

可能没有对应硬件。

或者：

```text
0x90000000 ~ 0xFFFFFFFF
```

部分区域没有映射设备。

这些区域称为：

```text
Hole
```

即地址空间中的“空洞”。

访问这些区域通常会导致：

* 总线错误（Bus Error）
* 异常（Exception）
* Simulator Assert
* Undefined Behavior

在 NEMU 中常表现为：

```text
invalid memory access
```

或者直接触发：

```c
assert(0);
```

---

## NEMU 是如何实现的

在 NEMU 中，物理内存是一个数组：

```c
uint8_t pmem[CONFIG_MSIZE];
```

例如：

```text
物理地址
0x80000000
      ↓
pmem[0]

0x80000001
      ↓
pmem[1]
```

访问内存时：

```c
addr = 0x80000010
```

最终会转换成：

```c
pmem[0x10]
```

---

而 MMIO 则采用另一套机制。

例如：

```c
map(0x10000000, uart_handler);
map(0x10001000, timer_handler);
```

当 CPU 访问这些地址时：

```c
paddr_read(addr)
```

不会访问 `pmem[]`。

而是直接调用设备处理函数：

```c
uart_handler(...)
timer_handler(...)
```

从而实现设备模拟。

---

## 从 CPU 的角度看世界

对于 CPU 而言：

```text
地址 → 数据
```

仅此而已。

CPU 并不关心：

* 这是内存
* 这是串口
* 这是键盘
* 这是显存

这些都由硬件平台负责解释。

因此：

```c
lw x1, 0(x2)
```

既可能是在读取 RAM：

```text
x2 = 0x80000000
```

也可能是在读取定时器：

```text
x2 = 0x10001000
```

指令完全相同。

差别仅在于地址不同。

---

## 为什么这样设计

RISC-V 以及大多数现代嵌入式系统采用统一地址空间（Unified Address Space）设计。

优点：

### 1. CPU 更简单

不需要专门设计：

```text
IN
OUT
PORT
```

等 I/O 指令。

普通的：

```text
Load / Store
```

即可访问设备。

---

### 2. 软件更统一

访问设备：

```c
*(volatile uint32_t *)UART_ADDR
```

访问内存：

```c
*(uint32_t *)ptr
```

编程模型完全一致。

---

### 3. 易于扩展

新增设备时只需要：

```text
给设备分配一个地址范围
```

即可接入系统。

---

## 与后续学习内容的联系

学习 PA 时，后面会不断遇到三个核心概念：

## MMIO

不同地址访问不同设备。

---

## ELF 加载

不同地址对应不同程序段。

例如：

```text
.text
.data
.bss
```

被加载到不同物理地址。

---

## 虚拟内存

虚拟地址：

```text
0x40000000
```

经过页表转换后可能变成：

```text
0x80000000
```

再映射到真正的物理内存。

---

从本质上说，这三件事情都在做同一件事：

> 根据地址，决定最终应该如何处理。

---

## 总结

RISC-V 的物理地址空间并不等于 RAM。

完整的物理地址空间通常包含：

```text
Physical Address Space
│
├── RAM（主存）
│
├── MMIO（设备映射区）
│
└── Hole（未实现区域）
```

对于 CPU 来说：

```text
地址
 ↓
地址译码
 ↓
RAM / Device / Invalid
```

