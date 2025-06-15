---
layout: default
title: Linux Kernel Boot Series
permalink: /boot-series/
---

# 🧠 Linux Kernel Boot Series (for ARM64 SoC)

## 1. 🧩 Power-On & Boot ROM

- [What happens when the SoC powers on](/2025/05/17/soc-power-on.html)
- [Role of the Boot ROM (BL0)](/2025/05/18/bootrom.html)
- [Jump to TF-A (Trusted Firmware-A)](/2025/05/19/jump-to-bl1.html)

## 2. 🛡️ TF-A: Secure World Initialization

- [Stages: BL1 → BL2 → BL31](/2025/05/20/stages-bl1-bl2-bl31.html)
- [EL3 Responsibilities](/2025/05/21/el3-responsibilities.html)
- [Transition from EL3 to EL2/EL1](/2025/05/22/transition-el3-to-el1.html)
- [PSCI Overview (`psci_cpu_on`, `system_off`)](/2025/05/23/psci-overview.html)

## 3. 🚀 Bootloader to Linux Kernel

- [Bootloader Overview](/2025/05/24/bootloader.html)
- [Entering the Linux Kernel: `head.S`](/2025/05/25/heads.html)
- [The Handoff: `bl start_kernel`](/2025/05/26/start-kernel.html)