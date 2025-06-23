---
layout: post
title: "Practical Debugging Tips for Real Systems"
date: 2025-06-27 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

- Always capture full logs: dmesg, serial console, syslog  
- Check for recently loaded modules  
- Grep source for suspicious functions  
- Enable earlyprintk or CONFIG_DEBUG_KERNEL when needed  
- In virtualized environments: use snapshots for reproducibility  