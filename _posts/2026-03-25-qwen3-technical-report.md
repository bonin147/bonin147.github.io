---
layout: post
title: "Qwen3 Technical Report: Thinking과 Non-Thinking의 통합"
title_en: "Qwen3 Technical Report: Unifying Thinking and Non-Thinking Modes"
date: 2026-03-25
categories: [LLM]
subcategory: "Paper Review"
tags: [Qwen, MoE, Reasoning, Distillation, Alibaba]
description: "Alibaba Qwen 팀이 발표한 Qwen3 시리즈를 소개합니다. Thinking/Non-Thinking 모드 통합, Strong-to-Weak Distillation 등 핵심 기여를 정리합니다."
bilingual: true
---

<div class="lang-ko" markdown="1">

Alibaba의 Qwen 팀이 2025년 5월에 발표한 **Qwen3** 시리즈의 기술 보고서를 소개합니다. Qwen3는 0.6B부터 235B까지 다양한 스케일의 모델을 포함하며, **Thinking 모드와 Non-Thinking 모드를 하나의 모델에 통합**한 것이 가장 큰 특징입니다.

## 핵심 기여

### 1. Thinking / Non-Thinking 모드 통합

기존에는 복잡한 추론이 필요한 작업에는 reasoning 모델(예: QwQ-32B, o1)을, 빠른 응답이 필요한 작업에는 chat 모델(예: GPT-4o)을 별도로 사용해야 했습니다. Qwen3는 이 두 가지를 **하나의 모델에 통합**합니다.

- **Thinking 모드**: `<think>...</think>` 태그 안에서 다단계 추론을 수행
- **Non-Thinking 모드**: 추론 과정 없이 빠르게 응답
- 사용자가 `/think`, `/no_think` 플래그로 모드 전환 가능
- **Thinking Budget**: 추론에 사용할 토큰 수를 지정하여 latency와 성능 간 균형 조절

### 2. Strong-to-Weak Distillation

대형 모델의 지식을 소형 모델로 효율적으로 전달하는 방법을 제시합니다.

- Teacher 모델의 output logits를 직접 student 모델에 distill
- RL 대비 약 **1/10의 GPU 시간**으로 유사하거나 더 나은 성능 달성
- Pass@1 성능뿐 아니라 Pass@64(탐색 능력)도 향상
- RL은 Pass@64를 개선하지 못하지만, distillation은 탐색 공간 자체를 확장

### 3. 119개 언어 지원

Qwen2.5의 29개 언어에서 **119개 언어**로 지원 범위를 대폭 확대했습니다. 다국어 데이터 annotation 시스템을 개발하여 30조 토큰 이상에 교육적 가치, 도메인, 분야 등의 메타데이터를 부여했습니다.

## 아키텍처

Qwen3는 Dense 모델 6종과 MoE 모델 2종으로 구성됩니다.

| 모델 | 구조 | 전체 파라미터 | 활성 파라미터 |
|------|------|-------------|-------------|
| Qwen3-0.6B | Dense | 0.6B | 0.6B |
| Qwen3-1.7B | Dense | 1.7B | 1.7B |
| Qwen3-4B | Dense | 4B | 4B |
| Qwen3-8B | Dense | 8B | 8B |
| Qwen3-14B | Dense | 14B | 14B |
| Qwen3-32B | Dense | 32B | 32B |
| Qwen3-30B-A3B | MoE | 30B | 3B |
| Qwen3-235B-A22B | MoE | 235B | 22B |

아키텍처는 Qwen2.5를 기반으로 하되, QKV-bias를 제거하고 **QK-Norm**을 도입했습니다. GQA, SwiGLU, RoPE, RMSNorm은 유지됩니다.

## 4단계 Post-Training 파이프라인

Flagship 모델은 정교한 4단계 학습 과정을 거칩니다:

