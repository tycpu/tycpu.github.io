---
layout: post
title: "sync structure misuses pattern"
date: 2025-07-15 12:00:00 +0900
tags: [linux, kernel, kernel context, structure]
---

# spinlock, mutex, and completion: How Linux Prevents Chaos

---

## 1. The common errors in using locks

We use locks to block race conditions in a multi-thread environment.  
But if you don't use lock structures correctly, there are some problems like:

- **Deadlock**: Hold each other's lock and wait forever  
- **Live Lock**: The status changes continuously, but there is no progress in executing  
- **Priority Inversion**: A higher-priority thread waits because a lower-priority thread holds the lock  
- **Performance Bottleneck**: Bad performance due to lock contention  

---

## 2. Frequent sync misuse patterns

### 1) Lock ordering violation

**Example:**

```c
// Thread A
lock(&L1);
lock(&L2);

// Thread B
lock(&L2);
lock(&L1);
```

→ Deadlock should occur because A holds lock L1 → L2,  
but B holds lock L2 → L1.

**Solution:**  
Make a document about lock order rules and keep them consistent throughout the system.

---

### 2) Double Lock / Nested Lock

**Example:**

```c
mutex_lock(&m);
...
mutex_lock(&m);  // wait forever because same thread tries to get same lock
```

→ Kernel mutex doesn't allow holding a recursive lock.  
So deadlock occurs.

**Solution:**  
If you need to hold a re-entry lock, try to use a recursive lock or completion instead of a mutex.

---
