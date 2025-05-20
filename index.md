---
layout: home
title: Linux Kernel Internals Series by Tycpu
---

Welcome! This blog documents my journey as a CPU software engineer working with Linux kernel internals — focusing on ARM64 boot, scheduling, and power management.

I work at Samsung Electronics in South Korea as a software engineer.  
I've written code related to CPU frequency drivers, governors, and the scheduler in the Exynos platform.

---

## 📚 Blog Curriculum

This blog is structured as a series of technical posts based on my real-world experience.  
I’m organizing it by topics so readers can follow along.

### 📦 0. Setup & Introduction
- ✅ Blog setup with GitHub Pages + minima
- ✅ Setting up a kernel dev environment for ARM64

### 🧠 1. Boot Process
- [How Linux boots on ARM64: from reset to `start_kernel()`](#)
- [Understanding `setup_arch()` and early MMU setup](#)
- [ARM64 memory layout and paging](#)
- [Device Tree explained](#)

### 🧵 2. CPU Bring-up & SMP
- [CPU bring-up and `secondary_start_kernel()`](#)
- [PSCI and CPU hotplug](#)

### ⚙️ 3. Scheduler
- [CFS scheduler explained](#)
- [What are `sched_domain` and `cpu_topology`](#)
- [Energy Aware Scheduling (EAS)](#)

### 🔋 4. cpufreq & Power Management
- [Writing a cpufreq driver](#)
- [Understanding OPP tables](#)
- [Thermal framework integration](#)

### 🔧 5. Debugging Tools & Practices
- [Using ftrace and trace-cmd](#)
- [Reading `sched_debug`](#)
- [Analyzing kernel logs and boot output](#)

---

## ✨ About This Blog

All posts are written in English and focused on helping both myself and others understand key Linux kernel internals from a CPU software perspective.

You can also browse the [README on GitHub](https://github.com/tycpu/tycpu.github.io) for a curriculum overview.
