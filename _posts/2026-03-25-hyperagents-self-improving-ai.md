---
layout: post
title: "HyperAgents: 스스로 개선하는 AI 시스템의 새로운 패러다임"
title_en: "HyperAgents: A New Paradigm for Self-Improving AI Systems"
date: 2026-03-25
categories: [LLM]
subcategory: "Paper Review"
tags: [Self-Improvement, OpenEndedness, MetaLearning, Agent, Meta]
description: "Meta의 HyperAgents 논문을 소개합니다. 자기 자신을 수정하고 개선할 수 있는 자기참조적 AI 에이전트 프레임워크입니다."
bilingual: true
---

<div class="lang-ko" markdown="1">

Meta에서 2026년 3월에 발표한 **HyperAgents** 논문을 소개합니다. 이 논문은 AI 에이전트가 **자기 자신의 개선 방법까지 스스로 수정**할 수 있는 자기참조적(self-referential) 프레임워크를 제안합니다.

## 핵심 문제: 고정된 Meta Agent의 한계

기존의 자기 개선(self-improving) AI 시스템들은 대부분 **고정된 meta-level 메커니즘**에 의존합니다. 예를 들어, Darwin Gödel Machine(DGM)은 코딩 에이전트가 자신의 코드를 수정하며 개선하는 구조인데, 이 개선 방법 자체는 사람이 설계한 고정된 instruction-generation 메커니즘에 의해 결정됩니다.

이 구조는 코딩 도메인에서는 잘 작동합니다. 코딩 능력이 향상되면 자기 수정 능력도 함께 향상되기 때문입니다. 하지만 **논문 리뷰, 로봇 제어, 수학 채점** 같은 다른 도메인에서는 task를 잘 수행하는 능력과 자기를 개선하는 능력이 반드시 일치하지 않습니다.

## HyperAgent란?

HyperAgent는 **Task Agent**와 **Meta Agent**를 하나의 수정 가능한 프로그램으로 통합한 자기참조적 에이전트입니다.

- **Task Agent**: 주어진 과제(코딩, 논문 리뷰 등)를 수행
- **Meta Agent**: 에이전트를 수정하고 새로운 에이전트를 생성
- **Hyperagent**: 이 둘을 하나의 편집 가능한 프로그램으로 결합

핵심은 Meta Agent 자체도 수정 대상이라는 점입니다. 즉, **개선하는 방법을 개선**할 수 있습니다. 논문에서는 이를 **Metacognitive Self-Modification**이라고 부릅니다.

## DGM-Hyperagents (DGM-H)

저자들은 기존 DGM의 open-ended exploration 구조를 유지하면서 hyperagent를 도입한 **DGM-H**를 제안합니다.

작동 방식은 다음과 같습니다:

1. **Archive 초기화**: 단일 hyperagent로 시작
2. **Parent 선택**: 성능에 비례하는 확률적 선택
3. **Metacognitive Self-Modification**: 선택된 parent가 자기 자신을 수정하여 새로운 hyperagent 생성
4. **평가**: 새로운 hyperagent를 task에서 평가
5. **Archive에 추가**: 평가된 hyperagent를 archive에 저장
6. **반복**: 2-5를 반복하며 점진적으로 개선

## 실험 결과

DGM-H는 4가지 도메인에서 평가되었습니다:

### 코딩 (Polyglot Benchmark)
- DGM과 유사한 수준의 성능 향상 달성
- 코딩에 특화되지 않았음에도 경쟁력 있는 결과

### 논문 리뷰
- 초기 agent의 0.0에서 **0.710**까지 성능 향상
- 오픈소스 static baseline(0.630)을 상회
- 단순한 프롬프트 수정에서 **체계적인 다단계 평가 파이프라인**으로 진화

### 로봇 보상 함수 설계
- 초기 0.060에서 **0.372**까지 향상
- 직접 최적화 metric을 사용하는 기본 보상 함수(0.348)도 초과
- 환경 제약 조건, 유효 상태 변수 등의 **내부 지식 베이스**를 자체적으로 구축

### 도메인 간 전이 (Cross-domain Transfer)
가장 흥미로운 결과는 **전이 학습** 실험입니다:
- 논문 리뷰 + 로봇 도메인에서 학습된 hyperagent를 **올림피아드 수준 수학 채점**에 적용
- 기존 DGM은 전이 시 imp@50 ≈ 0 (개선 없음)
- DGM-H는 imp@50 = **0.630**으로 상당한 개선 달성

이는 DGM-H가 **도메인에 독립적인 범용 자기 개선 전략**을 학습했음을 의미합니다.

## Meta Agent가 스스로 발견한 개선들

DGM-H의 meta agent가 자체적으로 발견한 대표적인 개선 사항들:

- **Performance Tracking**: 세대별 성능을 추적하고 개선 추세를 분석하는 시스템
- **Persistent Memory**: 수치적 점수 외에 인과적 가설, 전략적 계획을 저장하여 이전 세대의 발견을 활용
- **Multi-stage Evaluation Pipeline**: 단순 프롬프트 수정에서 체크리스트, 의사결정 규칙을 갖춘 구조적 평가 시스템으로 진화

## 의의와 한계

**의의:**
- 코딩 이외의 도메인에서도 open-ended self-improvement 가능성을 입증
- 개선 방법 자체가 개선될 수 있는 metacognitive self-modification 도입
- 학습된 self-improvement 전략이 도메인 간 전이 가능

