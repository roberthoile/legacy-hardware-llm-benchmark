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
