# Phase 1: OS Installation
## The Starting Point - Hewlett-Packard Pavilion a450n

The initial audit confirmed this machine as a survivor of the mid-2000s desktop era, still bearing its original "Designed for Windows XP" chassis stickers and Certificate of Authenticity (COA).

### System Specifications (Baseline)
* **CPU:** Intel Pentium 4 3.00GHz (Prescott)
* **RAM:** 2.00 GB DDR (System Maximum)
* **OS:** Microsoft Windows XP Home Edition, Version 2002, Service Pack 3
* **Virtual Memory:** 2047 MB Static Pagefile

### Storage Configuration
Analysis via Disk Management revealed a dual-drive setup featuring a mix of original PATA and repurposed SATA hardware:

| Disk | Model | Interface | Capacity | Partitioning |
| :--- | :--- | :--- | :--- | :--- |
| **Disk 0** | Samsung SP1604N | IDE/PATA | 160 GB | **C:** (NTFS, 143GB) / **D: HP_RECOVERY** (FAT32, 5.5GB) |
| **Disk 1** | Fujitsu MHY2200BH | SATA* | 200 GB | **F:** (NTFS, 178GB) / Unknown (NTFS, 7.9GB) |

*\*Note: The Fujitsu drive is a 2.5" laptop unit, indicating a post-factory hardware modification to increase local storage.*

### Baseline Performance Metrics (Idle)
To establish a comparison point for the resource-constrained AI environment, the system's idle state was measured under its native Windows XP SP3 installation (measured 3 minutes post-boot via Task Manager):

* **CPU Usage:** 0-1% (Idle), with occasional background spikes to 7-8%
* **Commit Charge (Total Footprint):** ~370 MB (Peak: ~643 MB)
* **Physical Memory Available:** ~1.53 GB / 2.00 GB
* **Active Processes:** 41

*Note: Capturing this legacy baseline is critical for evaluating the efficiency gains of the stripped-down antiX-26 Linux kernel implemented in Phase 2.*
    
## The Pivot
While Windows XP SP3 is historically significant, it presents insurmountable barriers for modern Large Language Model inference:

1. **Resource Bloat:** The legacy OS consumes ~370MB of RAM and runs over 40 background processes while completely idle. In a strict 2GB memory budget, this overhead is unacceptable.
2. **Toolchain Incompatibility:** Modern AI inference engines (like `llama.cpp`) require modern C++ compilers and build tools (CMake) that are unsupported or highly unstable in the XP environment.
3. **Kernel Limitations:** We require low-level memory management techniques, such as compressed RAM swap (zRAM), which are native to modern Linux kernels but unavailable in the NT 5.1 kernel.

To reclaim system resources and provide a viable compilation environment, the system must be transitioned to a hyper-optimized, systemd-free 32-bit Linux distribution (antiX-26).

## The Gotchas

### 1. The Hyper-Threading Discovery
The BIOS audit revealed the Pentium 4 is a "Prescott" core featuring Hyper-Threading (HT) Technology. While physically a single-core CPU, it presents as two logical cores to the OS. This is a critical advantage for the AI compilation phase, as it will allow `llama.cpp` to utilize two threads instead of one.

### 2. Legacy BIOS USB Priority
The v2.51 BIOS does not feature a dedicated "USB Boot" category. Bootable USB drives are classified as generic hard disks.
* **Solution:** To boot from USB, the drive must first be elevated to the #1 position within the `Hard Disk Drives` sub-menu before it can be selected as the primary boot device in the main Boot Priority sequence.

### 3. The Ventoy / GRUB2 Failure
Modern multiboot tools like Ventoy rely on GRUB2 payloads that the 2004 American Megatrends BIOS cannot correctly parse in legacy mode. Attempting to boot Ventoy results in a drop to a `grub>` rescue prompt rather than the intended boot menu.
* **Solution:** Abandon multiboot tools. Flash the antiX ISO directly to the USB drive using Rufus, strictly enforcing an **MBR partition scheme** and **BIOS (non-UEFI)** target.

### 4. CMOS Battery / Time Sync Errors
During the live USB boot, logging in as root triggered a `unix_chkpwd` warning stating the "password changed in future." This is a symptom of a dead motherboard CMOS battery, causing the hardware clock to default to 2004 while the OS files are timestamped 2026. This requires NTP synchronization once networked to prevent compilation timestamp errors.

### 5. `cli-installer` Auto-Partitioning Bypass
The antiX text installer bypassed the "Auto-install" wizard for the legacy IDE drive, dropping the installation into the `cfdisk` manual partitioning utility.
* **Solution:** Manually purged existing NTFS partitions, created a dedicated 4GB Swap partition (Type 82), allocated the remainder as a bootable Primary partition, and manually wrote the table to disk.

### 6. Legacy GRUB Placement
The installer defaults to installing the GRUB bootloader to the root partition. On a pre-UEFI American Megatrends BIOS, this results in an unbootable system.
* **Solution:** GRUB must be explicitly installed to the **MBR (Master Boot Record)** of `/dev/sda` to successfully hand off the boot sequence.

### 7. Corrupted USB Partition Table (Physical Level)
The 128GB USB drive was initially detected as two unusable, unformatted logical drives. Standard formatting via Windows Explorer failed, suggesting a corrupted partition table or MBR.
* **The Gotcha:** This state often mimics a hardware failure, but it is frequently a software-level lock caused by previous high-level imaging tools (like BalenaEtcher or Ventoy).
* **Solution:** Used `diskpart` via the Windows Command Prompt to `clean` the entire disk. This zeroed the partition table, allowing the drive to be initialized as a single MBR volume before flashing with Rufus.

### 8. antiX Persistence Script Sensitivity
Even with a correctly flashed ISO, the antiX "Live-USB" scripts failed to automatically "claim" the 100GB persistence partition on `/dev/sda2`.
* **The Gotcha:** The antiX persistence scripts are highly literal. Without specific boot codes, the system will not look past the primary boot partition (`sda1`), even if a second partition is labeled correctly.
* **Solution:** Injected the "Golden Boot String" into the boot parameters: `plab=rootfs live_swap=force slab=swapon persist_static`. 
    * `plab=rootfs` forces the system to scan for a specific label.
    * `live_swap=force` combined with `slab=swapon` manually bridges the gap to the 8GB swap partition.
    
## The Outcome
The environment is now fully stabilized on the 128GB "refurbished" USB drive. By utilizing the `persist_all` flag in conjunction with our custom partition mapping, the system has successfully initialized the following high-capacity volumes:

* **System Layer:** 30GB `rootfs` loopback file on `/dev/sda2`.
* **User Layer:** 60GB `homefs` loopback file on `/dev/sda2`.
* **Memory Safety:** 8GB dedicated physical swap on `/dev/sda3`.

### antiX-26 Baseline Performance Metrics (Idle)
*Measured post-handshake via `top` and `free -m`:*

* **CPU Usage:** 4.5% (Idle)
* **Memory Footprint:** 231 MB 
* **Physical Memory Available:** 1.86 GB / 2.00 GB
* **Active Processes:** 1

The migration to antiX 26 Core has transformed the Pentium 4 from a struggling workstation into a lean, dedicated inference node. By reducing the idle memory footprint to a mere 231 MB, you have successfully reclaimed approximately 15% of the total system RAM that was previously consumed by OS overhead. This efficiency gain, coupled with a minimal 4.5% CPU idle load, ensures that nearly the entire 2.00 GB memory budget and the Prescott core's hyper-threaded cycles are reserved for the intensive computational demands of the LLM toolchain.
