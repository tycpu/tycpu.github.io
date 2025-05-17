---
layout: post
title: "Analyzing start_kernel() in Linux Kernel"
date: 2025-05-17 11:00:00 +0900
tags: [linux, kernel, boot]
---

## 🧠 What is `start_kernel()`?

The `start_kernel()` function is the main entry point of the Linux kernel after it has been decompressed.

It initializes critical subsystems such as:
- memory management
- interrupt handlers
- scheduler
- and finally starts the first user-space process.

Here’s the definition:

```c
asmlinkage __visible void __init start_kernel(void)
{
    // Kernel startup logic...
}
```
