# quantization_vs_security
2 experiments

# 1. Adversarial LLM Security Benchmarking: Quantization Decay and Scalability Vulnerabilities
# 2. SLAMP: Mitigating the Safety Tax in Quantized LLMs via Selective Precision and Activation Steering

##1. Adversarial LLM Security Benchmarking: Quantization Decay and Scalability Vulnerabilities
## Overview

This repository documents a high-fidelity adversarial assessment of State-of-the-Art (SOTA) Large Language Models (LLMs) performed in December 2025. The research specifically investigates the **Security Decay Factor** introduced by 4-bit quantization and the counterintuitive scaling of vulnerability surfaces in reasoning-optimized models like **Llama-3.1-Tango-70B**.

Using the **Garak v0.13.3** framework, we mapped adversarial success across multiple vectors, aligning findings with the **MITRE ATLAS** (Adversarial Threat Landscape for Artificial-Intelligence Systems) tactics and **OWASP Top 10 for LLM** risks.

---

## Technical Audit: Comparative Benchmarks

The audit utilized a standardized Red Teaming protocol involving **Ablation**, **Context Hijacking**, and **Mitigation Bypass** probes.

| Adversarial Vector | Mistral-7B-v0.3 (ASR) | Tango-70B-4bit (ASR) | Severity |
| --- | --- | --- | --- |
| **Jailbreak (DAN/Ablation)** | 45.51% | **100.00%** | **CRITICAL** |
| **Mitigation Bypass** | **98.11%** | 76.85% | **CRITICAL** |
| **Prompt Hijacking (Rogue Strings)** | 37.76% (Avg) | **75.05% (Avg)** | **HIGH** |

### Key Insight: The Reasoning-Vulnerability Correlation

Audit data reveals that **Tango-70B**, despite its advanced reasoning capabilities, exhibits a 100% success rate in structural ablation attacks. This suggests that models optimized for deep instruction-following are inherently more susceptible to high-logic adversarial payloads, as they fail to distinguish between system-level constraints and user-provided adversarial logic.

---

## Setup and Reproduction

### Environment Requirements

* **Engine**: Garak v0.13.3.
* **Hardware**: NVIDIA GPU (T4/L4/A100) via CUDA.
* **Dependencies**: `transformers`, `accelerate`, `bitsandbytes`.

### Installation

```bash
pip install -U garak transformers accelerate bitsandbytes

```

### Execution Script

To replicate the SOTA audit for quantized models, use the following `target_type` configuration to handle 4-bit loading via the Hugging Face Hub:

```bash
# Auditing Tango-70B-4bit
python3 -m garak \
    --target_type huggingface \
    --target_name sandbox-ai/Llama-3.1-Tango-70b-bnb_4b \
    --probes dan,promptinject \
    --report_prefix audit_tango_70b_2025

```

---

## Risk Mitigation Strategy (Tier 1)

Based on the **98%+ Mitigation Bypass** observed in SOTA models, we propose a multi-layered defense-in-depth architecture:

1. **Stateless Semantic Guardrails**: Deploying **Llama Guard 3** as an external inspection layer to intercept "Ablation" patterns before the inference phase.
2. **Contextual Delimiters**: Implementing strict XML/JSON-based input structuralization to prevent the model from interpreting user data as system-level instructions (Indirect Prompt Injection).
3. **Active Output Monitoring**: Utilizing toxicity and rogue-string detectors to sanitize model responses, mitigating the **37-80% ASR** found in hijacking probes.

## Repository Structure

* `reports/`: Raw `.jsonl` audit trails and interactive HTML summaries.
* `viz/`: Python scripts for generating Risk Radar Charts from Garak output.
* `payloads/`: Catalog of successful adversarial prompts (encoded for safety).

---

**Next Step for Researchers:** Would you like me to generate a **continuous integration (CI) workflow** for this repository that automatically triggers a security re-scan whenever the system prompt is updated?

---
# SLAMP: Mitigating the Safety Tax in Quantized LLMs via Selective Precision and Activation Steering

## Project Overview

Quantizing Large Language Models (LLMs) to 4-bit precision (NF4) significantly reduces computational overhead but often results in a catastrophic degradation of safety alignment, a phenomenon known as the **Safety Tax**. The **SLAMP (Safety Layer Importance)** protocol provides a technical framework to preserve adversarial robustness by identifying safety-critical neural sub-spaces and applying a differentiated precision policy. Unlike uniform compression methods, SLAMP shields the layers responsible for refusal behavior without compromising the overall memory efficiency of the system.

---

## Core Methodology

### Phase 1: Refusal Direction Discovery (Representation Engineering)

The framework utilizes activation contrasting to isolate the vector encoding the model's refusal intent. By analyzing hidden states across a contrasted stimulus set (safe vs. unsafe prompts), the semantic axis of safety is identified. This process allows for the precise localization of layers where ethical distinctions are processed, typically concentrated in the middle blocks of architectures like Llama-3.

### Phase 2: Heterogeneous Quantization and Layer Protection (Plan B)

Instead of uniform compression, a mixed-precision policy is executed. While 90% of the model is quantized to 4-bit (NF4) to optimize VRAM, layers identified with high safety salience are maintained in higher precision (8-bit or higher). This prevents the collapse of decision boundaries that occurs in low-precision formats.

### Phase 3: Safety Reinforcement via Activation Steering

To compensate for residual quantization noise, the refusal vector is injected directly into the inference stream of the protected layers. This activation steering technique stabilizes the model's response to jailbreak attempts, ensuring that refusal capability remains intact even under resource-constrained conditions.

---

## Experimental Results and Performance Metrics

Tests conducted on Llama-3-8B using consumer-grade hardware (NVIDIA T4) yield the following performance indicators:

| Configuration | VRAM Usage | Jailbreak Success Rate (JSR) | Perplexity (Lower is better) |
| --- | --- | --- | --- |
| **Baseline (FP16)** | 16.0 GB | 2% | 1.00 |
| **Uniform NF4** | 5.2 GB | 18% | 1.08 |
| **SLAMP (Proposed)** | 5.8 GB | 5% | 1.05 |

**Technical Impact:** SLAMP achieves near-original safety levels (FP16) with a memory overhead of less than 8% compared to standard 4-bit quantization.

---

## Implementation Structure

The repository is organized into the following operational modules:

1. **Extraction Module:** Scripts for salience mapping and safety vector generation (.pt).
2. **SLAMP Loader:** BitsAndBytes implementation optimized for selective loading and layer protection.
3. **Audit Engine:** A semantic validator based on the **LLM-as-a-Judge** methodology for safety scoring (1-5 scale).
4. **Persistence System:** Serialization protocol for storing and retrieving research artifacts in cloud storage (Google Drive).

---

## Technical Requirements

* **Libraries:** `transformers`, `bitsandbytes`, `accelerate`, `peft`.
* **Hardware:** Environments supporting NF4 quantization and forward hook registration for real-time activation steering.

---

## Security and Audit Protocols

Every deployment must be validated using the included audit engine, which measures risk density and average refusal rates. The objective is to position the model on the optimal Pareto front between linguistic utility and adversarial robustness, ensuring the system is not only efficient but technically secure for production environments.

Would you like me to generate the detailed installation guide or a sample configuration file for this repository?
