---
layout: post
title: "What happens when the SoC powers on"
date: 2025-05-17 12:00:00 +0900
tags: [boot, arm64, soc, por]
---

## 🧱 Power-On Reset and the Boot ROM
***
<div style="margin:20px 0;"></div>

![Boot Flow Diagram](/assets/1-1.png)

---
### 🔌 Power-On Reset (POR)
---

Suppose that there is a SoC which would get some power from PMIC.  
If SoC gets power, it will be initalized with <span class="highlight">**reset state.**</span>  
So, a particular hardware circuit would maintain reset and quit it when the initializing is done.  
The name of circuit is <span class="highlight">**Power-On Reset (POR).**</span>
<div style="margin:40px 0;"></div>

---
### ⬇️ Reset Pin Behavior
---

If SoC starts to get some power, <span class="highlight">the **reset pin** would be **low (active)** state.</span>  
Reset pin is a hardware signal line that controls whether the SoC is in reset state or not.
And if the reset pin changes to high state, CPU jumps to **reset vector** immediately.(reset vector is the first PC address)  
Reset vector has the <span class="highlight">starting address of **boot ROM code**.</span>
<div style="margin:40px 0;"></div>

---
### 🧠 Why POR Circuit is Needed
---

The reason why there is a POR circuit is for initializing all digital blocks like CPU and memory controller.  
There are many important things for operating SoC — like **registers, clocks, and voltages.**

To explain POR circuit simply, <span class="highlight">this circuit **waits for stable operating conditions** </span>for everything mentioned in the previous line.

For example:

- POR circuit ensures that registers are properly initialized before releasing reset.  
- POR circuit waits for initializing internal logic like **clock stabilization**.

SoC operation starts with an external **XTAL** clock before PLL is locked.  
(XTAL is a low but stable clock oscillator)  
POR waits until **lock in PLL (Phase-Locked Loop)**.  
PLL is a kind of **frequency multiplier**.  
This will be locked after stabilization.  
(this lock is different with software lock. it means the phase and frequency are synchronized and stable.)  
So in order to operate normally, POR checks the **lock signal from PLL**.
<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> Remember, PLL is just one example that POR waits for. The others PLL waits for are:<br>
- Power rails stabilization: stable voltages must be applied for each IPs.<br>
- Watchdog: check watchdog circuit that completes initializing to start checking CPUs.<br>
- Fuse and OTP: ready for reading Fuse/OTP options for booting<br>
- Initial state of thermal sensor: check thermal that is ready for checking temperature.<br>
</div>
<div style="margin:40px 0;"></div>

---
### ✅ Exit of POR and Transition to BOOTROM
---

Once again, POR looks at the internal state of SoC to ensure stable operation.  
And if **PMIC changes the reset pin to HIGH**, POR **exits the reset state**.  
<span class="highlight">PMIC changes the reset pin **when SoC voltages are stable**.</span>
<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> reset pin's change to high state is just one of the condition for exiting reset state.<br>
I mentioned that POR waits for many conditions in the previous subtopic.<br>
Reset state is exited when all conditions are satisfied.<br>
</div>