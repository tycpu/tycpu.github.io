---
layout: post
title: "Transition from EL3 to EL2/EL1"
date: 2025-05-22 12:00:00 +0900
tags: [el3, el2, el1, tf-a, context-switch]
---

## 🔁 Transition from EL3 to EL2/EL1

---
### 🧭 The last role of EL3: When, and why this layer is transferred
---

EL3 is really important for **security and stable initialization.**  
But <span class="highlight">**it doesn't provide general execution environment like OS and hypervisor.**</span>  
So, we have to know about the role of each exception layer.

| Exception Level | Role |
|------------------|------|
| EL3 | Make up security with TrustZone and initialize |
| EL1 / EL2 | Execute OS   i.e) Linux |

EL3 gives control to EL1 or EL2(depending on platform configuration) when the following conditions are met:

<span style="color:blue">1) </span>**SoC is initialized stably** → can use DRAM, MMU, cache  
<span style="color:blue">2) </span>**Secure / Non-secure world are divided** → GIC, MMU setting completed  
<span style="color:blue">3) </span>**OS / Bootloader is ready to execute** → BL33 loaded  
<span style="color:blue">4) </span>**Registers like `SPSR_EL3`, `ELR_EL3` are ready**  


<div style="margin:40px 0;"></div>

---
## 🔧 The minimal setting for transition
---

When EL3 has to give control to EL1 or EL2,  
three registers must be set(SP for target EL should also be prepared.):

| Register | Purpose |
|----------|---------|
| `SCR_EL3` | for Secure → Non-secure state + instruction set architecture |
| `SPSR_EL3` | Next EL’s CPU mode, interrupt state, stack pointer behavior |
| `ELR_EL3` | Target address for control transfer |

### 🔹 SCR_EL3 major fields

| Field | Description |
|-------|-------------|
| NS | Set to 1 to switch to Non-secure World |
| RW | Set to 1 to use AArch64 (0 = AArch32) |

### 🔹 SPSR_EL3 major fields

| Field | Description |
|-------|-------------|
| M[3:0] | Mode (e.g. `0b0101 = EL1h`, `0b0100 = EL1t`, `0b1001 = EL2h`) |
|        | → Determines Exception Level and SP selection |
| D/A/I/F | Debug, SError, IRQ, FIQ disable flags (1 = disable) |
| SP | Selects SP_ELx or SP_EL0 (based on mode) |


<div style="margin:40px 0;"></div>

---
## 🚀 ERET: Not just a return
---

`ERET` is not just an instruction for PC return.  
It is an instruction for **transitioning between Exception Levels** and **restoring context with set registers.**

| Step | Description |
|------|-------------|
| 1) | Jump to address in `ELR_EL3` |
| 2) | Restore CPU state from `SPSR_EL3` |
| 3) | Switch to AArch64 (if `SCR_EL3.RW = 1`) |
| 4) | Enter Non-secure World (if `SCR_EL3.NS = 1`) |

| Step | Description |
|------|-------------|
| 1) | Set SCR_EL3(NS,RW...) |
| 2) | Set PSTATE(mode, interrupt mask) for next EL with SPSR_EL3 |
| 3) | Store jump address with ELR_EL3 |
| 4) | Jump Non-secure World with eret(if `SCR_EL3.NS = 1`) |

### 💡 Example code(jumps EL3 to non-secure)
```asm
MOV X0, #(1 << 0 | 1 << 10) // NS=1, RW=1 (Non-secure, AArch64)
MSR SCR_EL3, X0

MOV X1, #(0b1001 | (0b1111 << 6)) // EL2h, all interrupts masked
MSR SPSR_EL3, X1

LDR X2, =0x80080000 // EL2 entry point
MSR ELR_EL3, X2

ERET // Transition!
```

### ❗️ Transition considerations

<span style="color:blue">1) </span>Incorrect SCR_EL3 or SPSR_EL3 setting → transition fails  
<span style="color:blue">2) </span>Secure resource config must be complete before transition  
<span style="color:blue">3) </span>Memory/cache must be set up properly (e.g. MMU state)  


<div style="margin:40px 0;"></div>

---
## 🔐 Security: Secure → Non-secure
---

| Concept | Secure World | Non-secure World |
|---------|--------------|------------------|
| Access to secure DRAM | ✅ 가능 | ❌ 불가 |
| Access to secure interrupt | ✅ 가능 | ❌ 불가 |
| Access to TEE, cryptographic key | ✅ 가능 | ❌ 불가 |

### ✅ Protection mechanisms

| Type | Description |
|------|-------------|
| Memory Protection (TZASC) | Define secure DRAM area, cause fault on unauthorized access |
| Interrupt Protection (GIC) | Route secure/non-secure interrupts separately |
| Peripheral Access Control (TZPC) | Assign access policy to devices (e.g. UART0 secure only) |

---

### 🔄 Returning to Secure World

Returning is only possible using <span class="highlight">**`SMC` (Secure Monitor Call)**</span>:

- Called from Non-secure EL1/EL2
- Traps to EL3 (Secure Monitor handler)
- Handler checks and processes request securely


<div style="margin:40px 0;"></div>

---
## 📦 Code-based flow (TF-A: `bl31`)
---

### 🔹 1) Transition entry: `bl31_main()`

```c
void bl31_main(void)
{
    ...
    cm_prepare_el3_exit(NON_SECURE);
    ...
    el3_exit();
}
```

### 🔹 2) Register setup: cm_prepare_el3_exit()
```c
void cm_prepare_el3_exit(uint32_t security_state)
{
    ...
    write_scr_el3(scr_val);
    cm_set_next_eret_context(security_state);
    cm_set_elr_el3(security_state, entry_point);
    ...
}
```
### 🔹 3) Final transition: el3_exit in assembly
```asm
el3_exit:
    ldr x0, =cm_get_context(NON_SECURE)
    bl  restore_el3_sysregs   // Restore system registers
    bl  cm_exit_context       // Final context handling
    eret                      // Transition to EL1 or EL2
```

### ✅ Final summary flow
```text
bl31_main()
 └─ cm_prepare_el3_exit()       // Register setting
 └─ el3_exit()                  // Restore context → ERET
       └─ eret                  // EL3 → EL1 or EL2
```
