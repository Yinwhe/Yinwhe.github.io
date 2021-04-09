---
layout:	post
title: "RustSBI"
subtitle: ""
date: 2021-04-09 22:00:00
author: "Yinwhe"
header-style: text
tags:
    - Risc-V
---

# Intro

> `SBI` - RISC-V Supervisor Binary Interface

为管理模式软件提供SBI接口的高级特权软件称为SBI实现或管理模式执行环境(Supervisor Executing Environment `SEE`)，其可以是 `M Mode` 下 `Platform Runtime` 的：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdub4kqd7j30gy07eweo.jpg)

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gpdub5wm5lj30n20awt9c.jpg" alt="RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%201.png" style="zoom:74%;" />

**Why need it?**

- Provides an interface to access machine mode only registers
- Clear separation between Supervisor & Supervisor Execution Environment (SEE)
- Helps to run single OS image across different SEE
- Currently implemented by the Berkeley Boot Loader (BBL)

# Binary Encoding

所有的`SBI`函数共享一个二进制编码，这有助于混合`SBI`扩展。这种二进制编码与`RISC-V Linux`系统调用`ABI`相匹配，后者本身基于`RISC-V ELF psABI`中定义的调用约定。换句话说，SBI调用与标准RISC-V函数调用完全相同，除了：

- `ECALL` 被用与进行控制的转移，而不是 `CALL`
- a7 (or t0 on RV32E-based systems) 用来储存调用的SBI extension ID (`EID`)

有些SBI还会选择在 a6 中储存一个额外的Function ID (`FID`)，这允许SBI在一个extension中实现多个函数

`EID` 和 `FID` 都被编码为有符号的`32`位整数，参数传递遵循 `RISC-V` 的调用规范

`SBI Function`的返回值保存在 `a0` 和 `a1` 中，其中`a0`返回的是Error Code：

```cpp
struct sbiret {
	long error;
	long value;
};
```

| Error Type | Value |
| ---------- | ----- |
|SBI_SUCCESS|0|
|SBI_ERR_FAILED|-1|
|SBI_ERR_NOT_SUPPORTED|-2|
|SBI_ERR_INVALID_PARAM|-3|
|SBI_ERR_DENIED|-4|
|SBI_ERR_INVALID_ADDRESS|-5|
|SBI_ERR_ALREADY_AVAILABLE|-6|

# Base Extension (EID #0x10)

基本扩展部分被设计得尽可能小。因此，它只包含探测哪些`SBI`扩展可用以及查询`SBI`版本的函数。所有`SBI`实现都必须支持基扩展中的函数，因此没有定义错误返回：

| Function Name | FID  | EID  |
| ------------- | ---- | ---- |
|sbi_get_sbi_spec_version|0|0x10|
|sbi_get_sbi_impl_id|1|0x10|
|sbi_get_sbi_impl_version|2|0x10|
|sbi_probe_extension|3|0x10|
|sbi_get_mvendorid|4|0x10|
|sbi_get_marchid|5|0x10|
|sbi_get_mimpid|6|0x10|



# Legacy Extension (EIDS #0x00 - #0x0F)

Legacy SBI Extension 并不使用 FID，每个function都是独立的 extension

