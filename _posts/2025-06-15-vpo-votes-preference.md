---
layout: post
title: "VPO: 투표 수를 활용한 Preference Optimization"
title_en: "VPO: Leveraging the Number of Votes in Preference Optimization"
date: 2025-06-15
categories: [NLP]
subcategory: "Paper Review"
tags: [My-Paper, Preference-Optimization, RLHF, DPO, Voting, Computational-Linguistics]
description: "Preference 데이터의 투표 수 정보를 활용하여 더 정교한 preference optimization을 달성합니다."
bilingual: true
paper_url: "https://direct.mit.edu/coli/article/doi/10.1162/COLI.a.579/133943/VPO-Leveraging-the-Number-of-Votes-in-Preference"
arxiv_url: "https://arxiv.org/abs/2410.22891"
paper_venue: "Computational Linguistics 2025"
paper_authors: "Jae Hyeon Cho, Minkyung Park, Byungjun Lee"
---

<div class="lang-ko" markdown="1">

## 개요

기존 preference optimization(DPO 등)은 "A가 B보다 좋다"는 binary 비교만 활용한다. 하지만 실제 preference 데이터(예: Anthropic HH, Chatbot Arena)에는 **annotator들의 투표 수** 정보가 함께 제공된다.

5명 중 5명이 A를 선택한 경우와 3명만 A를 선택한 경우, preference의 확실성은 분명히 다르다. **VPO(Vote-based Preference Optimization)**는 이 투표 수 정보를 optimization objective에 직접 반영하여, 더 정교한 alignment을 달성한다.

## 핵심 기여

- **투표 수 기반 가중치**: Preference의 확실성에 비례하여 학습 신호의 강도를 조절
- **이론적 정당성**: Bradley-Terry 모델의 자연스러운 확장으로서의 수학적 근거 제시
- **범용 적용**: DPO, IPO 등 다양한 preference optimization 방법에 플러그인 형태로 적용 가능

## 실험 결과

Anthropic HH, UltraFeedback 등 실제 투표 수 정보가 있는 데이터셋에서, VPO는 기존 DPO 대비 일관된 성능 향상을 달성했다. 특히 투표 분포가 극단적(만장일치 vs. 박빙)인 샘플에서 차이가 두드러졌다.

</div>

<div class="lang-en" markdown="1">

## Overview

Existing preference optimization methods (DPO, etc.) only use binary comparisons: "A is better than B." But real preference datasets (e.g., Anthropic HH, Chatbot Arena) come with **annotator vote counts**.

When 5 out of 5 annotators choose A versus only 3, the certainty of the preference clearly differs. **VPO (Vote-based Preference Optimization)** directly incorporates this vote count information into the optimization objective for more precise alignment.

## Key Contributions

- **Vote-Based Weighting**: Adjusts learning signal strength proportional to preference certainty
- **Theoretical Justification**: Mathematical grounding as a natural extension of the Bradley-Terry model
- **Universal Applicability**: Can be applied as a plugin to various preference optimization methods including DPO and IPO

## Results

On datasets with real vote count information (Anthropic HH, UltraFeedback), VPO achieves consistent improvements over standard DPO. The gains are especially pronounced for samples with extreme vote distributions (unanimous vs. close calls).

</div>
