---
layout: post
title: "Bootloader"
date: 2025-05-24 12:00:00 +0900
tags: [u-boot, bootloader, arm64]
---

## 🚀 Bootloader

After SoC initialization and secure-world composition (BL1 → BL31),  
the system is ready to execute the Linux kernel in the **non-secure world**.

At this point, a software called **BL33** is executed.  
This software is commonly known as the **bootloader**.

The bootloader's role is simple but critical:  
> “Everything that must happen before the Linux kernel can start running.”

---

### 1️⃣ What Is a Bootloader?

We previously discussed **secure boot** in [Post 2-1](./2025-05-20-stages-bl1-bl2-bl31.html),  
which focused on the **secure world** and its initialization steps.

Now, we shift to the **non-secure world** and explore how the Linux kernel is booted.  
The key component in this process is the **bootloader**.

A **bootloader** is a program responsible for **loading the operating system** into memory and **executing it**.  
When the system gets power, the bootloader runs and initializes essential hardware,  
preparing the environment so that the kernel can safely start.

On most ARM-based systems, the most commonly used bootloader is **U-Boot**.

> The kernel cannot run on its own immediately after power-on,  
> because critical hardware like memory, timers, and serial ports are not yet ready.  
> The bootloader fills this gap.

---

### 🛠️ Bootloader Responsibilities

1. **Initialize hardware**  
   - Memory (DRAM), UART, timers, etc.

2. **Load the kernel image**  
   - Read the kernel binary (e.g., `Image`) from storage and load it into memory

3. **Apply the Device Tree**  
   - Provide hardware configuration to the kernel via a `.dtb` file

4. **Load initrd**  
   - A temporary root filesystem used before the real one is mounted

5. **Send bootargs**  
   - Pass CLI arguments to control kernel behavior (e.g., root device, console, cpufreq, thermal)

6. **Execute the kernel**  
   - Transfer control using commands like `bootm`, `booti`, or architecture-specific jumps

> 📌 **Important:** The bootloader’s job ends when the kernel starts.  
> After that, init systems and user space processes are handled by the kernel itself.

---

### 2️⃣ Main Elements Passed to the Linux Kernel

#### 📁 1) Kernel Image
- The bootloader loads the kernel binary into a specific memory address  
- The kernel starts execution from that address

#### 🌲 2) Device Tree (DTB)
- The kernel reads hardware information (CPU cores, memory, clocks, etc.) from the DTB  
- The bootloader loads the `.dtb` file into memory and tells the kernel where to find it

#### 🧰 3) initrd
- A temporary root filesystem before mounting the real rootfs  
- Contains kernel modules (.ko), init scripts, and other essential early-boot components

#### ⚙️ 4) bootargs
- Command-line arguments passed from the bootloader to the kernel  
- Controls boot behavior:  
  Examples:  
  - `root=/dev/mmcblk0p2`  
  - `console=ttyS0`  
  - `cpufreq.default_governor=performance`  
  - `thermal.zone0.*=passive`

---
