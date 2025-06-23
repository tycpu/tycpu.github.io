---
layout: post
title: "What is a kernel panic?"
date: 2025-06-23 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

- Difference between Oops and Panic  
    1) Oops: recoverable error, Panic: unrecoverable  

- Common causes of kernel panic:  
    1) NULL pointer dereference  
    2) Page fault in kernel mode  
    3) BUG()/assertion failures  
    4) Stack overflows  
    5) Kernel modules misbehavior  
