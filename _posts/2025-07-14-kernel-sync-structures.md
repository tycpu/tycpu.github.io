---
layout: post
title: "kernel sync structures"
date: 2025-07-14 12:00:00 +0900
tags: [linux, kernel, kernel context, structure]
---

# spinlock, mutex, and completion: How Linux Prevents Chaos

---
## 1. Why synchronization is needed
---

<div style="margin:20px 0;"></div>

In linux kernel, many tasks and interrups are executed on multi-core environment.  
So there are a lot of executing flows, and if they access to shared resources like struct and buffer,  
**data corruption + race condition + unexpectable operating** can occur.

So, linux kernel provides various type of structures for sync.

<div style="margin:40px 0;"></div>

---
## 2. Spinlock
---

<div style="margin:20px 0;"></div>

### 🔧 Features

| Key                | Value                                                       |
|--------------------|-------------------------------------------------------------|
| Is this sleepable? | ❌ (it can also be used in atomic context)                  |
| Context            | Process / Atomic / IRQ                                      |
| Usage              | quick lock, short protected context, protect resources in non-sleep env |

<div style="margin:20px 0;"></div>

### 💻 Example

```c
    spinlock_t lock;

    spin_lock_init(&lock);

    spin_lock(&lock);
    // critical section
    spin_unlock(&lock);
```

<div style="margin:20px 0;"></div>

### ⚠️ Warnings
1. never call sleep functions like msleep() and mutex_lock()  
2. spin_lock_irqsave() is interrupt-safe  
3. if spinlock gets lock for a long time, soft lockup can be occured.  

<div style="margin:40px 0;"></div>

---
## 3. Mutex
---

<div style="margin:20px 0;"></div>

### 🔧 Features

| Key                | Value                                                       |
|--------------------|-------------------------------------------------------------|
| Is this sleepable? | ✅                 |
| Context            | Process context only                                     |
| Usage              | lock for resource accessing with avaliable sleep |

<div style="margin:20px 0;"></div>

### 💻 Example

```c
    struct mutex my_mutex;

    mutex_init(&my_mutex);

    mutex_lock(&my_mutex);
    // critical section
    mutex_unlock(&my_mutex);
```

<div style="margin:20px 0;"></div>

### ⚠️ Warnings
1. it can't be used in IRQ and atomic context.  
2. it is sleepable, so lock can be kept up for a long time.  
3. checking code flow is needed for avoiding deadlock.  

<div style="margin:40px 0;"></div>
