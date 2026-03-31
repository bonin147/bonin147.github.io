---
layout: post
title: "한국어 특화 미묘한 욕설 데이터 생성 파이프라인"
title_en: "K/DA: Automated Data Generation Pipeline for Detoxifying Implicitly Offensive Language in Korean"
date: 2025-07-27
categories: [NLP]
subcategory: "Paper Review"
tags: [My-Paper, Korean-NLP, Detoxification, Data-Generation, ACL]
description: "한국어 특화 미묘한 욕설 자동으로 생성하는 파이프라인을 제안합니다."
bilingual: true
paper_url: "https://aclanthology.org/2025.acl-long.1039/"
arxiv_url: "https://arxiv.org/abs/2506.13513"
paper_venue: "ACL 2025"
paper_authors: "Minkyeong Jeon*, Hyemin Jeong*, Yerang Kim, Jiyoung Kim, Jae Hyeon Cho, Byung-Jun Lee"
---

<div class="lang-ko" markdown="1">

## 개요

한국어에는 문맥에 따라 의미가 달라지는 암묵적(implicitly) 혐오 표현이 많다. 기존의 영어 중심 detoxification 연구는 이런 한국어 특수성을 반영하지 못한다.

**K/DA**는 한국어 암묵적 혐오 표현을 자동으로 탐지하고 정화하기 위한 **데이터 생성 파이프라인**을 제안한다. LLM을 활용하여 대규모의 고품질 학습 데이터를 자동 생성함으로써, 별도의 수작업 annotation 없이도 효과적인 detoxification 모델을 학습할 수 있다.

## 핵심 기여

- **자동 데이터 생성**: LLM 기반 파이프라인으로 한국어 혐오 표현-정화 쌍 데이터를 대규모 생성
- **암묵적 혐오 표현 처리**: 명시적 욕설뿐 아니라, 맥락 의존적 혐오 표현까지 커버
- **실용적 성능**: 생성된 데이터로 학습한 모델이 수작업 데이터 기반 모델과 동등 이상의 성능 달성

## 실험 결과

제안한 파이프라인으로 생성된 데이터를 활용한 detoxification 모델은 기존 방법 대비 높은 정화 품질과 자연스러움을 보였으며, 특히 암묵적 표현에서 큰 성능 향상을 기록했다.

</div>

<div class="lang-en" markdown="1">

## Overview

Korean has many implicitly offensive expressions whose meaning shifts with context. Existing English-centric detoxification research fails to capture these Korean-specific nuances.

**K/DA** proposes an **automated data generation pipeline** for detecting and detoxifying implicit hate speech in Korean. By leveraging LLMs to automatically generate large-scale, high-quality training data, it enables effective detoxification models without manual annotation.

## Key Contributions

- **Automated Data Generation**: LLM-based pipeline generating large-scale Korean offensive-clean text pairs
- **Implicit Hate Speech Handling**: Covers not just explicit profanity but also context-dependent offensive expressions
- **Practical Performance**: Models trained on generated data match or exceed manually annotated baselines

## Results

The detoxification model trained on pipeline-generated data showed higher detoxification quality and naturalness compared to existing methods, with particularly significant improvements on implicit expressions.

</div>
