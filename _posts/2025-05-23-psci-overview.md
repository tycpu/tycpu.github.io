---
layout: post
title: "PSCI Overview (psci_cpu_on, system_off)"
date: 2025-05-23 12:00:00 +0900
tags: [psci, tf-a, power-management, smc]
---

## ⚡ PSCI Overview: `psci_cpu_on`, `system_off`, etc.

---

### 1. What is PSCI?

PSCI (Power State Coordination Interface) is the official power management interface defined by ARM.

Before PSCI, each ARM SoC vendor implemented their own way to control power (e.g., turning CPUs on/off, suspend, reboot).  
This led to **platform(kernel)-dependent kernel code** and made maintenance across devices very difficult.

To solve this, ARM introduced PSCI, which runs in **EL3 (typically via Trusted Firmware-A)** starting from ARMv7.  
The kernel or bootloader accesses PSCI through the **SMC (Secure Monitor Call)** instruction.

In summary:

> PSCI provides a **standardized way** for the OS to request power control actions via SMC to EL3.

---

### 2. PSCI Function Overview

Each PSCI function is identified by a **function ID defined by the SMCCC (SMC Calling Convention)**.  
These functions are used to manage CPU and system power states.

Here are the major PSCI functions:

#### 🔹 `psci_cpu_on`

- **Purpose**: Turn on a secondary CPU (used in SMP bring-up)
- **Parameters**:
  - `target_cpu`: MPIDR of the target CPU
  - `entry_point`: Address where the target CPU will start execution
  - `context_id`: Optional context value
- **Flow**:
  1. Kernel or bootloader calls `psci_cpu_on()`
  2. Sends request to EL3 (TF-A) via SMC
  3. TF-A powers on the CPU and jumps to `entry_point`

#### 🔹 `cpu_suspend`

- **Purpose**: Enter idle or low-power state
- **Flow**:
  1. Kernel calls `cpu_suspend()` during idle
  2. TF-A puts CPU into sleep state
  3. Wake-up events (IRQ/timer) resume the CPU

#### 🔹 `system_off`

- **Purpose**: Power off the system gracefully
- **Flow**:
  1. Kernel or userspace triggers shutdown
  2. TF-A powers off the system (usually via PMIC)

#### 🔹 `system_reset`

- **Purpose**: Reboot the system
- **Flow**:
  1. Kernel calls reboot
  2. TF-A triggers a hardware reset

#### 🔹 `psci_features`

- **Purpose**: Query which PSCI functions are supported
- **Example**:
  ```c
  psci_features(PSCI_CPU_ON);
  ```

#### 🔸 `Common Function IDs (SMCCC)`
  ```c
  #define PSCI_CPU_ON         0x84000003
  #define PSCI_CPU_SUSPEND    0x84000001
  #define PSCI_SYSTEM_OFF     0x84000008
  #define PSCI_SYSTEM_RESET   0x84000009
  ```
To call them:
  ```c
  #include <linux/arm-smccc.h>

  arm_smccc_smc(PSCI_CPU_ON, cpu_id, entry, context, 0, 0, 0, 0, &res);
  ```


### 3. PSCI Flow in TF-A
Even though PSCI is called by the kernel (EL1), the actual logic runs in EL3 (TF-A).  

🔧 Platform-Specific Implementation  
The real power control logic is implemented by each vendor, using the plat_psci_ops structure:
  ```c
  // Platform code (e.g., plat/board/plat_pm.c)
  static const plat_psci_ops_t my_platform_psci_ops = {
      .cpu_on        = my_cpu_on_impl,
      .cpu_off       = my_cpu_off_impl,
      .system_off    = my_system_off_impl,
      .system_reset  = my_system_reset_impl,
  };

  const plat_psci_ops_t* plat_setup_psci_ops(...) {
      return &my_platform_psci_ops;  // Register platform-specific handlers
  }
  ```
Then, PSCI uses this interface:
  ```c
  // TF-A common code
  const plat_psci_ops_t *psci_ops;

  int psci_cpu_on(u_register_t target_cpu, ...) {
      return psci_ops->cpu_on(target_cpu, ...);
  }
  ```

🧠 Summary  
PSCI is just a standard interface between kernel and TF-A.  
Actual hardware-specific operations must be implemented in vendor platform code via plat_psci_ops.


### 4. How the Kernel Uses PSCI
The Linux kernel uses PSCI to offload CPU and system power control to EL3.  
#### 🔧 SMC call example in the kernel
  ```c
  #include <linux/arm-smccc.h>

  arm_smccc_smc(PSCI_CPU_ON, target_cpu, entry_point, context_id,
              0, 0, 0, 0, &res);
  ```

#### 🔍 SMP bring-up example
  ```c
  // arch/arm64/kernel/smp_psci.c
  static const struct smp_operations smp_psci_ops = {
      .smp_boot_secondary = psci_cpu_on,
  };
  ```

When the kernel boots secondary cores, it uses psci_cpu_on(), which wraps the SMC call.  
This is completely different from plat_psci_ops, which is in EL3.  
🔸 smp_psci_ops is used by the kernel (EL1) to call PSCI  
🔸 plat_psci_ops is used by TF-A (EL3) to implement PSCI  

#### 📦 Device Tree Setup
The kernel will only use PSCI if it's defined in the device tree:
  ```dts
    psci {
      compatible = "arm,psci-1.0";
      method = "smc";
    };
  ```
If the "psci" node is present:
psci_dt_init() runs during boot  
PSCI version is checked using arm_smccc_version()  
Kernel prepares function IDs for use  


#### ✅ Summary
- PSCI provides a standard power management interface between the OS and platform firmware
- The kernel calls PSCI via SMC instructions using defined function IDs
- TF-A implements the PSCI handler in EL3
- Vendors must implement actual power control logic via plat_psci_ops
- The kernel must have "psci" node in the Device Tree to enable PSCI support

---
