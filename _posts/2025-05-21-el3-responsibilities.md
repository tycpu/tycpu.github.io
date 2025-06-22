---
layout: post
title: "EL3 Responsibilities"
date: 2025-05-21 12:00:00 +0900
tags: [tf-a, el3, secure-world, boot]
---

## 🛡️ EL3 Responsibilities

---
### 🔸 What is EL3?
---

First, we have to know about what is the EL3.  
EL3 is the highest permission level which is imported from ARMv8-A.  

| Exception Level | Description                            |
|-----------------|----------------------------------------|
| EL0             | User level for user application        |
| EL1             | Kernel level for OS like Linux         |
| EL2             | Hypervisor level for virtualization    |
| EL3             | Secure monitor level for world control |

<span class="highlight">**EL3 can access secure world.**</span>  
And EL3 divides the area between secure world and non-secure world.  
So, **other Exception Levels like EL1 and EL2 can access to non-secure world only.**

The main reason why only EL3 can access to secure world is for <span class="highlight">**TrustZone**</span>.  
If all exception levels can access secure world, there might be happened critical security problems.  
And then SoC can't be operated because of hacking or bug.  
So accessing to secure world is really risky, but something has to control and initialize for SoC.  
For that reason, **only EL3** can access to secure world like PSCI or SMC.


<div style="background:#f0f8ff; border-left:4px solid #007acc; padding:10px; margin:15px 0;">
💡 <strong>Note:</strong> Secure World is a secure execution environment in ARM TrustZone architecture.<br>
It handles sensitive operations like cryptographic processing, <strong>secure key storage</strong>, and authentication.<br>
But EL3 is not only for security. It also manages power like PSCI.<br>

Purpose: Isolate and protect security-critical tasks from the normal operating system.<br>
Runs: A lightweight Trusted OS (e.g., OP-TEE)<br>
Contains: Secure applications, cryptographic libraries, hardware access control logic<br>
Accessed via: Secure Monitor Call (SMC) from the Normal World<br>
Controls: Access to secure memory and peripherals, enforces system-wide security policies<br>
</div>

<div style="margin:40px 0;"></div>

---
### 🔸 What EL3 Initializes
---

The main goal of EL3 is preparing for execute secure + non-secure environment for system.  
And BL31 initializes EL3.

| Category             | Role                                                                 |
|----------------------|----------------------------------------------------------------------|
| Secure world         | Activate TrustZone, set up NS bit                                    |
| EL migration         | Migrate safely to EL2 or EL1                                          |
| PSCI initialization  | Register interface for CPU On/Off, reset                             |
| SMC handler          | Handle calls between non-secure and secure world                     |
| GIC setup            | Split interrupts for secure/non-secure                               |
| Context save         | Save initial registers for EL1/EL2                                   |
| Memory permission    | Divide secure and non-secure DRAM                                    |

<div style="margin:40px 0;"></div>

---
### 🔸 Why EL3 Runs on BL31
---

EL3 is important, so someone says "Faster is better. So EL3 would be operated on BL1 or BL2"  
But this is not right.  
For executing EL3, there are a few things that EL3 needs before.

| Requirement       | Reason                                                       |
|-------------------|--------------------------------------------------------------|
| DRAM              | EL3 needs to be loaded on DRAM(code size, with OP-TEE, etc.) |
| Secure regions    | Secure/non-secure world also loaded on DRAM                  |
| Device tree       | Loaded from BL2                                              |
| PSCI & SMC        | Handler setup possible only after above                      |

So we can know the reason why EL3 is executed on BL31.  
These are all possible **after BL2**.

Additionally, EL3 is not only for booting. It is also needed during system runtime.  
For example, PSCI and system reset are handled in EL3 secure firmware and triggered via SMC from non-secure world.  
This interface requires EL3.  
BL31 runs in a stable, pre-initialized environment (after BL1 / BL2).  
So for calling SMC interface, **EL3 must be operated on BL31**.

<div style="margin:40px 0;"></div>

---
### 🔸 Transition from EL3 to EL2/EL1
---

We also have to know how EL3 can jump to EL2 or EL1 — i.e., transition from secure to non-secure world.  
This is done using `eret`, but some registers must be set beforehand:

| Register    | Role                                             |
|-------------|--------------------------------------------------|
| SCR_EL3     | Secure config (NS bit: 0 = Secure, 1 = Non-secure) |
| SPSR_EL3    | Status of next EL                                |
| ELR_EL3     | Address to jump (PC)                             |
| SP_ELx      | Stack pointer for target EL                      |

#### 🔹 Example transition code

```asm
// set up EL2 to Non-secure
mov x0, #(1 << 0)         // NS = 1
msr SCR_EL3, x0

// EL2 mode, AArch64, disable interrupt
mov x1, #(0b1001 << 6)    // EL2h, AArch64
msr SPSR_EL3, x1

// set PC to transition
ldr x2, =0x80000000
msr ELR_EL3, x2

eret  // ← Exception Return to EL2

```

<div style="margin:40px 0;"></div>

---
### 🔸 Why PSCI & SMC Are Handled in EL3
---

PSCI (Power State Coordination Interface) is the official ARM interface for CPU power control.  
CPU control and power management are really sensitive tasks for operation,  
so it must be run in secure world.  

SMC (Secure Monitor Call) is used to request EL3 services from EL1/EL2.  
Example from Linux kernel:  

``` C
SMC #0x84000000  // → psci_cpu_on()
```
This must be handled in EL3.  
Therefore, PSCI and SMC both represent the interface between secure and non-secure worlds, and must be managed within EL3.  


We can summarize EL3 flow for booting like this:
![EL3 Boot Flow](/assets/el3.png)
