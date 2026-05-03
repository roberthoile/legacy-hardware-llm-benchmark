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
```bash
git clone [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)
cd llama.cpp
```

