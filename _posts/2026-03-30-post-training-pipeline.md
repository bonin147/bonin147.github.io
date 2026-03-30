---
layout: post
title: "Post-training Pipeline 개요"
title_en: "Post-training Pipeline Overview"
date: 2026-03-30
categories: [LLM]
subcategory: "Guide"
tags: [Post-Training, SFT, RLHF, DPO, GRPO, DAPO, RL, LoRA, QLoRA, DeepSpeed, PEFT]
description: "LLM Post-training의 전체 파이프라인을 정리합니다."
bilingual: true
---

<div class="lang-ko" markdown="1">

## Post-training이란

모델 학습은 크게 **Pre-training → Post-training** 순서로 진행된다. Pre-training이 대규모 코퍼스로 언어 능력을 학습하는 단계라면, Post-training은 모델을 실제 사용 목적에 맞게 조정하는 단계이다. 이 글에서는 Post-training의 전체 파이프라인을 다룬다.

최근 모델들의 Post-training은 크게 다음 흐름으로 진행된다:

> **SFT → Preference Optimization (RLHF/DPO) → RL**

SFT만 진행하기도 하고, SFT → RL로 바로 가기도 하며, SFT → RLHF만으로 마무리하기도 한다. 각 알고리즘이 요구하는 데이터의 형태와 목적이 다르기 때문에, 상황에 따라 학습 방법이 달라진다.

SFT 없이 RL 혹은 RLHF를 직접 수행하기도 하지만, **SFT를 통한 warm start를 추천한다.** RL 계열 알고리즘은 initial model에 대한 regularization term(예: KL divergence penalty)이 학습 알고리즘에 포함되어 있어 모델이 크게 변하지 않는다. 시작 모델이 해당 task를 잘 수행하지 못한다면, RL만으로는 충분한 개선을 기대하기 어렵다.

## 학습 알고리즘별 정리

### SFT (Supervised Fine-Tuning)

- **필요 데이터:** (input, target output)
- **목적:** Instruction-following 능력 부여, chat · coding · math 등 general task 학습
- **특징:** 가장 단순하고 빠른 학습 방법. 학습 시간과 메모리 사용량이 가장 적다.

### Preference Optimization

- **대표 알고리즘:** RLHF (PPO), DPO
- **필요 데이터:** (input, response A, response B, preference label)
- **목적:** 정답이 명확하지 않은 task — dialogue, writing, summarization 등
- **특징:** 사람(또는 AI)의 선호도 판단을 학습에 반영한다.

### RL (Reinforcement Learning)

- **대표 알고리즘:** GRPO, DAPO
- **필요 데이터:** (input, reward function)
  - 예: 수학 문제에서 정답을 맞추면 reward +1, 틀리면 -1
- **목적:** 정답 검증이 가능한 task — 수학, coding 등 output의 정답 여부를 명확히 판별할 수 있는 경우. Reasoning task에 주로 사용.

### 알고리즘 비교

| | SFT | Preference Optimization | RL |
|---|---|---|---|
| **데이터 형태** | (input, output) | (input, A, B, preference) | (input, reward function) |
| **대표 알고리즘** | - | RLHF(PPO), DPO | GRPO, DAPO |
| **적합한 task** | General task | 주관적 판단 task | 정답 검증 가능한 task |
| **학습 시간** | 빠름 | 보통 | 느림 |
| **메모리 사용량** | 적음 | 많음 | 많음 |

> RL 계열은 여러 모델을 메모리에 동시에 올려야 하기 때문에 SFT 대비 메모리 사용량이 크게 증가한다.

## PEFT (Parameter Efficient Fine-Tuning)

PEFT는 학습 알고리즘이 아니라 **학습 방식의 최적화 전략**이다. SFT, RLHF, RL 어떤 알고리즘에든 적용할 수 있다. 목적은 Full Fine-Tuning(FT)과 유사한 성능을 더 적은 메모리와 연산으로 달성하는 것이다.

### LoRA (Low-Rank Adaptation)

Pre-trained model을 freeze하고, 소규모 adapter parameter만 새로 도입하여 학습한다. Post-training에서는 FT와 동등한 성능을 달성할 수 있다는 분석이 있다. Freeze된 모델은 inference만 수행하고 gradient를 계산하지 않으므로, VRAM 사용량에서 큰 차이가 발생한다. 학습 속도도 빠르다.

### QLoRA (LoRA + Quantization)

LoRA에서 freeze된 Pre-trained model을 4-bit quantization하여 올리는 방식이다. LoRA 대비 추가적인 메모리 절감이 가능하다.

### DeepSpeed

