---
layout:	post
title: "计算机组成Overview"
subtitle: "Introduction"
date: 2021-03-05 14:00:00
author: "Yinwhe"
header-style: text
tags:
    - Computer Organization
---



# Intro

## Basic

**发展历史** - `ENIAC` → EDVAC → EDSAC → ...

**处理器类型：**

- `RISC` (Reduced Instruction-Set Computer) - MIPS, PA-RISC, SPARC, Alpha, PowerPC, I860 I960(Intel)
- `CISC` (Complex Instruction-Set Computer) - 80x86, Pentium, Pentium Pro, Pentium II & III (Intel)

**计算机类型** - PC, Server, SuperComputer, Embedded Computer...

<details>
  <summary>计算机体系结构的 8 个主要思想</summary>
  <ol>
    <li>面向摩尔定律的设计</li>
    <li>使用抽象简化设计</li>
    <li>加速经常性时间</li>
    <li>通过并行提高性能</li>
    <li>通过流水线提高性能</li>
    <li>通过预测提高性能</li>
    <li>存储层次化 - Register → Cache → Memory → Disk → Tape</li>
    <li>通过冗余提高可靠性</li>
      </ol>
</details>



## Software

**Software** - 包括 `Application Software` 和 `System Software`

- AS 更倾向于 user
- SS 则倾向于 programer，它包括 OS, Complier, Assembler 等

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1go8zj8t9gfj30fo0f43zw.jpg" alt="01-Overview%20c57cde6c8d1a4c08974d1cf46908bbc9/Untitled.png" style="zoom:33%;" />

**语言** - 机器语言 → 汇编语言 → 高级语言

- 高级语言将被 `complier` 编译成汇编语言，再由 `assembler` 汇编成机器语言

## Hardware

`冯诺伊曼架构`下的五大组件 - ALU, Control Unit, Memory, Input, Output

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1go8zji4800j30vi0u0dkh.jpg" alt="01-Overview%20c57cde6c8d1a4c08974d1cf46908bbc9/Untitled%201.png" style="zoom:33%;" />

<details>
  <summary>一些组件介绍</summary>
  <font color="red">Display</font> - CRT (Raster Cathode Ray Tube 老式显像管), LCD(Liquid Crystal Display), LED
  	<li>图像由<b>像素矩阵</b>组成，可以表示成二进制位的矩阵，成为<b>位图(Bit Map)</b></li>
  	<li>计算机硬件采用<b>光栅刷新缓冲区</b>（也称<b>帧缓冲区</b>）来保存位图以支持图像</li>
  <font color="red">MotherBoard</font> -包含 Memory, I/O Device, processor<br/>
  <font color="red">Processor</font> - Datapath, Controler, Cache(SRAM)<br/>
  <font color="red">Memory</font>Main Memory 是易失的(volatile)，Secondary Memory 是非易失的(nonvolatile)
</details>

# Performance

## Definition

Performance的定义是困难的，因为不同的计算机关注点不一样，例如PC更关注**响应时间** (Response Time)，而服务器更关注**吞吐量**(Throughput)

- 一般来说，降低响应时间是能够提高吞吐量的

这里主要考虑响应时间，故将Performance定义为 $Performance = \frac{1}{Execution\  Time}$

度量`执行时间`的几个计时方式：

- `Elapsed Time` - 总共的响应时间，包括Processing, I/O, OS overhead等
- `CPU Time` - 只包含 Processing 的时间
- `CPU Clock`

## Classical CPU Time

$CPU\ Time=\frac{Instructions*CPI}{Frequency }$

- `Instructions` - 指令数
- `CPI` - Clock Cycle Per Instruction, 每条指令的平均时钟数； $CPI = \frac{ClockNumber}{Instructions}$
- `Frequency` - 时钟频率

## Performance Factor

`Algorithm` - affects IC, possibly CPI
`Programming language` - affects IC, CPI
`Compiler` - affects IC, CPI
`ISA` - affects IC, CPI, Frequency

## Development Restraint

> Three Walls

`**Power Wall` 功耗墙**

$Power=Capacitive\ Load*Voltage^2*Frequency$

目前遇到的问题是

- 电压无法进一步下降以降低功耗，因为过低的电压将导致晶体管泄漏电流过大
- 过高的功率将使普通商用处理器难以冷却

`**Memory Wall**` - 内存操作的速度过于落后，很难满足CPU的需求

**`ILP Wall` -** 指很难继续在单个进程的指令流中找到足够的并行性来保持更高性能

## Amdahl Law

$T_{improved} = \frac{T_{affected}}{Improvement\ Factor}+T_{unaffected}$

## 谬误与陷阱

1. 在改进计算机的某个方面时总期望总性能的提高与改进大小成正比
2. 低利用率的计算机功耗更低
3. 使用性能公式的子集来度量性能

    如使用**MIPS(每秒百万指令数)**来衡量性能；$MIPS=\frac{Instruction Count}{Execution\ Time*10^6}$

    有三个问题 - MIPS没有考虑指令的能力；不同的程序有不同的MIPS；MIPS可以独立于性能而发生变化