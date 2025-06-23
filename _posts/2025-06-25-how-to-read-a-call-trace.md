---
layout: post
title: "How to Read a Call Trace"
date: 2025-06-25 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

- Structure of a kernel call trace:  
    → Stack trace example with address, offset, and function names  
- Identify the actual crash point, not just the last function  
- Understand key registers: RIP/EIP/PC, etc.  
- Use tools like addr2line, gdb, kallsyms to resolve symbols  
