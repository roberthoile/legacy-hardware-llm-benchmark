# Phase 2: The Toolchain Build

With the OS successfully stabilized on the persistent USB environment, Phase 2 focuses on transforming the lean antiX 
Core installation into a functional C++ development node. The primary challenge is balancing modern compilation 
requirements against 20-year-old 32-bit hardware constraints.

---

## 1. Environment Preparation
Before fetching source code, we must resolve the "Future Timestamp" Gotcha caused by the dead CMOS battery and pull 
the necessary build headers.

### System Clock Synchronization
Because the CMOS battery is failed, the system defaults to 2004. This prevents `apt` from validating SSL certificates. We must set the date manually before installing synchronization tools.

```bash
# 1. Manual jump-start (Required for SSL/APT to work)
date -s "2026-05-02 21:45:00"

# 2. Install and run ntpdate for precision sync
apt update
apt install ntpdate
ntpdate -u pool.ntp.org
```

### Essential Package Installation
With the clock synchronized, the system can now validate repository certificates.

```bash
apt update
apt install build-essential cmake git curl pkg-config
```

### Toolchain Verification
The antiX 26 repositories provided a surprisingly modern toolchain for this 32-bit architecture:
* **GCC Version:** 14.2.0 (Full C++20/C++23 support)
* **CMake Version:** 3.31.6

---

## 2. Hardware-Specific Optimization Strategy
The Pentium 4 "Prescott" architecture introduced **SSE3** instructions. To maximize inference performance, we must ensure the compiler targets these specifically.

* **Target Architecture:** `prescott`
* **Instruction Set:** SSE3, MMX, SSE, SSE2
* **Optimization Level:** `-O3`

---

## 3. Repository Setup & Build
We are targeting **llama.cpp** due to its minimal dependency tree and highly optimized CPU-only inference engine.

### Cloning Source
To leverage the 60GB persistent volume and avoid crowding the system partition, the project was cloned into the `/home` directory.

```bash
cd /home
git clone [https://github.com/ggerganov/llama.cpp.git](https://github.com/ggerganov/llama.cpp.git)
cd llama.cpp
```

### Compilation
The build was executed using two threads to utilize the Pentium 4's Hyper-Threading capability.

Note: An initial path error occurred due to moving the directory from /root to /home after configuration. This was resolved by deleting the build directory and re-running cmake .. from the new location to update the absolute paths in CMakeCache.txt.

```bash
mkdir build && cd build
cmake ..
cmake --build . --config Release -j 2
```

## 4. Build Metrics & Verification
The compilation completed successfully overnight, proving the stability of the persistent environment under high sustained load.

### Resource Utilization (Peak Build)
* Peak Swap Usage: ~11.6 MB (The 8GB physical swap partition on sda3 provided ample headroom).
* CPU Behavior: Hyper-Threading successfully utilized 100% of the physical core via two logical threads. dmesg confirmed no thermal throttling or OOM events during the 6-8 hour build.
* Build Time: originally estimated at 1 - 1.5 hours, triggered and ran overnight in under 8 hours

### Post-Build Resource Footprint
* Disk Impact: ~665 MB total (Source + Binaries) in /home/llama.cpp.
* System Root Impact: Minimal (< 600 MB total used on the system partition).
* Available Memory: ~1.78 GB available for inference (Post-build idle).

### Binary Verification
The build produced a functional llama-cli binary optimized for the i686 architecture with SSE3 support.
