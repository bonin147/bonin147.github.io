---
layout: post
title: "Beyond RAG vs. Long-Context: Distraction-Aware Retrieval로 효율적인 Knowledge Grounding 달성하기"
title_en: "Beyond RAG vs. Long-Context: Learning Distraction-Aware Retrieval for Efficient Knowledge Grounding"
date: 2026-01-17
categories: [NLP]
subcategory: "Paper Review"
tags: [RAG, Long-Context, LLM, Knowledge-Grounding, ICLR]
description: "RAG와 Long-Context의 이분법을 넘어, distraction-aware retrieval을 통해 효율적인 knowledge grounding을 달성하는 방법을 제안합니다."
bilingual: true
paper_url: "https://arxiv.org/abs/2509.21865"
paper_venue: "ICLR 2026"
paper_authors: "Seong-Woong Shim, Myunsoo Kim, Jae Hyeon Cho, Byungjun Lee"
---

<div class="lang-ko" markdown="1">

> **추천 점수: 4 / 5 (Recommend)**
>
> *RAG와 Long-Context의 이분법을 깨고, retrieval 자체를 distraction-aware하게 학습시킨 실용적 접근.*

## 개요

RAG(Retrieval Augmented Generation)와 Long-Context 접근법은 각각 한계가 있다. RAG는 retriever의 품질에 의존하고, long-context는 불필요한 정보(distraction)까지 모델에 주입한다.

본 논문은 이 이분법을 넘어서, **retriever가 distraction을 인식하고 필터링하는 능력을 학습**하도록 하여, 효율적이면서도 정확한 knowledge grounding을 달성한다.

## 핵심 기여

- **Distraction-Aware Retrieval**: Retrieved passage 중 실제로 답변에 필요한 정보만 선별하는 학습 기반 접근
- **효율성과 정확도의 균형**: Long-context 대비 적은 토큰으로 동등 이상의 성능 달성
- **실용적 프레임워크**: 기존 RAG 파이프라인에 최소한의 수정으로 적용 가능

## 실험 결과

다양한 knowledge-intensive 벤치마크에서 기존 RAG와 long-context 방식 모두를 상회하는 성능을 보였으며, 특히 distractor가 많은 환경에서 강건한 성능을 유지했다.

</div>

<div class="lang-en" markdown="1">

> **Rating: 4 / 5 (Recommend)**
>
> *Breaks the RAG vs. Long-Context dichotomy by teaching the retriever to be distraction-aware — practical and effective.*

## Overview

RAG (Retrieval Augmented Generation) and long-context approaches each have limitations. RAG depends on retriever quality, while long-context injects unnecessary information (distractions) into the model.

This paper moves beyond this dichotomy by **teaching the retriever to recognize and filter distractions**, achieving efficient and accurate knowledge grounding.

## Key Contributions

- **Distraction-Aware Retrieval**: A learning-based approach that selects only the passages actually needed for answering
- **Balancing Efficiency and Accuracy**: Achieves comparable or better performance than long-context with fewer tokens
- **Practical Framework**: Can be applied to existing RAG pipelines with minimal modifications

## Results

The method outperforms both standard RAG and long-context approaches across various knowledge-intensive benchmarks, showing particularly robust performance in distraction-heavy environments.

</div>
