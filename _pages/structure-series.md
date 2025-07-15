---
layout: default
title: Kernel Structure Series
permalink: /structure-series/
---

# 💥 Building Blocks of the Linux Kernel: Core Structures, Contexts, and Mechanisms

## 1. The basic concepts for understanding kernel structure

- [Understanding Kernel Context: Process, Atomic, Interrupt](/2025/07/10/kernel-contexts-explained.html)
- [Sleepable vs Atomic: What Can You Use Where?](/2025/07/11/sleep-vs-atomic-structures.html)

## 2. Synchronization Structures in Linux Kernel

- [spinlock, mutex, and completion: How Linux Prevents Chaos](/2025/07/14/kernel-sync-structures.html)
- [Common Misuses That Lead to Panic (e.g., Sleep in Atomic)](/2025/07/15/sync-structure-misuse-patterns.html)

## 3. Asynchronization Structures in Linux Kernel

- [Workqueue, Timer, and Tasklet Deep Dive](/2025/07/16/kernel-async-structures.html)
- [When to Use What: Choosing the Right Async Mechanism](/2025/07/17/async-context-selection-guide.html)

## 4. Interrupt and Hardware-related structures

- [Understanding IRQ Contexts & irq_handler_t](/2025/07/18/irq-contexts-and-handlers.html)
- [How tasklets and softirq Fit into Hardware Handling](/2025/07/19/irq-softirq-tasklet.html)

## 5. Relation and Flow between structures in Linux Kernel

- [From device to driver: How Structures Link Together](/2025/07/20/structure-linkage-device-driver.html)
- [Tracking Workflows: work_struct, kthread, and task_struct](/2025/07/21/kernel-structure-flow.html)

## 6. Example of usage and debugging about structures

- [Real Debug Case: Use-after-Free in Timer Callback](/2025/07/22/timer-use-after-free-debug.html)
- [Structure-related Kernel Panic Analysis on ARM64](/2025/07/23/arm64-structure-panic-debug.html)