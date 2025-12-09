# Short-Story-Assingment

# TZ-LLM: Protecting On-Device Large Language Models with Arm TrustZone

This repository provides a structured overview of TZ-LLM, a secure and efficient system architecture designed to protect on-device Large Language Models (LLMs) using Arm TrustZone. TZ-LLM resolves the core tension between memory efficiency, fast inference, and model confidentiality through innovations in secure memory scaling and accelerator sharing.

---

## Problem Overview

Running LLMs on mobile and edge devices improves privacy, reduces latency, and enables offline operation.  
However, it exposes model providers to significant leakage risks because plaintext parameters appear in memory during inference.

Existing approaches fail to meet one or more of these requirements:
- Efficient dynamic secure memory scaling  
- Secure and low-overhead accelerator usage  
- Minimal TEE Trusted Computing Base (TCB)

TZ-LLM addresses these challenges holistically.

---

## System Architecture

TZ-LLM introduces two key innovations:

### 1. Pipelined Parameter Restoration
A mechanism that overlaps:
- Secure memory scaling  
- Parameter loading from flash  
- AES decryption  
- LLM operator execution  

Since LLM computation graphs have deterministic parameter access patterns, TZ-LLM prefetches parameters in topological order and hides restoration overhead under computation.

Key techniques:
- Priority-based scheduling  
- Preemptive restoration to eliminate compute stalls  
- Partial parameter caching for improved warm-start performance  

### 2. Secure TEE–REE NPU Co-Driver Design
TZ-LLM separates the NPU driver into:
- TEE Data Plane (≈1K LoC): secure job execution, DMA protection, interrupt handling  
- REE Control Plane: scheduling, device power and frequency control  

This design enables secure NPU execution without reinitializing full drivers, minimizing switching overhead and TCB expansion.

---

## Security Model

TZ-LLM protects against:
- Direct memory extraction  
- DMA attacks from compromised REE or malicious devices  
- Iago attacks on memory allocation, NPU scheduling, and model loading  

Security mechanisms include:
- TZASC memory isolation  
- TZPC-controlled device access  
- Checksum validation for model loading  
- Monotonic job numbering to prevent reordering or replay attacks  

Only the TEE OS, TEE-side NPU driver, and LLM inference framework are trusted.

---

## Key Features

| Capability | Description |
|-----------|-------------|
| Elastic Secure Memory Scaling | Dynamically grows and shrinks secure memory regions while maintaining contiguity. |
| Pipelined Restoration | Overlaps decryption, loading, and allocation with execution. |
| Secure NPU Time-Sharing | Enables LLM inference on NPUs inside the TEE without heavy switching overhead. |
| Minimal TCB | Only ~100 LoC added to TEE OS and ~1K LoC added for secure NPU support. |
| Partial Parameter Caching | Reduces cold-start latency and improves TTFT under low memory pressure. |

---

## Performance Summary

Evaluations performed on TinyLlama-1.1B, Qwen2.5-3B, Phi-3-3.8B, and Llama-3-8B show:

### Time-To-First-Token (TTFT)
- 76.1%–90.9% reduction compared to a naive TEE baseline  
- 5.2%–28.3% overhead compared to REE pipelined loading  

### Decoding Speed
- 0.9%–23.2% faster than TEE CPU-only execution  
- Only 1.3%–4.9% slower than REE NPU execution  

### NPU Sharing Overhead
- Up to 3.0% degradation for LLM inference  
- Up to 3.8% degradation for concurrent NPU applications  

---

## Implementation Overview

Platform: OpenHarmony OS, Rockchip RK3588 (CPU + NPU)

Components:
- Modified llama.cpp inference framework  
- TEE user-mode NPU data-plane driver  
- REE TZ and NPU driver extensions  
- Dynamic TZASC and TZPC runtime configuration  

Code size changes:
- TEE OS modifications: ~112 LoC  
- LLM TA extensions: ~2.2K LoC  
- REE kernel changes: 364 LoC  

---

## Compatibility and Limitations

### Compatible With
- Quantized LLMs (e.g., 8-bit)  
- Mobile and embedded NPUs  
- Low-memory devices  
- Deterministic transformer models  

### Limitations
- Reduced efficiency for mixture-of-experts (MoE) or early-exit models  
- Does not protect against physical memory attacks  
- Some structural metadata may leak via timing or memory channels  

---

## Future Directions

- Hardware-backed memory encryption with Arm CCA  
- Extending the secure execution boundary to GPUs and heterogeneous accelerators  
- Improving support for MoE and adaptive transformers  
- Integrating with parameter offloading systems for extremely large models  

---

## Deliverables

### Slide Deck  

### YouTube Presentation  

### Medium Article https://medium.com/@mannnada05/on-device-llms-are-finally-becoming-practical-heres-what-engineers-need-to-know-in-2025-7cc24d544e25

---

## Research Paper

This repository summarizes concepts from:  
**"TZ-LLM: Protecting On-Device Large Language Models with Arm TrustZone"** (EUROSYS ’26).

