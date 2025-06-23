---
layout: post
title: "Typical Root Causes of Kernel Panics"
date: 2025-06-26 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---
- NULL or invalid memory access  
- Use-after-free or accessing freed structures  
- Invalid access to DMA/slab regions  
- Sleeping in atomic context (e.g. in IRQ handler)  
- Watchdog timeouts / soft lockups  