분산 학습 시 메모리를 효율적으로 관리하고 학습/추론 속도를 높여주는 프레임워크이다. Full Fine-Tuning에서 주로 사용하며, 멀티 GPU 환경에서 병렬 학습을 최적화한다. ZeRO Stage 0, 1, 2가 있으며, **Stage 2 < 1 < 0** 순으로 속도가 빠르지만 더 많은 VRAM을 요구한다.

## 실전 팁

- **LoRA 사용 시** learning rate를 FT 대비 약 **10배** 높게 설정한다.
- **RL/RLHF 학습 시** learning rate를 SFT 대비 약 **10배** 낮게 설정한다.
- 학습 hyperparameter가 다양하지만, 실전에서는 주로 **batch size**와 **learning rate**를 조정한다. 나머지는 널리 쓰이는 default 값을 사용하는 경우가 많다.
- Batch size와 learning rate의 적절한 값은 model size, dataset size 등에 따라 달라지므로, 매번 탐색이 필요하다.

</div>

<div class="lang-en" markdown="1">

## What is Post-training?

Model training follows two main stages: **Pre-training → Post-training**. While pre-training teaches the model general language capabilities from large-scale corpora, post-training adapts the model for its intended use case. This article covers the full post-training pipeline.

The typical post-training flow in recent models is:

> **SFT → Preference Optimization (RLHF/DPO) → RL**

In practice, the pipeline varies: some models use SFT alone, others go SFT → RL directly, and some stop at SFT → RLHF. Each algorithm requires different data formats and serves a different purpose.

While it is possible to apply RL or RLHF without SFT, **starting from an SFT-warmed model is recommended.** RL-family algorithms include a regularization term (e.g., KL divergence penalty) against the initial model, which prevents the model from changing drastically. If the starting model performs poorly on the target task, RL alone cannot compensate.

## Training Algorithms

### SFT (Supervised Fine-Tuning)

- **Data format:** (input, target output)
- **Purpose:** Teaching instruction-following, general tasks like chat, coding, and math
- **Characteristics:** The simplest and fastest training method, with the lowest memory footprint.

### Preference Optimization

- **Key algorithms:** RLHF (PPO), DPO
- **Data format:** (input, response A, response B, preference label)
- **Purpose:** Tasks without clear-cut answers — dialogue, creative writing, summarization
- **Characteristics:** Incorporates human (or AI) preference judgments into training.

### RL (Reinforcement Learning)

- **Key algorithms:** GRPO, DAPO
- **Data format:** (input, reward function)
  - e.g., +1 reward for a correct math answer, -1 for incorrect
- **Purpose:** Tasks where correctness is verifiable — math, coding, and other problems with deterministic answers. Widely used for reasoning tasks.

### Algorithm Comparison

| | SFT | Preference Optimization | RL |
|---|---|---|---|
| **Data format** | (input, output) | (input, A, B, preference) | (input, reward function) |
| **Key algorithms** | - | RLHF (PPO), DPO | GRPO, DAPO |
| **Best for** | General tasks | Subjective tasks | Verifiable tasks |
| **Training time** | Fast | Moderate | Slow |
| **Memory usage** | Low | High | High |

> RL-based methods require multiple models loaded in memory simultaneously, leading to significantly higher memory usage compared to SFT.

## PEFT (Parameter Efficient Fine-Tuning)

PEFT is not a training algorithm but an **optimization strategy for training efficiency**. It can be applied to any algorithm — SFT, RLHF, or RL. The goal is to achieve performance comparable to Full Fine-Tuning (FT) with less memory and compute.

### LoRA (Low-Rank Adaptation)

Freezes the pre-trained model and introduces a small set of trainable adapter parameters. Studies suggest that LoRA can match FT performance in post-training settings. Since the frozen model only performs inference without computing gradients, the VRAM savings are substantial. Training is also faster.

### QLoRA (LoRA + Quantization)

Applies 4-bit quantization to the frozen pre-trained model on top of LoRA, achieving further memory reduction beyond standard LoRA.

### DeepSpeed

A distributed training framework that optimizes memory management and accelerates training/inference. Primarily used with Full Fine-Tuning for multi-GPU parallel training. It offers ZeRO Stages 0, 1, and 2 — **Stage 2 < 1 < 0** in speed, but higher stages use less VRAM.

## Practical Tips

- **When using LoRA**, set the learning rate roughly **10x higher** than with full fine-tuning.
- **For RL/RLHF**, set the learning rate roughly **10x lower** than for SFT.
- Although there are many hyperparameters, in practice **batch size** and **learning rate** are the primary knobs to tune. Other parameters are typically left at widely-used defaults.
- Optimal batch size and learning rate depend on model size, dataset size, and other factors, so they need to be searched for each setup.

</div>
