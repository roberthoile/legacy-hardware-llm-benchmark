# Legacy Hardware LLM Benchmark
### *Analyzing LLM Inference on 32-bit x86 Architecture (Pentium 4)*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Architecture: x86-32](https://img.shields.io/badge/Architecture-x86--32-blue)](https://en.wikipedia.org/wiki/IA-32)
[![OS: antiX-26](https://img.shields.io/badge/OS-antiX--26-purple)](https://antixlinux.com/)

## 1. Project Philosophy
In an era of hyper-scale AI and H100 clusters, software efficiency is often overlooked. This project explores the "performance floor" of modern Large Language Models (LLMs) by implementing a functional AI assistant on a 20-year-old consumer desktop. This investigation serves to demonstrate:
* **System Optimization:** Squeezing performance out of single-core, 32-bit hardware.
* **Resource Awareness:** Managing a strict 2GB RAM budget for both OS and Inference.
* **Legacy Preservation:** Repurposing "e-waste" into functional, localized AI edge nodes.

---

## 2. Hardware Specifications: "The Relic"
The target machine is a mid-2000s **HP Pavilion a450n**, representing the peak of the Windows XP generation.

| Component | Specification |
| :--- | :--- |
| **CPU** | Intel Pentium 4 3.00GHz (Prescott Core, SSE3 support) |
| **RAM** | 2 GB DDR (Maxed for 32-bit architecture) |
| **Storage** | 160GB Ultra ATA (IDE/PATA) Hard Drive |
| **Graphics** | Integrated Intel Extreme Graphics 2 |
| **Architecture** | 32-bit (IA-32) |

---

## 3. The Technical Challenge
Running modern LLMs on this hardware presents three primary hurdles:
1.  **32-bit Dependency:** Most modern AI toolchains (PyTorch, TensorFlow) have dropped 32-bit support.
2.  **Memory Constraints:** A 1.1B parameter model typically requires 4GB+ of RAM; this implementation utilizes 4-bit quantization and aggressive swap-management to fit within 2GB.
3.  **Instruction Sets:** Lack of AVX/AVX2 instructions requires a custom compilation of the inference engine to utilize the Prescott's SSE3 set for matrix multiplication.

---

## 4. Implementation Stack
* **OS:** antiX-26 (Core) - A systemd-free, 32-bit Debian-based Linux.
* **Inference Engine:** `llama.cpp` (Custom 32-bit build with SSE3 flags).
* **Model:** `TinyLlama-1.1B-Chat-v1.0` (GGUF, Q4_K_M quantization).
* **Monitoring:** `htop` for RAM/Swap tracking; `lm-sensors` for thermal monitoring.

---

## 5. Benchmarking Metrics
This repository tracks and compares the following performance data:

* **TTFT (Time to First Token):** Latency between the prompt and the start of the response.
* **TPS (Tokens Per Second):** The sustained generation speed.
* **Thermal Delta:** CPU temperature rise during 100% load on the Prescott core.
* **VRAM vs System RAM:** Tracking the footprint in a non-GPU environment.

---

## 6. How to Reproduce
1.  **Prepare the OS:** Install antiX-26 Core (32-bit).
2.  **Toolchain Setup:**
    ```bash
    sudo apt update
    sudo apt install build-essential cmake git htop
    ```
3.  **Build Inference Engine:**
    ```bash
    git clone [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)
    cd llama.cpp
    mkdir build && cd build
    cmake .. -DLLAMA_NATIVE=OFF -DLLAMA_SSE3=ON
    make
    ```
4.  **Download Model:**
    Place a 4-bit quantized GGUF model (e.g., TinyLlama 1.1B) into the `/models` directory.

---

## 7. Reflections for the Job Search
As a Lead Software Engineer, this project reinforces the importance of **constrained environment engineering**. Whether developing for edge devices, IoT, or optimizing cloud spend, the ability to profile and optimize at the system level remains a critical, evergreen skill.

---

## Author
**Robert "Bob" Hoile**
*Vice President & Lead Software Engineer*
