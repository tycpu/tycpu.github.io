---
layout: post
title: "Common Kernel Panic Messages & What They Mean"
date: 2025-06-24 12:00:00 +0900
tags: [linux, kernel, kernel panic, debugging]
---

---
## 🚀 Concepts of Each kernel log messages
---

### 1) "Unable to handle kernel NULL pointer dereference"
#### Meaning
This message means kernel code accesses to NULL(0x0) address or nearby that is protected.

#### Example  
``` yaml
    Unable to handle kernel NULL pointer dereference at virtual address 00000000
    pgd = c0004000
    [00000000] *pgd=00000000
    Internal error: Oops: 5 [#1]
    CPU: 0    Tainted: G        W
    PC is at my_driver_probe+0x2c/0x80
```

#### Reason  
- access to areas that are not initialized  
- dosen't check NULL before access to pointer in structure  
- dosen't clean-up after resource allocation during drive probe()  

#### Tips for analyzing
- check function name with "PC is at ~~" and offset.  
    we can know error function and specific location in this function with this message.  
- there are many cases that are with NULL pointer parameter.  
    So, try to check other functions that are in call trace chain.  

### 2) "Kernel BUG at...:
#### Meaning
This message is generated when BUG() macro is called.

#### Example
``` yaml
    kernel BUG at fs/buffer.c:1285!
    invalid opcode: 0000 [#1] SMP
    CPU 0
    Modules linked in: ...
    RIP: 0010:[<ffffffff81234567>] buffer_insert_list+0x97/0xa0
```

#### Reason
- BUG_ON is a kind of condition for ASSERT in kernel. It is for integrity.  
- Lock state inconsistency. Attempt to wrong unlock.  
    example)  
    ``` c
        spin_unlock(&lock); // BUG: unlock without prior lock
        spin_lock(&lock);
        spin_unlock(&lock);
        spin_unlock(&lock); // BUG: double unlock
    ```
- List corruption (prev/next pointer invalid). pointer inconsistency  
    example)  
    ``` c
        list_del(entry);
        list_del(entry); // BUG: double delete
    ```
- Memory allocator internal error  
    example)
    ``` c
        kfree(ptr);
        kfree(ptr); // double free → allocator internal error
    ```

#### Tips for analyzing
- find the location of calling BUG() with git grep  
- check the sequence of call trace  

### 3) "No init found. Try passing init=..."
#### Meaning
There is a fail during executing init process. Kernel can't find the first process in user space.

#### Example
``` yaml
    RAMDISK: Couldn't find valid RAM disk image starting at 0.
    No init found. Try passing init= option to kernel.
    Kernel panic - not syncing: No init found.
```

#### Reason
- crash in init binary  
- root filesystem mount fault  
- problem in initramfs or initrd  

#### Tips for analyzing
- check boot args (root=, init= parameters)  
- initramfs rebuild (dracut -f or update-initramfs -u)  
- check minimal shell with booting test(init=/bin/bash)  


### 4) "Kernel panic - not syncing: Fatal exception"
#### Meaning
There is a critical exception for system. So kernel occurs PANIC.

#### Example
``` yaml
    Kernel panic - not syncing: Fatal exception
    CPU: 0 PID: 123 Comm: insmod Not tainted 5.4.0-42-generic #46-Ubuntu
    Call Trace:
    dump_stack+0x6d/0x8b
    panic+0x101/0x2e3
    oops_end+0xdf/0x100
    ...
```

#### Reason
- NULL pointer dereference with panic_on_oops=1 option  
- BUG() call  
- hardware fault - unrecoverable exception  

#### Tips for analyzing
- check call trace before panic message  
- check module load sequence with dmseg  
- check the location with low-debug tool like trace32  


### 5) "Kernel panic - not syncing: Attempted to kill init!"
#### Meaning
The init process(PID 1) is end. Can't operate system.

#### Example
``` yaml
    Kernel panic - not syncing: Attempted to kill init! exitcode=0x0000000b
    CPU: 0 PID: 1 Comm: init Not tainted 3.10.0-1160.el7.x86_64 #1
    Call Trace:
    ...
```

#### Reason
- init process crash  
- impairment root filesystem(init fail)  
- static init binary misconfiguration like busybox  

#### Tips for analyzing
- analyze exitcode(segfault, missing lib...)  
- check the state of root fs mount  

### 6) "VFS: Unable to mount root fs"
#### Meaning
fail to mount root filesystem

#### Example
``` yaml
    VFS: Unable to mount root fs on unknown-block(0,0)
    Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

#### Reason
- wrong root= parameter  
- no include storage driver in initramfs  
- mssing ext4/xfs driver  

#### Tips for analyzing
- check boot args(root =)  
- check whether the storage device driver built-in  

### 7) "general protection fault: 0000 [#1] SMP"
#### Meaning
invalid memory or register access in kernel mode

#### Example
``` yaml
    general protection fault: 0000 [#1] SMP
    CPU: 0 PID: 1234 Comm: myapp Tainted: G    B D W
    RIP: 0010:[<ffffffff81234567>] faulty_func+0x17/0x60
```

#### Reason
- NULL ponter dereference  
- field offset miscalculation of structure  

#### Tips for analyzing
- check RIP address function and offset  
- check caller function parameter with call trace  
- check struct state with crash tool  
