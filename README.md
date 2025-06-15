# Linux Kernel Internals Series by Tycpu

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

### 🧠 1. Boot to Life: Understanding Linux Kernel Boot
- [From Reset to Boot ROM: The First Instructions in an SoC](#)
- [TF-A and EL3: Secure World Initialization Explained](#)
- [Bootloader to Kernel: Setting Up for `head.S`](#)
- [Diving into `head.S`: The Final Step to `start_kernel()`](#)

### 💥 2. Surviving Panic: Debugging Kernel Crashes & Call Traces
- [Understanding kernel panic, Oops, and BUG()](#)
- [Call trace decoding: PC, LR, and symbol offset](#)
- [How to get early logs with `earlycon` and `printk`](#)

### 🔌 3. Device Tree & Driver Binding: When the Hardware Disappears
- [Anatomy of a devicetree: compatible, reg, interrupts](#)
- [How `of_match_table` links to `probe()`](#)
- [initcall levels and debugging missing devices](#)

### 📊 4. Trace Everything: Kernel Tracing Tools for Performance Debugging
- [Using `ftrace` and `trace-cmd` effectively](#)
- [Visualizing task behavior with `kernelshark`](#)
- [Profiling CPU usage with `perf`](#)

### 🔋 5. DVFS in Practice: cpufreq driver & governor Design
- [How cpufreq drivers work: `target_index()` and policies](#)
- [Writing a schedutil-style governor: `get_util()` and boost](#)
- [Thermal & OPP integration with cpufreq](#)

### ⚙️ 6. Task Scheduling & CPU Affinity: How the Kernel Thinks
- [CFS: how tasks are picked and run](#)
- [CPU topology, `sched_domain`, and affinity](#)
- [Energy Aware Scheduling: balancing performance and power](#)

---

## ✨ About This Blog

All posts are written in English and focused on helping both myself and others understand key Linux kernel internals from a CPU software perspective.

Feel free to check out the source code and follow along as I explore the kernel.