**Set Timer (EID #0x00)**

```cpp
void sbi_set_timer(uint64_t stime_value)
```

为`stime_value`时间之后的事件设置时钟，`stime_value`以绝对时间为单位。这个函数同时也清除挂起的计时器中断位pending timer interrupt bit

如果Supervisor希望清除计时器中断而不调度下一个计时器事件，它可以请求一个计时器中断到无限远的未来(即(uint64_t)-1)，或者它可以通过清除`sie.stie`来屏蔽计时器中断。

**Console Putchar (EID #0x01)**

```cpp
void sbi_console_putchar(int ch)
```

把`ch`代表的数据写到调试控制台。与`sbi_console_getchar()`不同，如果还有任何待传输的字符，或者接收终端还没有准备好接收字节，则此`SBI`调用将阻塞；如果控制台根本不存在，那么字符就会被丢弃。

**Console Getchar (EID #0x02)**

```cpp
int sbi_console_getchar(void)
```

从调试控制台读取一个字节，成功则返回字节，失败返回`-1` ；这是Legacy Extension中唯一一个具有非void返回类型的SBI调用。

**Clear IPI (EID #0x03)**

```cpp
void sbi_clear_ipi(void)
```

如果有的话，清除所有的pending IPI(Inter-Processor Interrupt)；不推荐使用，因为S mode可以直接清除`sip.SSIP`

**Send IPI (EID #0x04)**

```cpp
void sbi_send_ipi(const unsigned long *hart_mask)
```

向`hart_mask`中定义的所有harts发送一个IPI，IPI以Supervisor Software Interrupts的形式出现在接收端。

`hart_mask`是一个虚拟地址，指向harts的位向量。位向量表示为一个unsigned long序列，其长度等于系统中hart的数量除以unsigned long中的位数，向上取整到下一个整数。

**Remote FENCE.I (EID #0x05)**

```cpp
void sbi_remote_fence_i(const unsigned long *hart_mask)
```

指导其他的`hart`执行 `Fence_i` 指令，其他hart通过`hart_mask`来指定。

**Remote SFENCE.VMA (EID #0x06)**

```cpp
void sbi_remote_sfence_vma(const unsigned long *hart_mask,
                           unsigned long start,
                           unsigned long size)
```

指导其他的`hart`执行`sfence.vma`指令，其他hart通过hart_mask来指定。其范围包括虚拟地址从`start`到`start+size`的所有虚拟地址。

**Remote SFENCE.VMA with ASID (EID #0x07)**

```cpp
void sbi_remote_sfence_vma_asid(const unsigned long *hart_mask,
                                unsigned long start,
                                unsigned long size,
                                unsigned long asid)
```

同上一指令，但只包含了给定的 `ASID`

**System Shutdown (EID #0x08)**

```cpp
void sbi_shutdown(void)
```

将所有hart置于关机状态，这个SBI调用不返回。

[Untitled](https://www.notion.so/26eac55a3fbb4e339dcdc0f46d855cd5)

## Timer Extension (EID #0x54494D45 "TIME")

**Set Timer (FID #0)**

```cpp
struct sbiret sbi_set_timer(uint64_t stime_value)
```

为`stime_value`时间之后的事件设置时钟，`stime_value`以绝对时间为单位。这个函数同时也清除挂起的计时器中断位。

如果Supervisor希望清除计时器中断而不调度下一个计时器事件，它可以请求一个计时器中断到无限远的未来(即(uint64_t)-1)，或者它可以通过清除`sie.stie`来屏蔽计时器中断。

## IPI Extension (EID #0x735049 "sPI: s-mode IPI")

**Send IPI (FID #0)**

```cpp
struct sbiret sbi_send_ipi(unsigned long hart_mask,
                           unsigned long hart_mask_base)
```

向`hart_mask`中定义的所有harts发送一个IPI，IPI以Supervisor Software Interrupts的形式出现在接收端。

返回值 - `SBI_SUCCESS` IPI was sent to all the targeted harts successfully.

## RFENCE Extension (EID #0x52464E43 "RFNC")

任何希望使用地址范围(即`start_addr`和`size`)的函数必须遵守以下限制：Remote fence function会充当一个 full TLB flush，如果

- `start_addr` 和 `size` 都是 0
- `size` 等于 $2^{XLEN}-1$

**Remote FENCE.I (FID #0)**

```cpp
struct sbiret sbi_remote_fence_i(unsigned long hart_mask, unsigned long hart_mask_base)
```

- 命令Remote Harts执行 `FENCE.I` 指令
- 返回值如果是 `SBI_SUCCESS` 则表面 IPI成功发送到了所有目标harts

**Remote SFENCE.VMA (FID #1)**

```cpp
struct sbiret sbi_remote_sfence_vma(unsigned long hart_mask,
                                    unsigned long hart_mask_base,
                                    unsigned long start_addr,
                                    unsigned long size)
```

- 命令remote harts执行 `SFENCE.VMA` 指令，包含的虚拟地址范围由 `start` 和 `size` 指定
- 返回值如果是 `SBI_SUCCESS` 则表面 IPI成功发送到了所有目标harts
- 返回值如果是 `SBI_ERR_INVALID_ADDRESS` ，则`start`或者`size`不合法

**Remote SFENCE.VMA with ASID (FID #2)**

```cpp
struct sbiret sbi_remote_sfence_vma_asid(unsigned long hart_mask,
                                         unsigned long hart_mask_base,
                                         unsigned long start_addr,
                                         unsigned long size,
                                         unsigned long asid)
```

- 基本同上，但只包含给定的`ASID`

**Remote HFENCE.GVMA with VMID (FID #3)**

```cpp
struct sbiret sbi_remote_hfence_gvma_vmid(unsigned long hart_mask,
                                          unsigned long hart_mask_base,
                                          unsigned long start_addr,
                                          unsigned long size,
                                          unsigned long vmid)
```

- 指示Remote hart执行一个或多个`HFENCE.GVMA`指令，涵盖从start到size的Guest物理地址范围，仅针对给定的`VMID` ；这函数调用仅对实现`hypervisor extension`的harts有效。

**Remote HFENCE.GVMA (FID #4)**

```cpp
struct sbiret sbi_remote_hfence_gvma(unsigned long hart_mask,
                                     unsigned long hart_mask_base,
                                     unsigned long start_addr,
                                     unsigned long size)
```

- 指示Remote hart执行一个或多个`HFENCE.GVMA`指令，包括所有从start到size的Guest物理地址范围。此函数调用仅对实现`hypervisor extension`的harts有效。

**Remote HFENCE.VVMA with ASID (FID #5)**

```cpp
struct sbiret sbi_remote_hfence_vvma_asid(unsigned long hart_mask,
                                          unsigned long hart_mask_base,
                                          unsigned long start_addr,
                                          unsigned long size,
                                          unsigned long asid)
```

- 指示Remote hart执行一个或多个`HFENCE.VVMA`指令，涵盖了调用hart的给定ASID和当前VMID(在hgatp CSR中)的start和size之间的guest虚拟地址范围。此函数调用仅对实现`hypervisor extension`的harts有效。

**Remote HFENCE.VVMA (FID #6)**

```cpp
struct sbiret sbi_remote_hfence_vvma(unsigned long hart_mask,
                                     unsigned long hart_mask_base,
                                     unsigned long start_addr,
                                     unsigned long size)
```

- 指示Remote hart执行一个或多个`HFENCE.VVMA`指令，涵盖了调用hart的当前VMID(在hgatp CSR中)的start和size之间的guest虚拟地址范围。此函数调用仅对实现`hypervisor extension`的harts有效。

## Hart State Management Extension (EID #0x48534D "HSM")

Hart状态管理(HSM)扩展引入了一套Hart状态和一组功能，允许管理模式软件请求Hart状态改变。

[一些state](https://www.notion.so/e6ce29b0fd574f33ab2603379e9cf449)

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%202.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubfas4yj30n20dm0to.jpg)

**HART start (FID #0)**

```cpp
struct sbiret sbi_hart_start(unsigned long hartid,
                             unsigned long start_addr,
                             unsigned long opaque)
```

- 请求`SBI`以S-mode在`start_addr`指定的地址开始执行目标hart，一些特定寄存器的值做如下指定：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%203.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubhq263j31240eimyu.jpg)

- 这个调用是**异步的**，只要`SBI`能够确保返回值是正确的，`sbi_hart_start()`就**可以在目标hart开始执行之前返回**。建议如果`SBI`是在M-mode下执行的平台运行时固件，那么它必须在以S-mode执行**之前**配置PMP和其他M-mode的状态。
- `hartid`参数指定要启动的目标hart。
- `start_addr`指向一个运行时指定的物理地址，在这个地址中，hart可以以监督模式开始执行。

一些可能的返回值为：

- SBI_SUCCESS - 正常，hart即将开始执行
- SBI_ERR_INVALID_ADDRESS - `start_addr`是无效的，可能是有以下原因：该地址不合法；该地址被PMP禁止在S-mode下运行。
- SBI_ERR_INVALID_PARAM - `hartid`不是一个有效的`hartid`，相应的hart不能在S-mode下启动。
- SBI_ERR_ALREADY_AVAILABLE - 给定的hart已经启动了
- SBI_ERR_FAILED - 因未知原因启动失败

**HART stop (FID #1)**

```cpp
struct sbiret sbi_hart_stop(void);
```

- 请求`SBI`停止在S-mode下执行的该hart(调用者)，并将其所有权返回给`SBI`。此调用在正常情况下不会返回。`sbi_hart_stop()`必须在S-mode中断被禁用的情况下调用

可能的返回值： SBI_ERR_FAILED - 停止当前hart失败

**HART get status (FID #2)**

```cpp
struct sbiret sbi_hart_get_status(unsigned long hartid);
```

- 获取给定hart的当前状态(或HSM State id)，可能的错误为： SBI_ERR_INVALID_PARAM，给定的hartid或start_addr不合法
- 由于并发的`sbi_hart_start()`或`sbi_hart_stop()`或`sbi_hart_suspend()`调用，harts可以在任何时候转换HSM状态，这个函数的返回值**可能不代表**在验证返回值时hart的实际状态。

**HART suspend (FID #3)**

```cpp
struct sbiret sbi_hart_suspend(uint32_t suspend_type,
                               unsigned long resume_addr,
                               unsigned long opaque)
```

- 请求`SBI`将调用的hart置于由参数`susend_type`指定的平台特定挂起(或低功耗)状态。当接收到中断或平台特定的硬件事件时，hart将**自动从挂起状态恢复正常执行**。
- 平台特定暂停状态的`hart`可以是**保留**或**非保留**性质的。保留挂起状态将保留所有特权模式的hart register和CSR值，而非保留挂起状态将不保留hart register和CSR值。
    - 从保留挂起状态恢复是很简单而直接的。
    - 从非保留挂起状态恢复相对来说更复杂，需要软件恢复各种hart寄存器和csr。当从非保留挂起状态恢复时，hart会跳转到由 `resume_addr` 指定的S-mode地址上，一些特定寄存器做如下指定：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%204.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubla80qj311s0egabq.jpg)

- 参数的`susend_type`是`32`位宽的，取值做如下指定：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%205.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubnmuekj316e0icadd.jpg)

- `resume_addr`参数指向一个运行时指定的物理地址，hart可以在非保留挂起之后以S-mode继续从该处执行。

一些可能的返回值：

- SBI_SUCCESS - hart已经中止并成功地从保留中止状态恢复。
- SBI_ERR_INVALID_PARAM - `suspend_type` 不合法
- SBI_ERR_NOT_SUPPORTED - `suspend_type` 合法但没有实现
- SBI_ERR_INVALID_ADDRESS - resume_addr是无效的，可能是由以下原因造成：它不是一个有效的物理地址；该地址被PMP禁止在S-mode下运行。
- SBI_ERR_FAILED - 挂起请求因未知原因失败。

## System Reset Extension (EID #0x53525354 "SRST")

The System Reset Extension provides a function that allow the supervisor software to request **system-level reboot or shutdown**. The term "system" refers to the world-view of supervisor software and the underlying SBI implementation could be machine mode firmware or hypervisor.

**System reset (FID #0)**

```cpp
struct sbiret sbi_system_reset(unsigned long reset_type,
                               unsigned long reset_reason)
```

- 根据提供的`reset_type`和`reset_reaso` 重置系统。这是一个**同步调用**，如果成功则不会返回。
- reset_type 是一个32位的值，取值如下：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%206.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubsu53qj316e0ewac9.jpg)

- `reset_reason`是一个**可选**参数，表示系统重置的原因。该参数宽32位，可能取值如下：

![RustSBI%20f122ceddc0db47e19575c3d232745e79/Untitled%207.png](https://tva1.sinaimg.cn/large/008eGmZEly1gpdubqh9c8j31640eo0v2.jpg)

一些可能的返回值：

- SBI_ERR_INVALID_PARAM - `reset_type`或`reset_reason`无效。
- SBI_ERR_NOT_SUPPORTED - `reset_type` 有效但并没有被实现
- SBI_ERR_FAILED - 未知原因导致重置请求失败。