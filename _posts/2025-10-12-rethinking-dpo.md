---
layout: post
title: "Rethinking DPO: Rejected Response가 Preference Misalignment에 미치는 영향"
title_en: "Rethinking DPO: The Role of Rejected Responses in Preference Misalignment"
date: 2025-10-12
categories: [NLP]
subcategory: "Paper Review"
tags: [My-Paper, DPO, RLHF, Preference-Optimization, Alignment, EMNLP]
description: "DPO에서 rejected response의 역할을 재조명하고, preference misalignment 문제를 분석합니다."
bilingual: true
paper_url: "https://aclanthology.org/2025.findings-emnlp.433/"
arxiv_url: "https://arxiv.org/abs/2506.12725"
paper_venue: "Findings of EMNLP 2025"
paper_authors: "Jae Hyeon Cho, JunHyeok Oh, Myunsoo Kim, Byung-Jun Lee"
---

<div class="lang-ko" markdown="1">

## 개요

DPO(Direct Preference Optimization)는 reward model 없이 직접 preference를 최적화할 수 있어 널리 사용되고 있지만, 학습된 모델이 인간의 선호와 misalign되는 현상이 빈번하게 보고되고 있다. 구체적으로 두 가지 문제가 있다:

1. Sampling 없이 학습하기 때문에 **OOD(Out-of-Distribution) action의 확률이 증가**하는 경향
2. **Chosen response의 확률을 충분히 높이지 못하는** 현상

본 연구에서는 이 문제의 근본 원인을 **rejected response가 loss function을 지배하는 구조적 불균형**에서 찾고, 이를 해결하는 BDPO(Bounded-DPO)를 제안한다.

## DPO의 구조적 문제

DPO의 loss function은 다음과 같다:

$$\max_\theta \mathbb{E}_\mathcal{D}\left[\log \sigma\left(\hat{s}_\theta(w;l) - \hat{s}_{\text{ref}}(w;l)\right)\right]$$

여기서 $\hat{s}_\theta(w;l) := \beta \log \frac{\pi_\theta(y_w|x)}{\pi_\theta(y_l|x)}$ 이다.

문제는 이 loss의 최적해가 $\pi_\theta(y_l|x) \to 0$이면 $\pi_\theta(y_w|x)$의 값에 관계없이 달성될 수 있다는 점이다. 즉, **rejected response의 확률만 줄이면 loss가 줄어들기 때문에**, 모델이 chosen response의 확률을 높이는 방향으로 학습하지 않게 된다.

Gradient 관점에서 보면, $\pi_\theta(y_l|x)$에 대한 gradient에 $\frac{1}{\pi_\theta(y_l|x)}$ 항이 포함되어 있어, rejected response의 확률이 작아질수록 gradient가 폭발적으로 커진다. 반면 chosen response에 대한 gradient는 상대적으로 안정적이어서, **최적화가 rejected response 쪽으로 편향**된다.

## 기존 해결책의 한계

### DPO+NLL

DPO에 NLL(Negative Log-Likelihood) loss를 추가하는 방법이다:

$$\mathcal{L}_{\text{DPO+NLL}} = \mathcal{L}_{\text{DPO}} + \alpha \cdot \mathcal{L}_{\text{NLL}}$$

Chosen response의 확률을 직접 높이려는 시도이지만, **$\alpha$ 값에 극도로 민감**하다. $\alpha$가 작으면 DPO의 문제가 그대로 남고, 크면 reference model에서 크게 벗어난다. 실험에서 $\alpha$를 0.01부터 10까지 탐색했지만, 어떤 값에서도 BDPO를 넘지 못했다.

### DPOP

Chosen response의 확률이 reference model보다 낮아지면 penalty를 부여하는 방법이다. 하지만 penalty가 활성화되는 조건이 제한적이어서, **DPO의 근본적인 불균형을 해결하지 못한다.**

## 제안 방법: BDPO (Bounded-DPO)

핵심 아이디어는 단순하다. DPO loss의 분모에서 $\pi_\theta(y_l|x)$를 **mixture distribution**으로 대체한다:

$$\pi_{\text{mix}}(y|x) = \lambda \pi_\theta(y|x) + (1-\lambda)\pi_{\text{ref}}(y|x)$$

이렇게 하면 분모가 $(1-\lambda)\pi_{\text{ref}}(y_l|x)$로 lower bound되어, rejected response의 확률이 0에 가까워져도 gradient가 폭발하지 않는다.

### 이론적 보장

