---
layout: post
title: "The Handoff: bl start_kernel"
date: 2025-05-26 12:00:00 +0900
tags: [linux, kernel, start_kernel, boot-sequence]
---

## 🔧 The Handoff: `bl start_kernel`


### 🚪 enter process of start_kernel()

We discussed about head.S in the last post.  
At the head.S, MMU, page table, and stack are initialized before entering <span class="highlight">__primary_switched</span>.  
So we can summarize the flow about entering start_kernel() like this:

``` text 
  bootloader
    ↓
  kernel_entry (head.S)
    ↓
  el2_setup → el1_setup → create page tables, mmu enable, set up stack
    ↓
  __primary_switched:
      mov x0, fdt
      bl start_kernel
```

<span class="highlight">__primary_switched</span> is the end point of setting for platform which is for entering C code.  
So after __primary_switched, all of assembly initializings are done and jump to C functions.  
The reason why this process is needed is because before __primary_switched, C code can't be executed — because  
there are no virtual addresses, stack, page tables, and etc...  
(all of these must be needed for executing C code)

***
### ⚙️ what things are executed in start_kernel()?
<span class="highlight">start_kernel()</span> is the starting point of initializing C code in linux kernel  
and setting point of primary sub-systems like scheduler and IRQ.

This function is located in `init/main.c` and defined like this:
``` c
  asmlinkage __visible void __init start_kernel(void)
```
- asmlinkage: this function is called in assembly code, so arguments passing is forced with stack.
- __visible: prevent this symbol from being removed by linker optimization
- __init: mark for erasing from the momory after initializing

The initialing flow at start_kernel() can be summarize like this:
``` text
  start_kernel()
  ├── setup_arch()           # intiailize env with architecture
  ├── setup_command_line()   # parse commandline
  ├── setup_nr_cpu_ids()
  ├── mm_init()              # initialize memory subsystem like vamlloc and page allocator
  ├── sched_init()           # intialize kernel scheduler data structure
  ├── time_init()            # initialize timer
  ├── console_init()         # starting point of executing printk
  └── rest_init()            # start kerenl tasks: execute kthreadd, init
```

And especially after <span class="highlight">rest_init()</span>, these are executed:
``` text
  rest_init()
  ├── kernel_thread(kernel_init)
  │   └──→ PID 1 (init process)
  └── kernel_thread(kthreadd)
      └──→ PID 2 (manage kernel threads)
```
So at this point, kernel starts multitasking and finish to start userspace.

***

### 🧰 init process: what else does it initialize?

+) Major initializations performed by the init process (representative examples):
| Init task | Example |
|-----------|---------|
| Root filesystem mount | rootfs, initramfs, real disk partition |
| Make device node | udev, static dev |
| Set userland environment | /etc/inittab, /init, systemd |
| Load driver | via init script or systemd |
| Start userspace service | logger, sshd, etc |

***

### 🧠 after start_kernel(): where is the start point about "kernel is executing?"
I want to discuss about:  
<span class="highlight">If kernel is entered to start_kernel(), can I ensure the booting is done?</span>

- Until `start_kernel()`, the thread is single  
→ this point is executed with a single thread, so interrupt is also off  
→ subsystems of kernel like memory and scheduler are ready for executing at this point  
→ but kernel can't be executed with multitasking, and userspace can't be operated  

- At `rest_init()`, multitasking is started:
``` c
  rest_init()
  {
      kernel_thread(kernel_init, ...);  // → PID 1: init process
      kernel_thread(kthreadd, ...);     // → PID 2: manage kernel thread
      cpu_startup_entry(CPUHP_ONLINE);  // → idle task loop (PID 0)
  }
```
→ from rest_init(), kernel runs tasks with scheduler

***

### ✅ Conclusion

After <span class="highlight">rest_init()</span>,  
especially finishing <span class="highlight">init process (kernel_init)</span>,  
we can conclude this point is the **starting of kernel execution**.