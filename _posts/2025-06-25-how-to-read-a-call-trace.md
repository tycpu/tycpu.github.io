---
layout: post
title: "How to Read a Call Trace"
date: 2025-06-25 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

---
## 🚀 1. basic structure of Kernel Call Trace
---

### 1) what is Call Trace?

Call Trace is the print of stack trace when kernel gets PANIC or Oops during execution.  
Especially if there is a PANIC, we can know the reason why there is an error in this excution flow.  
If we follow the call path in Call Trace, we can know the condition of bug and code path of this situation.  

So, Call Trace is the core of analyzing lock, interrupt and resource allocation sequence.  

<div style="margin:20px 0;"></div>

### 2) Example of Call Trace

#### Example

There is an example of Call Trace (ARM arch):

```yaml
Unable to handle kernel NULL pointer dereference at virtual address 0000000000000000
Mem abort info:
ESR = 0x96000004
EC = 0x25: DABT (current EL), IL = 32 bits
SET = 0, FnV = 0
EA = 0, S1PTW = 0
Data abort info:
ISV = 0, ISS = 0x00000004
CM = 0, WnR = 0

CPU: 0 PID: 1234 Comm: insmod Tainted: G        W
Hardware name: ARM Development Platform
pstate: 60400005 (nZCv daif +PAN -UAO)
pc : my_driver_probe+0x2c/0x80
lr : platform_drv_probe+0x20/0x40
sp : ffff80001237bd00
x29: ffff80001237bd00 x28: 0000000000000000 ...
...

Call trace:
my_driver_probe+0x2c/0x80
platform_drv_probe+0x20/0x40
really_probe+0x94/0x2b0
driver_probe_device+0x54/0xe0
device_driver_attach+0x5c/0x70
__driver_attach+0x9c/0xe0
bus_for_each_dev+0x5c/0xa0
driver_attach+0x24/0x30
bus_add_driver+0x14c/0x1f0
driver_register+0x74/0x130
__pci_register_driver+0x44/0x50
my_driver_init+0x1c/0x100
do_one_initcall+0x54/0x200
kernel_init_freeable+0x260/0x2d0
kernel_init+0x14/0x100
ret_from_fork+0x10/0x20
```

<div style="margin:20px 0;"></div>

#### information about main fields

| Field | Example | Description |
|---|---|---|
| **pc** | `my_driver_probe+0x2c/0x80` | function name and offset that are indicated the instruction of crash |
| **lr** | `platform_drv_probe+0x20/0x40` | Link Register (return address), function address of caller |
| **sp** | ffff80001237bd00 | stack pointer |
| **x29~x28...** | | ARM64 general purpose register dump |


<div style="margin:20px 0;"></div>

#### Tips for analyzing PANIC with registers
- PC (program counter)  
➔ it means executing instruction now  
➔ we can know the code line which triggers exception accurately  

- LR (Link Register)  
➔ it means caller function address (return address)  
➔ backtrace reconstruction follows stack frame with LR register  

- SP (Stack Pointer)  
➔ it means the start address of current stackframe  
➔ check and detect overflow in kernel stack  

- Register Dump (x0 ~ x30)  
➔ it means general purpose registers  
➔ x29 indicates frame pointer(fp) and x30 indicates link register(lr)  
➔ x0 ~ x7 mean function parameter (we can know parameter values with these registers in crash)  

<div style="margin:20px 0;"></div>

#### Call Trace Section
We can see call trace below like this backtrace:
``` yaml
    Call trace:
        my_driver_probe+0x2c/0x80
        platform_drv_probe+0x20/0x40
        ...
```
The sequence of flow is:
``` bash
    current function ➔ caller ➔ caller of caller
```
So we should read Call trace top to bottom during debugging.  

And we can know some basic informations in Call trace.  

- At line my_driver_probe+0x2c/0x80:  
➔ my_driver_probe means the function name in this execution flow.  
➔ 0x2c means the distance from start address of this function to crash.  
➔ 0x80 means the size of this function.  

<div style="margin:20px 0;"></div>

#### ARM64 Symbol Resolution
We can de-assemble crash address for finding specific location in code level.  
It can be both objdump and addr2line tools.  

- Objdump is used for de-assemble binary to assembly codes.  
➔ So we can find the specific location if we have an address of crash.  
➔ But with only this, we can't know the specific location in user code level.  
- At that time, we can use addr2line to know the code location.  

We can use these like:  
``` bash
    objdump -d -S vmlinux > vmlinux.objdump  # check function name with offset
    addr2line -e vmlinux (address)           # find the specific location in user code level
```