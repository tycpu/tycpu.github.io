---
layout: post
title: "Understanding Kernel Context: Process, Atomic, Interrupt"
date: 2025-07-10 12:00:00 +0900
tags: [linux, kernel, kernel context, structure]
---

# Understanding Kernel Context: Process, Atomic, Interrupt

---
## 1. why do we have to understand about kernel context?
---

Context means like flow of code executing.  
And we can find three types of context in linux kernel.

| Context Type      | Can Sleep | Can Scheduling | Examples                          |
|-------------------|-----------|----------------|-----------------------------------|
| Process context   | ✅        | ✅             | system call, kthread, etc         |
| Atomic context    | ❌        | ❌             | spinlock section, preempt off     |
| Interrupt context | ❌        | ❌             | IRQ handler, softirq, NMI         |

We can often see like this error.
``` yaml
    BUG: sleeping function called from invalid context
```
This means that some structures or functions are called and these are not allowed in current context.  
So we have to know about these three types of context.  


<div style="margin:40px 0;"></div>

---
## 2. Three contexts: Process, Atomic, Interrupt
---

### 1) Process Context
characteristic
→ there is a current process that is executed now
→ can sleep (msleep(), wait_event()..)
→ can scheduling
→ most of normal functions are called in this context

example
→ general executing for kernel functions
→ in the system calls (read(), ioctl(), open()...)
→ kthread or workqueue callback
→ handler for reading files in /proc and /sys

available structures
→ mutex, semaphore, completion, wait_event...
→ schedule(), msleep()...

<div style="margin:20px 0;"></div>

### 2) Atomic Context
characteristic
→ can't sleep
→ can't get preemption
→ can't call the function which is sleeping
→ can be executed very short

example (how to enter)
→ get spin_lock()
→ rcu_read_lock()
→ preempt_disable()
→ local interrupt disable (local_irq_disable())

available structures
→ spinlock, rcu, atomic_t
→ busy loop (cpu_relax()...)

prohibitions
→ never be called in msleep(), mutex_lock(), wait_event() calling. (Kernel Panic)