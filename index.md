---
layout: home
title: Linux Kernel Boot Series
---

# 🧠 Linux Kernel Boot Series (for ARM64 SoC)

This blog documents my learning journey as a CPU software engineer  
working on Linux kernel and low-level boot internals for ARM64-based SoCs.  
The series is organized by boot stages and exception levels.

---

## 🧩 1. Power-On Reset and the Boot ROM

- [What happens when the SoC powers on](/2025-05-17-soc-power-on.md)
- [Role of the Boot ROM (BL0)](/2025-05-18-bootrom.md)
- [Jump to TF-A (Trusted Firmware-A)](/2025-05-19-jump-to-bl1.md)

---

## 🛡️ 2. TF-A: Secure World Initialization

- [Stages: BL1 → BL2 → BL31](/2025-05-20-stages-bl1-bl2-bl31.md)
- [EL3 Responsibilities](/2025-05-21-el3-responsibilities.md)
- [Transition from EL3 to EL2/EL1](/2025-05-22-transition-el3-to-el1.md)
- [PSCI Overview (`psci_cpu_on`, `system_off`)](/2025-05-23-psci-overview.md)

---

## 🚀 3. Entering the Linux Kernel

- [U-Boot: Bootloader Duties](/2025-05-24-u-boot.md)
- [Entering the Linux Kernel: `head.S`](/2025-05-25-heads.md)
- [The Handoff: `bl start_kernel`](/2025-05-26-start-kernel.md)

---

## 📌 Notes

All posts are written in English to help me prepare for global tech roles,  
and focus on the real-world understanding of the Linux kernel boot process.  
