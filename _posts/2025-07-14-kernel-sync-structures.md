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


<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> mutex lock sleeps when it has to wait for another task.<br>
but spinlock doesn't sleep. It just repeats same instruction until completion other task. <br>
</div>


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

---
## 4. Completion
---

<div style="margin:20px 0;"></div>

### 🔧 Features

| Key                | Value                                                       |
|--------------------|-------------------------------------------------------------|
| Is this sleepable? | ✅                 |
| Context            | Process context only                                     |
| Usage              | synchronization with event (ex. waiting for task completion) |

<div style="margin:20px 0;"></div>

### 💻 Example

```c
    DECLARE_COMPLETION(my_comp);

    // producer side
    complete(&my_comp);

    // consumer side
    wait_for_completion(&my_comp);
```

<div style="margin:20px 0;"></div>

### ⚠️ Warnings
1. It can be used only in process context because it causes sleep.  
2. After using complete_all(), all of waiting tasks wake up.  

<div style="margin:40px 0;"></div>

---
### 5. Compare
---
<br>

| structure       | sleepable         |  available context      | main usage         |
|-----------------|-------------------|-------------------------|--------------------|
| spinlock        | ❌               | Process / Atomic / IRQ | fast protection for resources (non-sleep) |
| mutex           | ✅               | Process context only   | common protection for sharing resources   |
| completion      | ✅               | Process context only   | sync for async events (waiting)           |

<div style="margin:40px 0;"></div>

---
### 6. Bad usages
---
<br>

``` c
    spin_lock(&lock);
    msleep(100);  // ❌ : try to sleep in atomic context
    spin_unlock(&lock);
```

``` c
    irq_handler(...) {
        mutex_lock(&lock);  // ❌ : try to use mutex lock in IRQ context
    }
```

These bad usages can cause warning message like this:  

``` sql
    BUG: sleeping function called from invalid context
```

So, we can use these structures in situation like this:

| Situation                | recommended structure                                  |
|--------------------------|---------------------------------------------------------|
| IRQ handler or atomic context | spinlock                 |
| handling user request (system call)            | mutex                         |
| have to wait until completion of a task         | completion           |

