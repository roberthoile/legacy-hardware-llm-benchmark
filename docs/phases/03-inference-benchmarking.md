# Phase 3: Inference & Benchmarking
With the environment stabilized and the toolchain verified, Phase 3 focuses on the primary objective: measuring the performance of Large Language Models on 20-year-old hardware.

---

## 1. Model Selection & Rationale
To find the "performance ceiling" of the Pentium 4 (Prescott), we have selected a range of models that fit within the 1.78 GB available RAM window. All models utilize Q4_K_M (4-bit) quantization to maximize speed without total loss of coherence.

| Model | Parameters | Quantization | Size (Disk) | Expected Role |
| ----- | ---------- | ------------ | ----------- | ------------- |
| Qwen2.5-0.5B | 500M | Q4_K_M | ~398 MB | High-speed baseline | 
| TinyLlama-1.1B | 1.1B | Q4_K_M | ~669 MB | Industry standard small-scale test |
| SmolLM2-1.7B | 1.7B | Q4_K_M | ~1.06 GB | The "Upper Limit" for 2GB RAM |

## 2. Testing Methodology
Each model will be subjected to the same standardized prompt to ensure consistency across benchmarks.

* Primary Metric: Tokens per second (t/s).
* Secondary Metric: Memory consumption (RES) and Swap pressure.
* Prompt: "Explain the concept of a mathematical limit in three simple sentences."
* Hardware State: Single-user CLI mode, 2 logical threads (HT enabled).

## 3. Baseline Execution Command
The following command structure will be used for all tests to ensure the prompt is processed identically:

```bash
./llama-cli -m models/[MODEL_NAME].gguf -p "Explain the concept of a mathemat
```
## 4. Test Results: 
### Qwen2.5-0.5B (The Baseline)
The first successful inference test on the Pentium 4 architecture.

#### Statistics
| Metric | Result |
| :--- | :--- |
| **Prompt Processing (PP)** | 1.1 t/s |
| **Token Generation (TG)** | 0.9 t/s |
| **Total Execution (Real)** | 2m 49.8s |
| **CPU User Time** | 5m 24.5s |
| **Status** | **SUCCESS** |

#### Observation
The benchmark confirms successful utilization of the Prescott's Hyper-Threading capability; the `user` time is nearly 2x the `real` time, indicating both logical cores were fully saturated. At ~0.9 t/s, the model is functional for non-real-time tasks. No thermal throttling or memory swapping was triggered during the 169-second run.

### TinyLlama-1.1B (The Industry Small-Scale Test)
Stepping up to a 1.1B parameter model provides a direct look at the scaling tax on the Pentium 4 architecture.

#### Statistics
| Metric | Result |
| :--- | :--- |
| **Prompt Processing (PP)** | 0.5 t/s |
| **Token Generation (TG)** | 0.5 t/s |
| **Total Execution (Real)** | 5m 56.1s |
| **CPU User Time** | 10m 59.2s |
| **Status** | **SUCCESS** |

#### Observation
The benchmark shows a predictable performance drop compared to the 0.5B baseline, but the system remains functional for isolated queries. Hyper-threading efficiency remains high, with `user` time nearly doubling the `real` elapsed time, proving the Prescott core is still effectively managing the instruction pipeline even under increased math complexity.

### SmolLM2-1.7B (The "Swap Wall")
This test was designed to push the hardware past its physical 2GB RAM limit to observe the impact of virtual memory on inference speeds.

#### Statistics
| Metric | Result |
| :--- | :--- |
| **Prompt Processing (PP)** | 0.3 t/s |
| **Token Generation (TG)** | 0.0 t/s |
| **Total Execution (Real)** | 46m 13.0s |
| **CPU User Time** | 13m 19.4s |
| **Status** | **FAILED (Swap Bound)** |

#### Observation
This model officially pushed the HP a450n past its limit. While the weights technically fit in the RAM/Swap window, the resulting I/O pressure on the USB 2.0 bus created an insurmountable bottleneck. The "Real" time was over 3x the "User" time, indicating the CPU spent the vast majority of the session in an I/O wait state. While the model's qualitative output was accurate, the speed is unusable for any practical application.
