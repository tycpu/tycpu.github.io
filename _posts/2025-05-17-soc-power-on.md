---
layout: post
title: "What happens when the SoC powers on"
date: 2025-05-17 12:00:00 +0900
tags: [boot, arm64, soc, por]
---

## 🧱 Power-On Reset and the Boot ROM

![Boot Flow Diagram](../assets/1-1.png)

### 🔌 Power-On Reset (POR)

Suppose that there is a SoC which would get some power from PMIC.  
If SoC gets power, it will be initalized with reset state.  
So, a particular hardware circuit would maintain reset and quit it when the initializing is done.  
The name of circuit is **Power-On Reset (POR).**

***

### ⬇️ Reset Pin Behavior

If SoC starts to get some power, the **reset pin** would be **low (active)** state.  
Reset pin is a hardware signal line that controls whether the SoC is in reset state or not.
And if the reset pin changes to high state,  
CPU jumps to **reset vector** immediately.(reset vector is the first PC address)  
Reset vector has the starting address of **boot ROM code**.

***

### 🧠 Why POR Circuit is Needed

The reason why there is a POR circuit is for initializing all digital blocks like CPU and memory controller.  
There are many important things for operating SoC — like **registers, clocks, and voltages.**

To explain POR circuit simply, this circuit **waits for stable operating conditions** for everything mentioned in the previous line.

***

For example:

- POR circuit ensures that registers are properly initialized before releasing reset.  
- POR circuit waits for initializing internal logic like **clock stabilization**.

SoC operation starts with an external **XTAL** clock before PLL is locked.(XTAL is a low but stable clock oscillator)  
POR waits until **lock in PLL (Phase-Locked Loop)**.  
PLL is a kind of **frequency multiplier**.  
This will be locked after stabilization.  
(this lock is different with software lock. it means the phase and frequency are synchronized and stable.)  
So in order to operate normally, POR checks the **lock signal from PLL**.

***

### ✅ Exit of POR and Transition to BOOTROM

Once again, POR looks at the internal state of SoC to ensure stable operation.  
And if **PMIC changes the reset pin to HIGH**, POR **exits the reset state**.  
PMIC changes the reset pin **when SoC voltages are stable**.