**Theorem 1.** BDPO loss를 최소화하는 최적 정책 $\pi^*$는 $\pi^*(y_w|x) = 1$, $\pi^*(y_l|x) = 0$을 만족한다. 즉, **이상적인 preference alignment에 수렴**한다.

**Theorem 2.** 학습 과정에서 chosen response의 확률에 대한 lower bound가 보장된다:

$$(1-\lambda)\pi_{\text{ref}}(y_w|x) \le \pi_\theta(y_w|x)$$

이는 DPO에서는 보장되지 않는 성질이다.

## Loss Contour 분석

$\pi_{\text{ref}}(y_w|x) = 0.4$, $\pi_{\text{ref}}(y_l|x) = 0.1$일 때의 loss contour를 비교하면:

- **DPO**: $\pi_\theta(y_l|x)$를 0에 가깝게 줄이는 것만으로 loss가 최소화됨. $\pi_\theta(y_w|x)$의 값은 무관
- **DPOP**: DPO와 유사한 패턴. Penalty는 $\pi_\theta(y_w|x)$가 매우 낮을 때만 활성화
- **DPO+NLL**: 개선되지만, $\alpha$ 값에 따라 contour가 극적으로 변화
- **BDPO**: $\pi_\theta(y_w|x)$를 높이고 $\pi_\theta(y_l|x)$를 낮추는 방향 모두에 대해 균형 잡힌 최적화

## 학습 동역학

학습 과정에서의 주요 지표 변화:

- **DPO**: Chosen과 rejected 확률이 **모두 감소** — chosen이 증가해야 하는데 그렇지 않음
- **DPOP**: Rejected는 감소하지만 chosen은 reference 근처에 머무름
- **DPO+NLL**: Chosen은 증가하지만 KL divergence가 ~347까지 폭발 — reference model에서 심하게 이탈
- **BDPO**: Chosen 증가, rejected 감소, KL divergence ~177로 안정적 — **모든 목표를 균형 있게 달성**

## 실험 결과

### IFEval (Instruction-Following, Qwen2.5-0.5B)

| Method | Total Score |
|--------|------------|
| DPO | 25.25 |
| DPOP | 26.46 |
| DPO+NLL | 25.95 |
| SPPO | 26.57 |
| **BDPO** | **27.15** |

### GSM8K (수학 추론, 4-shot CoT)

| Method | Accuracy |
|--------|----------|
| DPO | 27.45% |
| DPOP | 29.04% |
| DPO+NLL | 26.91% |
| **BDPO** | **29.95%** |

### MT-Bench

| Method | Average |
|--------|---------|
| DPO | 3.45 |
| DPOP | 3.23 |
| DPO+NLL | 3.11 |
| **BDPO** | **4.04** |

### 대형 모델에서의 일반화

Qwen2.5-7B에서도 BDPO가 74.28로 가장 높은 IFEval 점수를 기록했으며, LLaMA 3.2-1B에서도 57.79로 최고 성능을 달성했다. 추가 연산 비용은 전혀 없다 — 모든 방법이 동일한 학습 시간(~6h 10m)을 소요했다.

### $\lambda$ Ablation

$\lambda = 0.5$가 IFEval(27.15)과 GSM8K(29.95%) 모두에서 최적이었으며, 모든 $\lambda$ 값에서 DPO를 상회했다. $\lambda \to 1$이면 DPO로 수렴함을 실험적으로도 확인했다.

</div>

<div class="lang-en" markdown="1">

## Overview

DPO (Direct Preference Optimization) is widely used for aligning language models without reward model training, yet models trained with DPO frequently exhibit preference misalignment. Specifically, two issues arise:

1. The probability of **OOD (Out-of-Distribution) actions increases** due to the absence of sampling during training
2. DPO **fails to sufficiently increase the probability of chosen responses**

In this work, we trace the root cause to a **structural imbalance where rejected responses dominate the loss function**, and propose BDPO (Bounded-DPO) to address it.

## The Structural Problem of DPO

The DPO loss function is:

$$\max_\theta \mathbb{E}_\mathcal{D}\left[\log \sigma\left(\hat{s}_\theta(w;l) - \hat{s}_{\text{ref}}(w;l)\right)\right]$$

where $\hat{s}_\theta(w;l) := \beta \log \frac{\pi_\theta(y_w|x)}{\pi_\theta(y_l|x)}$.

The problem is that the optimal solution can be achieved as long as $\pi_\theta(y_l|x) \to 0$, regardless of $\pi_\theta(y_w|x)$. Since **reducing the rejected response probability alone decreases the loss**, the model never learns to increase the chosen response probability.

