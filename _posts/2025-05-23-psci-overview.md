---
layout: post
title: "PSCI Overview (psci_cpu_on, system_off)"
date: 2025-05-23 12:00:00 +0900
tags: [psci, tf-a, power-management, smc]
---

## ⚡ PSCI Overview: `psci_cpu_on`, `system_off`, etc.

> 📝 This post will explain:
>
> - What PSCI (Power State Coordination Interface) is  
> - What kind of APIs it provides (like `psci_cpu_on`, `cpu_suspend`, `system_off`)  
> - How TF-A handles power management for Linux  
> - How the kernel interacts with it using SMC calls

This is the standardized interface between the OS and platform firmware for power control.
