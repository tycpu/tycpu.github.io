---
layout: post
title: "sleep vs atomic structures"
date: 2025-07-11 12:00:00 +0900
tags: [linux, kernel, kernel context, structure]
---

# Sleepable vs Atomic: What can you use where?

---
## 1. Review: what is context
---

Context means like flow of code executing.  
And we can find three types of context in linux kernel.

| Context Type      | Can Sleep | Can Scheduling | Examples                          |
|-------------------|-----------|----------------|-----------------------------------|
| Process context   | ✅        | ✅             | system call, kthread, etc         |
| Atomic context    | ❌        | ❌             | spinlock section, preempt off     |
| Interrupt context | ❌        | ❌             | IRQ handler, softirq, NMI         |

<div style="margin:40px 0;"></div>

---
## 2. Classification by structure and function
---


| Structure / Functions     | Can sleep | Available Context             |
|---------------------------|-----------|-------------------------------|
| mutex                     | ✅        | Process only                  |
| spinlock                  | ❌        | Process / Atomic / IRQ        |
| msleep()                  | ✅        | Process only                  |
| wait_event()              | ✅        | Process only                  |
| rcu_read_lock()           | ❌        | Process / Atomic / IRQ        |
| completion                | ✅        | Process only                  |
| kzalloc(GFP_KERNEL)       | ✅        | Process only                  |
| kzalloc(GFP_ATOMIC)       | ❌        | Atomic / IRQ context          |



<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> actually, rcu_read_lock() is not a type of lock(block X). <br>
But this is used for ensuring consistency between reading and writing of RCU. <br>
So there is a strong constraint in rcu_read_lock() that means non-sleep. <br>
Because if rcu_read_lock() can be executed in sleep context, <br>
writers should wait and this can result delay. <br>
So, if rcu_read_lock() is executed in Process context, <br>
it couldn't sleep. <br>
</div>

<div style="margin:40px 0;"></div>

---
## 3. Example of wrong usages
---

``` c
    spin_lock(&lock);
    msleep(100);  // ❌ sleeping in atomic context
```
✅ spin_lock() disables preemption and enters atomic context.  
You must not call any sleep function such as msleep() in this state.  


``` c
    irq_handler(...) {
        mutex_lock(&lock);  // ❌ invalid context
    }
```
✅ IRQ handler runs in interrupt context where both sleep and scheduling are disallowed.  
But mutex_lock() may block and sleep internally, so it's only allowed in process context.  
Using it here leads to a kernel warning or panic.  

<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> so, the selection of structure is really important. <br>
If you want to protect some resources in sleep-available context, you can use mutex. <br>
But if you try to use spinlock in that context, it would be fault. <br>
</div>

<div style="margin:40px 0;"></div>

---
## 5. Hints for debugging
---

### 1) in_atomic() or WARN_ON(in_atomic())

``` c
    if (in_atomic()) {
        pr_warn("This code is running in atomic context!\n");
    }
```
- in_atomic returns true when the current context is atomic (sleep is not allowed).  
- this is useful for condition checking.  

### 2) dump_stack()
- dump_stack() prints current status of stack trace.  
- it can include message like this:

``` php
    BUG: sleeping function called from invalid context at kernel/mutex.c:123
    in_atomic(): 1, irqs_disabled(): 0, non_block: 0
```

✅ This means a sleepable structure (like mutex) was used in a non-sleepable context (like atomic or IRQ).  

### 3) checking context status in ARM64 architecture

``` less
    [   12.345678] CPU: 1 PID: 1234 Comm: my_task
    [   12.345680] pstate: 60400005 (nZCv daif +PAN -UAO BTYPE=--)
    [   12.345682] esr: 96000045 -- Data abort
    [   12.345684] far: 00000000deadbeef
    [   12.345686] BUG: sleeping function called from invalid context
    [   12.345688] in_atomic(): 1, irqs_disabled(): 0, preempt_count: 1
```

- pstate: flag of current CPU status. daif means interrupt disable or not.  
- esr: type of exception. 96000045 means translation fault.  
- far: Fault Address Register — address that triggered the fault.  
- preempt_count: if it is not 0, can't preemption(atomic context).  
- in_atomic(): if it is 1, can't sleep.  