1. **Long CoT Cold Start**: 수학, 코딩, 논리 등의 long chain-of-thought 데이터로 SFT
2. **Reasoning RL**: 강화학습을 통해 추론 능력 강화 (예: AIME'24 점수 70.1 → 85.1)
3. **Thinking Mode Fusion**: Non-thinking 능력을 thinking 모델에 통합하는 continual SFT
4. **General RL**: 20개 이상의 태스크에 대한 범용 강화학습 (instruction following, format following, preference alignment, agent ability 등)

## 주요 벤치마크 결과

### Flagship 모델 (Qwen3-235B-A22B, Thinking 모드)

| 벤치마크 | Qwen3-235B | DeepSeek-R1 | OpenAI-o1 | Gemini 2.5 Pro |
|---------|-----------|-------------|-----------|----------------|
| AIME'24 | **85.7** | 79.8 | 74.3 | 92.0 |
| AIME'25 | 81.5 | 70.0 | 79.2 | **86.7** |
| LiveCodeBench v5 | **70.7** | 64.3 | 63.9 | 70.4 |
| CodeForces | **2056** | 2029 | 1891 | 2001 |
| BFCL v3 | **70.8** | 56.9 | 67.8 | 62.9 |

### Dense 모델 하이라이트 (Qwen3-32B)

Qwen3-32B는 특히 주목할 만합니다:
- 이전 최강 reasoning 모델 QwQ-32B를 대부분의 벤치마크에서 상회
- 비공개 모델 OpenAI-o3-mini와 비견되는 성능
- Non-thinking 모드에서도 Qwen2.5-72B-Instruct(2배 이상 큰 모델)를 능가

### 소형 모델의 효율성

Strong-to-Weak Distillation 덕분에 소형 모델도 뛰어난 성능을 보입니다:
- **Qwen3-4B**: DeepSeek-R1-Distill-Qwen-14B(3.5배 큰 모델)와 비견
- **Qwen3-8B**: AIME'25에서 67.3으로 DeepSeek-R1-Distill-Qwen-32B(49.6)를 대폭 상회
- **Qwen3-30B-A3B**: 활성 파라미터 3B로 QwQ-32B 수준의 성능

## Thinking Budget의 효과

Thinking 모드에서 사용할 토큰 수를 제한할 수 있으며, 토큰 수가 증가할수록 성능이 일관되게 향상됩니다. 이를 통해 사용자는 응답 속도와 품질 사이의 트레이드오프를 태스크 복잡도에 따라 유연하게 조절할 수 있습니다.

## 의의

- **실용성**: 하나의 모델로 reasoning과 빠른 응답 모두 가능하여 배포 복잡성 감소
- **효율성**: Distillation을 통해 소형 모델에서도 강력한 추론 능력 확보
- **오픈소스**: 전체 모델이 오픈 웨이트로 공개되어 연구/산업계 활용 가능
- **다국어**: 119개 언어 지원으로 글로벌 활용 범위 확대

> **논문 정보**
> - 제목: Qwen3 Technical Report
> - 저자: Qwen Team (Alibaba)
> - HuggingFace: [Qwen](https://huggingface.co/Qwen)
> - GitHub: [QwenLM/Qwen3](https://github.com/QwenLM/Qwen3)

</div>

<div class="lang-en" markdown="1">

This post introduces the **Qwen3** series technical report, published by Alibaba's Qwen team in May 2025. Qwen3 includes models ranging from 0.6B to 235B parameters, with the key innovation being the **integration of Thinking and Non-Thinking modes into a single model**.

## Key Contributions

### 1. Unified Thinking / Non-Thinking Modes

Previously, complex reasoning tasks required dedicated reasoning models (e.g., QwQ-32B, o1), while quick responses needed chat models (e.g., GPT-4o). Qwen3 **unifies both into a single model**.

- **Thinking mode**: Multi-step reasoning within `<think>...</think>` tags
- **Non-Thinking mode**: Fast, direct responses without explicit reasoning
- Users switch modes with `/think` and `/no_think` flags
- **Thinking Budget**: Control the number of tokens allocated to reasoning, balancing latency and performance

### 2. Strong-to-Weak Distillation

An efficient method for transferring knowledge from large to small models:

- Direct distillation of output logits from teacher to student models
- Achieves comparable or better performance with ~**1/10 GPU hours** compared to RL
- Improves not only Pass@1 but also Pass@64 (exploration capability)
- RL fails to improve Pass@64, while distillation expands the search space itself

### 3. 119 Language Support

Language coverage expanded dramatically from 29 (Qwen2.5) to **119 languages**. A multilingual data annotation system was developed, tagging over 30 trillion tokens with metadata for educational value, domain, and field.

## Architecture

Qwen3 consists of 6 Dense models and 2 MoE models:

| Model | Type | Total Params | Active Params |
|-------|------|-------------|--------------|
| Qwen3-0.6B | Dense | 0.6B | 0.6B |
| Qwen3-1.7B | Dense | 1.7B | 1.7B |
| Qwen3-4B | Dense | 4B | 4B |
| Qwen3-8B | Dense | 8B | 8B |
| Qwen3-14B | Dense | 14B | 14B |
| Qwen3-32B | Dense | 32B | 32B |
| Qwen3-30B-A3B | MoE | 30B | 3B |
| Qwen3-235B-A22B | MoE | 235B | 22B |

The architecture builds on Qwen2.5, removing QKV-bias and introducing **QK-Norm**. GQA, SwiGLU, RoPE, and RMSNorm are retained.

## 4-Stage Post-Training Pipeline

Flagship models undergo a sophisticated 4-stage training process:

1. **Long CoT Cold Start**: SFT with long chain-of-thought data for math, coding, and logic
2. **Reasoning RL**: Reinforcement learning to strengthen reasoning (e.g., AIME'24: 70.1 → 85.1)
3. **Thinking Mode Fusion**: Continual SFT to integrate non-thinking capabilities into the thinking model
4. **General RL**: Broad RL across 20+ tasks (instruction following, format following, preference alignment, agent ability, etc.)

## Key Benchmark Results

### Flagship Model (Qwen3-235B-A22B, Thinking Mode)

| Benchmark | Qwen3-235B | DeepSeek-R1 | OpenAI-o1 | Gemini 2.5 Pro |
|-----------|-----------|-------------|-----------|----------------|
| AIME'24 | **85.7** | 79.8 | 74.3 | 92.0 |
| AIME'25 | 81.5 | 70.0 | 79.2 | **86.7** |
| LiveCodeBench v5 | **70.7** | 64.3 | 63.9 | 70.4 |
| CodeForces | **2056** | 2029 | 1891 | 2001 |
| BFCL v3 | **70.8** | 56.9 | 67.8 | 62.9 |

### Dense Model Highlight (Qwen3-32B)

Qwen3-32B deserves special attention:
- Outperforms previous best reasoning model QwQ-32B on most benchmarks
- Competitive with closed-source OpenAI-o3-mini
- In non-thinking mode, surpasses Qwen2.5-72B-Instruct (a model more than 2x larger)

### Small Model Efficiency

Thanks to Strong-to-Weak Distillation, small models achieve impressive performance:
- **Qwen3-4B**: Comparable to DeepSeek-R1-Distill-Qwen-14B (3.5x larger)
- **Qwen3-8B**: Scores 67.3 on AIME'25, far exceeding DeepSeek-R1-Distill-Qwen-32B (49.6)
- **Qwen3-30B-A3B**: QwQ-32B-level performance with only 3B active parameters

## Thinking Budget Effect

The number of tokens allocated to thinking can be controlled, with performance consistently improving as the budget increases. This allows users to flexibly adjust the tradeoff between response speed and quality based on task complexity.

## Significance

- **Practicality**: One model handles both reasoning and quick responses, reducing deployment complexity
- **Efficiency**: Distillation enables strong reasoning in small models
- **Open-source**: All models released as open weights for research and industry use
- **Multilingual**: Support for 119 languages enables global adoption

> **Paper Info**
> - Title: Qwen3 Technical Report
> - Authors: Qwen Team (Alibaba)
> - HuggingFace: [Qwen](https://huggingface.co/Qwen)
> - GitHub: [QwenLM/Qwen3](https://github.com/QwenLM/Qwen3)

</div>