From a gradient perspective, the gradient w.r.t. $\pi_\theta(y_l|x)$ contains a $\frac{1}{\pi_\theta(y_l|x)}$ term that explodes as the rejected probability approaches zero, while the chosen response gradient remains stable. This creates an **optimization bias toward the rejected response**.

## Limitations of Existing Solutions

### DPO+NLL

Adds an NLL (Negative Log-Likelihood) loss to DPO:

$$\mathcal{L}_{\text{DPO+NLL}} = \mathcal{L}_{\text{DPO}} + \alpha \cdot \mathcal{L}_{\text{NLL}}$$

This attempts to directly increase chosen response probability, but is **extremely sensitive to $\alpha$**. Small $\alpha$ leaves DPO's problem intact; large $\alpha$ causes significant deviation from the reference model. We searched $\alpha$ from 0.01 to 10, but no value surpassed BDPO.

### DPOP

Penalizes when chosen response probability drops below the reference model. However, the penalty only activates under limited conditions, **failing to address DPO's fundamental imbalance**.

## Proposed Method: BDPO (Bounded-DPO)

The core idea is simple. We replace $\pi_\theta(y_l|x)$ in the DPO loss denominator with a **mixture distribution**:

$$\pi_{\text{mix}}(y|x) = \lambda \pi_\theta(y|x) + (1-\lambda)\pi_{\text{ref}}(y|x)$$

This lower-bounds the denominator by $(1-\lambda)\pi_{\text{ref}}(y_l|x)$, preventing gradient explosion even as the rejected probability approaches zero.

### Theoretical Guarantees

**Theorem 1.** The optimal policy $\pi^*$ minimizing the BDPO loss satisfies $\pi^*(y_w|x) = 1$ and $\pi^*(y_l|x) = 0$ — it **converges to ideal preference alignment**.

**Theorem 2.** During training, a lower bound on the chosen response probability is guaranteed:

$$(1-\lambda)\pi_{\text{ref}}(y_w|x) \le \pi_\theta(y_w|x)$$

This property is not guaranteed under standard DPO.

## Loss Contour Analysis

Comparing loss contours with $\pi_{\text{ref}}(y_w|x) = 0.4$ and $\pi_{\text{ref}}(y_l|x) = 0.1$:

- **DPO**: Loss is minimized by reducing $\pi_\theta(y_l|x)$ to near zero alone; $\pi_\theta(y_w|x)$ is irrelevant
- **DPOP**: Similar pattern to DPO; penalty only activates when $\pi_\theta(y_w|x)$ is very low
- **DPO+NLL**: Improved, but contours change dramatically with $\alpha$
- **BDPO**: Balanced optimization toward both increasing $\pi_\theta(y_w|x)$ and decreasing $\pi_\theta(y_l|x)$

## Training Dynamics

Key metric trajectories during training:

- **DPO**: Both chosen and rejected probabilities **decrease** — chosen should increase but doesn't
- **DPOP**: Rejected decreases but chosen stays near reference
- **DPO+NLL**: Chosen increases but KL divergence explodes to ~347 — severe deviation from reference
- **BDPO**: Chosen increases, rejected decreases, KL divergence stable at ~177 — **all objectives balanced**

## Experimental Results

### IFEval (Instruction-Following, Qwen2.5-0.5B)

| Method | Total Score |
|--------|------------|
| DPO | 25.25 |
| DPOP | 26.46 |
| DPO+NLL | 25.95 |
| SPPO | 26.57 |
| **BDPO** | **27.15** |

### GSM8K (Math Reasoning, 4-shot CoT)

| Method | Accuracy |
|--------|----------|
| DPO | 27.45% |
| DPOP | 29.04% |
| DPO+NLL | 26.91% |
| **BDPO** | **29.95%** |

### MT-Bench

| Method | Average |
|--------|---------|
| DPO | 3.45 |
| DPOP | 3.23 |
| DPO+NLL | 3.11 |
| **BDPO** | **4.04** |

### Generalization to Larger Models

BDPO also achieves the highest IFEval score on Qwen2.5-7B (74.28) and LLaMA 3.2-1B (57.79). There is zero additional computational cost — all methods require identical training time (~6h 10m).

### $\lambda$ Ablation

$\lambda = 0.5$ is optimal for both IFEval (27.15) and GSM8K (29.95%), and BDPO outperforms DPO across all tested $\lambda$ values. As $\lambda \to 1$, BDPO converges to DPO, as confirmed experimentally.

</div>
