# Offline Deployment of a 27B LLM with llama.cpp

Deploying **Qwen3.8-27B-Uncensored-Cyber** (GGUF) on dual NVIDIA Tesla T4 GPUs and running cybersecurity inference **with internet access disabled** — no cloud AI API involved at any point.

---

## Overview

This project demonstrates that a 27-billion-parameter language model can be loaded from local storage and queried entirely offline. The GPUs were provided by a Kaggle runtime, but the inference itself ran locally inside that runtime: the model weights sat on the local filesystem, `llama.cpp` executed locally, and the final test was performed after internet access was switched off.

The distinction matters — this is **local inference inside a cloud compute environment**, not inference through a cloud AI service.

## Environment

| Component | Specification |
| --- | --- |
| GPUs | 2 × NVIDIA Tesla T4 |
| GPU memory | ~15 GB per GPU |
| CUDA | 13.0 |
| Driver | 580.159.04 |
| OS | Linux x86_64 |
| Compiler | GNU 11.4.0 |
| Inference engine | llama.cpp (CUDA build) |

## Model

| Detail | Value |
| --- | --- |
| Model | Qwen3.8-27B-Uncensored-Cyber |
| Quantization | IQ4_XS (imatrix, from q8) |
| Format | GGUF |
| Size | 14.96 GiB |

## Build

`llama.cpp` was cloned and compiled with CUDA support:

```bash
cmake -B build \
  -DGGML_CUDA=ON \
  -DGGML_CUDA_NO_VMM=ON \
  -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc \
  -DCUDAToolkit_ROOT=/usr/local/cuda
```

Resulting build:

```
0.4.1-dev
build 11061
commit b23efaa2e
```

## Dual-GPU configuration

The model layers were split across both T4s:

```bash
--device CUDA0,CUDA1 \
--split-mode layer \
--tensor-split 1,1 \
-ngl 99
```

## Results

| Metric | Value |
| --- | --- |
| Prompt processing | 94.5 tokens/second |
| Generation | 15.2 tokens/second |

Test prompt: *"What is the CIA triad in cybersecurity?"* — the model correctly described Confidentiality, Integrity, and Availability, both online and after internet access was disabled.

## Problems faced and solutions

| Problem | Solution |
| --- | --- |
| 404 on model download | The requested filename did not match the repository filename; the correct GGUF filename was identified and downloaded. |
| Large model size (~15 GB) | Storage usage was monitored throughout the run. |
| Very slow generation | The model uses a reasoning process; a 256-token generation limit was set for practical testing. |
| Terminal appears stuck at `>` | Not a hang — generation was still running. `nvidia-smi` confirmed active GPU utilisation; a token limit made responses more predictable. |
| Kaggle session not starting | The accelerator was busy or the GPU quota was near its limit. The session was stopped and restarted, the GPU re-selected in Settings, and the notebook re-run after a short wait. |

## Repository structure

```
.
├── README.md
└── report/
    ├── Assignment1_Local_AI_Deployment_Report.pdf
    └── Assignment1_Local_AI_Deployment_Report.docx
```

## Full report

The complete report — 15 sections with 12 annotated screenshots covering GPU detection, compilation, model download, online inference, disabling internet access, and the final offline test 


## Key takeaways

- Verifying GPU and CUDA availability before deployment
- Compiling `llama.cpp` with CUDA support
- Deploying GGUF models from local storage
- Distributing a large model across multiple GPUs
- Troubleshooting model-loading and inference issues
- Confirming that inference works with no network connection

Running a model offline keeps prompts and data off external services, which matters when working with confidential information, internal security documentation, sensitive logs, or private organizational data. Security then depends on protecting the machine, the model files, and the generated outputs.

## Authors

Azam Abbas · Awaiz Ahmed · Abdul Rafey
