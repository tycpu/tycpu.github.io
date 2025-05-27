---
layout: post
title: "The Handoff: bl start_kernel"
date: 2025-05-26 12:00:00 +0900
tags: [linux, kernel, start_kernel, boot-sequence]
---

## 🔧 The Handoff: `bl start_kernel`

> 📝 This post will describe:
>
> - Where and how `start_kernel()` is defined  
> - What happens right before jumping to C code  
> - Why this is the official entry into the main kernel

This marks the end of the boot sequence and the start of kernel initialization.