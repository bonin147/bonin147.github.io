---
layout: post
title: "Rethinking DPO: Rejected Response가 Preference Misalignment에 미치는 영향"
title_en: "Rethinking DPO: The Role of Rejected Responses in Preference Misalignment"
date: 2025-10-12
categories: [NLP]
subcategory: "Paper Review"
tags: [DPO, RLHF, Preference-Optimization, Alignment, EMNLP]
description: "DPO에서 rejected response의 역할을 재조명하고, preference misalignment 문제를 분석합니다."
bilingual: true
paper_url: "https://aclanthology.org/2025.findings-emnlp.433/"
arxiv_url: "https://arxiv.org/abs/2506.12725"
paper_venue: "Findings of EMNLP 2025"
paper_authors: "Jae Hyeon Cho, JunHyeok Oh, Myunsoo Kim, Byung-Jun Lee"
---

<div class="lang-ko" markdown="1">

> **추천 점수: 5 / 5 (Strong Recommend)**
>
> *DPO의 근본적인 한계를 rejected response 관점에서 분석 — alignment 연구의 필수 레퍼런스.*

## 개요

DPO(Direct Preference Optimization)는 RLHF의 복잡한 reward model 학습 없이 직접 preference를 최적화할 수 있어 널리 사용되고 있다. 하지만 실제로 학습된 모델이 인간의 선호와 misalign되는 현상이 빈번하게 보고되고 있다.

본 논문은 이 문제의 원인을 **rejected response의 역할**에서 찾는다. DPO의 학습 과정에서 rejected response가 어떻게 gradient에 기여하는지를 이론적으로 분석하고, 이것이 preference misalignment를 유발하는 메커니즘을 규명한다.

## 핵심 기여

- **Rejected Response 분석**: DPO 학습에서 rejected response가 모델 업데이트에 미치는 정확한 영향을 이론적으로 도출
- **Misalignment 메커니즘 규명**: 왜 DPO가 특정 조건에서 인간 선호와 반대 방향으로 학습하는지를 설명
- **실증적 검증**: 다양한 alignment 벤치마크에서 이론적 분석이 실제 현상과 일치함을 확인

## 실험 결과

이론적 분석에 기반한 실험에서, rejected response의 품질과 다양성이 DPO 성능에 결정적 영향을 미침을 확인했다. 이를 바탕으로 한 간단한 개선 방법이 기존 DPO 대비 일관된 성능 향상을 달성했다.

</div>

<div class="lang-en" markdown="1">

> **Rating: 5 / 5 (Strong Recommend)**
>
> *A fundamental analysis of DPO's limitations through the lens of rejected responses — essential reading for alignment research.*

## Overview

DPO (Direct Preference Optimization) is widely used for aligning language models without the complexity of reward model training. However, models trained with DPO frequently exhibit preference misalignment with human preferences.

This paper traces the root cause to **the role of rejected responses**. We theoretically analyze how rejected responses contribute to gradients during DPO training and identify the mechanism by which this causes preference misalignment.

## Key Contributions

- **Rejected Response Analysis**: Theoretical derivation of the exact impact of rejected responses on model updates during DPO training
- **Misalignment Mechanism**: Explains why DPO can learn in the opposite direction of human preferences under specific conditions
- **Empirical Validation**: Confirms that theoretical analysis matches observed phenomena across multiple alignment benchmarks

## Results

Experiments grounded in our theoretical analysis confirm that the quality and diversity of rejected responses critically impact DPO performance. A simple improvement method based on these findings achieves consistent gains over standard DPO.

</div>
