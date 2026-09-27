# Project Conclusion: The "Performance Floor" of Modern AI

## 1. Hardware Architecture & OS Tuning

The foundation of this project relied on stripping away every unnecessary cycle. By deploying antiX-26 (Core), we reduced 
the idle system footprint to just 231 MB, reclaiming approximately 15% of the total physical memory compared to the original 
Windows XP baseline.
* Instruction Set Optimization: By building llama.cpp for the i686 target, we let its native-architecture auto-detection pick up the Prescott core's SSE3 support, bridging the gap to modern tensor math without hand-tuned flags.
* Logical Parallelism: Utilizing Hyper-Threading allowed the system to perform ~1.6x more work than the elapsed "real" time, effectively maximizing the pipeline of the single physical core.

## 2. Benchmarking Results: The Scaling Law of Legacy Silicon

The results below define the "breaking point" for consumer hardware from the mid-2000s.
| Model | Parameters | Quantization | Speed (t/s) | Status |
| ----- | ---------- | ------------ | ----------- | ------ |
| Qwen2.5-0.5B | 0.5 Billion | Q4_K_M | 0.9 t/s | Optimal |
| TinyLlama-1.1B | 1.1 Billion | Q4_K_M | 0.5 t/s | Functional | 
| SmolLM2-1.7B | 1.7 Billion | Q4_K_M | 0.0 t/s | Failed (Swap Bound) | 

## 3. Key Engineering Insights

The primary bottleneck discovered was not raw compute power, but Memory Bandwidth.
* The "Swap Wall": Once the model size exceeded ~1.5B parameters, the kernel was forced to use the 8GB Swap Partition. Because this swap was hosted on a USB 2.0 bus, I/O wait times consumed 70% of the session, turning a 5-minute task into a 46-minute wait.
* Throughput Density: This project demonstrates that for legacy "Edge AI," the most critical metric is Throughput per unit of physical RAM. On the Pentium 4, the 0.5B parameter model provides the best "intelligence-to-latency" ratio.

## 4. Future Directions: Breaking the Wall

To move past the "SmolLM2 Wall" on this hardware, two potential upgrades were identified:
  1. SATA SSD Migration: Replacing the USB-based swap with a physical IDE-to-SATA SSD would reduce I/O latency by a magnitude, potentially making 1.5B+ models usable.
  2. Model Distillation: As distilled 1B parameter models become more advanced in 2026, the a450n could serve as a dedicated, air-gapped reasoning node for simple text extraction tasks.