**한계:**
- 여전히 FM(Foundation Model) 호출에 의존
- Archive 관리, parent selection 등 일부 구성 요소는 고정
- Safety와 관련된 근본적인 과제가 남아 있음

## 마무리

HyperAgents는 "더 나은 솔루션을 찾는 것"을 넘어 **"개선하는 방법을 개선하는"** AI 시스템의 가능성을 보여줍니다. Self-improving AI가 코딩 도메인을 넘어 범용적으로 작동할 수 있다는 것을 실증적으로 보여준 의미 있는 연구입니다.

> **논문 정보**
> - 제목: HyperAgents
> - 저자: Jenny Zhang, Bingchen Zhao, Wannan Yang, Jakob Foerster, Jeff Clune, Minqi Jiang, Sam Devlin, Tatiana Shavrina
> - 소속: UBC, Vector Institute, University of Edinburgh, NYU, FAIR at Meta, Meta Superintelligence Labs
> - 코드: [GitHub](https://github.com/facebookresearch/Hyperagents)
> - arXiv: [2603.19461](https://arxiv.org/abs/2603.19461)

</div>

<div class="lang-en" markdown="1">

This post introduces **HyperAgents**, published by Meta in March 2026. The paper proposes a self-referential framework where AI agents can **modify not only how they solve tasks, but also how they improve themselves**.

## The Core Problem: Fixed Meta Agents

Most existing self-improving AI systems rely on **fixed meta-level mechanisms**. For example, the Darwin Gödel Machine (DGM) allows coding agents to modify their own code, but the improvement process itself is determined by a handcrafted, fixed instruction-generation mechanism.

This works well in coding domains, where improving coding ability naturally improves self-modification ability. But in domains like **paper review, robotics, or math grading**, task-solving skills and self-improvement skills are not necessarily aligned.

## What is a HyperAgent?

A HyperAgent is a self-referential agent that integrates a **Task Agent** and a **Meta Agent** into a single editable program.

- **Task Agent**: Solves the target task (coding, paper review, etc.)
- **Meta Agent**: Modifies agents and generates new ones
- **Hyperagent**: Combines both into one editable program

The key insight is that the Meta Agent itself is editable — the system can **improve how it improves**. The authors call this **Metacognitive Self-Modification**.

## DGM-Hyperagents (DGM-H)

The authors extend the original DGM's open-ended exploration structure with hyperagents to create **DGM-H**.

The process works as follows:

1. **Initialize archive** with a single hyperagent
2. **Select parent** proportional to performance
3. **Metacognitive self-modification**: Parent generates a modified version of itself
4. **Evaluate** the new hyperagent on target tasks
5. **Add to archive**
6. **Repeat** steps 2-5 for continued improvement

## Experimental Results

DGM-H was evaluated across four domains:

### Coding (Polyglot Benchmark)
- Achieves gains comparable to the original DGM
- Competitive results despite not being handcrafted for coding

### Paper Review
- Performance improves from 0.0 to **0.710**
- Outperforms the open-sourced static baseline (0.630)
- Evolves from simple prompt tweaks to **structured multi-stage evaluation pipelines**

### Robotics Reward Design
- Improves from 0.060 to **0.372**
- Surpasses the default reward function that directly optimizes the evaluation metric (0.348)
- Autonomously builds an **internal knowledge base** of environment constraints and reward-scaling heuristics

### Cross-domain Transfer
The most compelling result is the **transfer experiment**:
- Hyperagents trained on paper review + robotics were applied to **Olympiad-level math grading**
- Original DGM: imp@50 ≈ 0 (no improvement)
- DGM-H: imp@50 = **0.630** (substantial improvement)

This demonstrates that DGM-H learns **domain-agnostic self-improvement strategies**.

## Self-Discovered Meta Improvements

Notable improvements the meta agent discovered autonomously:

- **Performance Tracking**: Systems to track scores across generations and analyze improvement trends
- **Persistent Memory**: Storage of causal hypotheses and strategic plans beyond numerical scores, enabling future generations to build on past discoveries
- **Multi-stage Evaluation Pipelines**: Evolution from simple prompt tweaks to structured evaluation with checklists and decision rules

## Significance and Limitations

**Significance:**
- Demonstrates open-ended self-improvement beyond coding domains
- Introduces metacognitive self-modification where the improvement process itself evolves
- Shows learned self-improvement strategies transfer across domains

**Limitations:**
- Still relies on Foundation Model (FM) calls
- Some components (archive management, parent selection) remain fixed
- Fundamental safety challenges remain open

## Takeaway

HyperAgents moves beyond "searching for better solutions" to **"improving how the system searches for improvements."** This work provides empirical evidence that self-improving AI can work as a general-purpose framework beyond coding, marking a significant step toward open-ended AI systems.

> **Paper Info**
> - Title: HyperAgents
> - Authors: Jenny Zhang, Bingchen Zhao, Wannan Yang, Jakob Foerster, Jeff Clune, Minqi Jiang, Sam Devlin, Tatiana Shavrina
> - Affiliations: UBC, Vector Institute, University of Edinburgh, NYU, FAIR at Meta, Meta Superintelligence Labs
> - Code: [GitHub](https://github.com/facebookresearch/Hyperagents)
> - arXiv: [2603.19461](https://arxiv.org/abs/2603.19461)

</div>
