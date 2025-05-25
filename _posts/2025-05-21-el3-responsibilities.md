---
layout: post
title: "EL3 Responsibilities"
date: 2025-05-21 12:00:00 +0900
tags: [tf-a, el3, secure-world, boot]
---

## 🛡️ EL3 Responsibilities

> 📝 This post will explain:
>
> - What EL3 is responsible for during early boot  
> - How Secure World is configured  
> - What BL31 does before handing off to EL1 or EL2  
> - How SMC calls and context switching are handled

This is the most privileged level of execution and the core of TF-A’s secure boot.
