---
layout: post
title: "Transition from EL3 to EL2/EL1"
date: 2025-05-22 12:00:00 +0900
tags: [el3, el2, el1, tf-a, context-switch]
---

## 🔁 Transition from EL3 to EL2/EL1

> 📝 This post will cover:
>
> - How control is transferred from EL3 to EL2 (Hypervisor) or EL1 (Kernel)  
> - What registers or mechanisms are used for the transition  
> - What a clean handoff looks like  
> - How security state is preserved during this transition

This marks the shift from Secure World to Non-secure World.
