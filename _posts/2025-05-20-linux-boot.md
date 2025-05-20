# How Linux Boots on ARM64: From Reset to `start_kernel()`

## 🔍 Overview
Brief explanation of the boot sequence this post will cover, and why understanding this stage matters for kernel developers.

---

## 🧱 1. Power-On Reset and the Boot ROM
- What happens when the SoC powers on

Suppose that there is a SoC which would get some power from PMIC.  
If SoC gets power, a particular circuit will be operated for reset.  
The name of circuit is **Power-On Reset (POR).**

---

If SoC starts to get some power, the **reset pin** would be **low (active)** state.  
Reset pin is some kind of signal, and if the state of reset pin changes to high,  
CPU jumps to **reset vector** immediately.  
Reset vector has the starting address of **boot ROM code**.

---

The reason why there is a POR circuit is for initializing all digital blocks like CPU and memory controller.  
There are many important things for operating SoC — like **registers, clocks, and voltages.**  
To explain POR circuit simply, this circuit **waits for stable operating conditions** for everything mentioned in the previous line.

---

For example:

- POR circuit waits for reset of all previously written registers.
- It also waits for **clock stabilization**.

SoC operation starts with **XTAL**, which is a low but stable clock oscillator.  
POR waits until **lock in PLL (Phase-Locked Loop)**.  
PLL is a kind of **frequency multiplier**.  
This will be locked after stabilization.  
So in order to operate normally, POR checks the **lock signal from PLL**.

---

Once again, POR looks at the internal state of SoC to ensure stable operation.  
And if **PMIC changes the reset pin to HIGH**, POR **exits the reset state**.  
PMIC changes the reset pin **when SoC voltages are stable**.


- Role of the Boot ROM (BL0)

I mentioned before that if the reset pin gets into a high state and the POR exits the reset state,  
the CPU jumps to the reset vector.  
The reset vector has the starting address of BOOTROM.

---

BOOTROM is a code that is hard-coded with assembly and is read-only.  
This code is custom-made for vendors like Samsung and Qualcomm.

The reason why this code is written in assembly is because,  
at this time, there is no MMU (used for virtual addressing), no cache, and no stack.  
So the developer can't write BOOTROM code using virtual addresses or assume register initialization.

Additionally, BOOTROM is operated in a very limited space in the SoC, like SRAM.  
This is due to booting speed and security reasons.  
If this code is too heavy, booting becomes slower and the system becomes more exposed to potential attacks.

---

We have to understand why BOOTROM is needed.  
This code is not only the starting point of booting.

---

**First**, I said that this code is the starting point of booting.  
But we have to focus on the word **"starting."**  
If this point can't start, the SoC cannot operate at all.  
So this code is always mandatory and **must be executed no matter what**.

---

**Second**, if this code is hacked, the SoC also cannot be operated.  
So **security is extremely important** for BOOTROM code.  
Fortunately, this code is hard-coded, so it’s very difficult to hack it.

Also, BOOTROM code **verifies the digital signature** of all bootloaders.  
BOOTROM has the **vendor’s public key** embedded in advance, and it uses it to verify the authenticity of bootloaders.

---

**Third**, the CPU needs an entry point for the next bootloader, like BL1.  
BOOTROM provides this entry point to the CPU.

This is made possible with a **boot mode pin**.  
The boot mode pin is a kind of hardware, like a signal switch.  
Using this signal, BOOTROM can determine **the location of the boot source**.

Additionally, BOOTROM performs some simple initial tasks like **SRAM initialization and watchdog disable**.  
After that, other bootloaders can be executed in **a more advanced environment** like DRAM and C language.

- Jump to TF-A (Trusted Firmware-A)

We need to execute the next bootloader, called **BL1**.  
BOOTROM is responsible for loading this bootloader before executing it.

---

### 🔧 How does BOOTROM know where to load BL1 from?

The boot source is determined by hardware option values like:

- **boot mode pin**
- **fuse**
- **OTP**

These are set by each vendor during board or SoC design.  
For example, the boot mode pin might work like this(this is for the location which BOOTROM gets BL1 from):

```
00 → eMMC booting  
01 → SPI NOR booting  
10 → USB booting (for debugging)
```

---

### 📥 Where is BL1 loaded?

BL1 is loaded into **SRAM**.

The reason for this is that the main memory (DRAM) is not initialized yet.  
DRAM initialization happens later in **BL2**.  
Since SRAM consumes less power and is directly accessible early in the boot process, it's used for BL1.

Typical memory layout:

```
SRAM address:           0x0004_0000 ~ 0x0004_FFFF  
BL1 loading address:    0x0004_0000
```

So, BOOTROM reads `bl1.bin` from the selected boot source (eMMC or SPI)  
and copies it into the SRAM address. Then it jumps to that address to execute BL1.

---

### 🚪 How does BOOTROM find the entry point of BL1?

There are two main methods:

#### ✅ 1. Fixed Entry Address

A simple method: the entry address is fixed in hardware design.

```
BL1 Entry Address = 0x00040000
```

BOOTROM just copies the image and jumps to that address directly.

---

#### ✅ 2. Image Header

Another method is using a metadata header embedded in the image:

```c
typedef struct {
    uint32_t magic;         // Magic number for validation
    uint32_t entry_point;   // <-- This is key
    uint32_t image_size;
    uint32_t checksum;
} bl1_header_t;
```

BOOTROM reads this header, parses the `entry_point` field, and jumps to it.

In many platforms, both methods are used together:
- Fixed loading address (e.g., `0x40000`)
- Dynamic entry point (e.g., `0x40080` from header)

---

### 🏁 How does BOOTROM jump to BL1?

Once the entry point is determined, BOOTROM sets it into a register  
and uses a branch instruction to jump:

```asm
ldr     x0, =0x00040000  
br      x0
```

At this point, CPU begins executing BL1’s first instruction.

---

### 🔐 Secure Boot Consideration

If Secure Boot is enabled, BOOTROM will:

- **Verify the digital signature** of `bl1.bin`
- If the signature is invalid, it will **not jump** and halt or retry from another media

---

### 🔄 Boot Flow Summary

1. BOOTROM gets power and runs  
2. Reads setting for loading BL1 from pin/fuse  
3. Loads BL1 from selected storage (eMMC, SPI, USB, etc.)  
4. Copies BL1 image into SRAM  
5. Gets BL1 entry point (fixed or via header)  
6. Sets PC register to entry point  
7. Executes `branch` to start BL1  
8. Now, **BL1 is running**

---

## 🛡️ 2. TF-A: Secure World Initialization
- Stages: BL1 → BL2 → BL31
- EL3 responsibilities
- Transition from EL3 to EL2 or EL1
- PSCI overview (psci_cpu_on, system_off)

---

## 🚀 3. U-Boot: Bootloader Duties
- U-Boot loads kernel image (`Image`) and Device Tree (`.dtb`)
- Passes bootargs and memory info to the kernel
- Commands like `booti`, `bootm`, `fdt addr`, etc.

---

## 🧠 4. Entering the Linux Kernel: `head.S`
- What happens in `arch/arm64/kernel/head.S`
- MMU enabling, early page tables
- Stack setup, zeroing `.bss`, physical to virtual transition
- Symbol: `__primary_switched`

---

## 🔧 5. The Handoff: `bl start_kernel`
- Final step in assembly before jumping to C code
- Where `start_kernel()` is defined
- Why this is the logical cutoff for boot sequence analysis

---

## 📌 Summary & What’s Next
- Key stages recap: TF-A → U-Boot → head.S → `start_kernel()`
- Tease next post: inside `start_kernel()` itself