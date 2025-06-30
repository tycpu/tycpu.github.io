---
layout: post
title: "What is a kernel panic?"
date: 2025-06-23 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

---
## 1️⃣ Oops vs Panic
---

Oops means <span class="highlight">**"Recoverable Kernel Error"**</span>.  
If kernel detects some Oops,  
the system dosen't stop and just kills the process which causes problem.  
For example, if there is a NULL pointer dereference(Oop) in the kernel,  
It is handled with memory protection and the log is generated about this Oop.  
So we can know with log if there are some Oops or not during kernel run.

<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> memory protection is with MMU.<br>
Kernel code tries to access to memory, <strong>MMU checks there is a valid mapping or not</strong>.<br>
If there is no valid mapping in that address, <strong>Page fault(Data Abort) exception</strong> is occured.<br>
And exception handler in kernel handles this execption<strong>(kills process)</strong>.<br>

+) if there is an option like panic_on_oops in boot parameter, Oops are translated to Panic.<br>
</div>

And Panic means <span class="highlight">**"Unrecoverable Fatal Error"**</span>.  
**If there is a panic, the system is judged that it dosen't be operated safely**.  
So, the system could be **halt or re-booting**.  
For example, critical memory corruption, stack overflow, and serious hardware fault are kind of Panic.

<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> <strong>system halt is default</strong> when there is a panic.<br>
And if there is an option like panic=1,<br>
system would be re-booting if there is a panic.<br>

And the crash dump can be get with rampdump(kdump).<br>
</div>

<div style="margin:40px 0;"></div>

---
## 2️⃣ Common causes of kernel panic
---

### 1) <span style="color:blue;"> Null pointer dereference </span>
- If kernel code refers to NULL address, **the fault is generated because of memory protection**.  
+) basically, NULL pointer dereference is Oop, but if the address which kernel tries to access is critical,
this action would be the reason of Panic.  
- **NULL dereference in kernel space would be critical for system** unlike with user space dereference.

<div style="margin:20px 0;"></div>

### 2) <span style="color:blue;"> Page fault in kernel mode </span>
- Page fault is caused **when MMU can't find virtual address in page table**.  
- If Kernel accessed to virtual address that dosen't be found in page table, page fault exception(Panic) is occured.  

<div style="margin:20px 0;"></div>

### 3) <span style="color:blue;"> BUG()/assertion failures </span>
- BUG() is a kind of macro which means **the system is reached to some code that must be not accessed logically**.
- Trigger panic directly to ensure system consistency.

<div style="margin:20px 0;"></div>

### 4) <span style="color:blue;"> Stack overflows </span>
- Kernel stack could be overflow because of **recursion or big size local variables**.
- The overflow might be critical because **it can pollute other memories**.

<div style="margin:20px 0;"></div>

### 5) <span style="color:blue;"> Kernel modules misbehavior </span>
- If there are some **uncompatible modules or some drivers with bugs** in system.
- Panic could be caused with invalid memory access, and symbol resolution error.
