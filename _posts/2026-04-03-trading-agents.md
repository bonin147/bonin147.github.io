---
layout: post
title: "TradingAgents: 실제 증권사를 모방한 Multi-Agent 트레이딩 프레임워크"
title_en: "TradingAgents: Multi-Agent LLM Financial Trading Framework Mimicking Real Brokerages"
date: 2026-04-03
categories: [LLM]
subcategory: "Paper Review"
tags: [LLM, Agent, Multi-Agent, Finance, Trading, Backtesting]
description: "여러 전문 Agent가 토론과 검증을 거쳐 매매 결정을 내리는 TradingAgents 프레임워크를 리뷰합니다."
bilingual: true
arxiv_url: "https://arxiv.org/abs/2412.20138"
---

<div class="lang-ko" markdown="1">

최근 LLM에는 Agent라는 키워드가 대세이다. LLM을 이용해 Financial Trading하려는 노력은 꾸준히 있었다. 이 논문에서는 하나의 LLM이 모든 판단을 하는 것이 아닌, 여러 specialist agent를 만들고 여러 agent가 토론과 검증을 거쳐 매매 결정을 내리게 하여 **실제 증권사를 모방하는 Framework**를 제안한다.

---

## Background

기존 Financial AI 연구는 대부분
- 단일 Agent 중심 접근
- 데이터 수집 → 활용 정도의 간단한 agent 접근

정도에 머물렀다.

저자는 기존 금융 multi agent system의 limitation으로 두 가지를 언급한다.
1. 현실의 조직(증권사)을 반영하지 못하고 있다.
2. 자연어만으로 agent가 소통을 하고 있다.

현실의 조직은 다양한 관점의 팀이 데이터를 모으고, 반대 의견을 내고, 리스크를 점검하고, 최종적으로 집행하지만, 기존의 연구는 이런 구조를 반영하지 못했다.

자연어로 의사 소통을 하면 정보가 쉽게 손실된다. 메세지가 길어질수록 중요한 포인트가 묻히고, 이전 맥락이 왜곡된다. 이 논문에서는 자연어 대화로만 쓰지 않고, 구조화된 문서(report) 기반 커뮤니케이션을 함께 도입한다.

## Overview

이 논문에서는, Agent를 실제 Trading 조직처럼 구성한다.

애널리스트 포지션이 분석을 하면, 리서처가 찬반 토론을 하고, 트레이더가 매매 판단을 내리면, 리스크 매니저가 리스크를 조정하고, 매니저가 최종 승인을 하는 형태이다.

이 논문에서는 LLM이 주식을 잘 맞추기 보다는, LLM Agent를 어떻게 조직하면 각 의사결정에 대해 더욱 설명이 가능해지는가? 에 초점을 맞춘다.

----

## TradingAgents

<img src="/assets/images/trading-agents/overview.png" alt="TradingAgents Framework Overview" style="width:100%; margin: 1.5rem 0;">

Trading Agents 구성:

**1) Analyst Team**
- 펀더멘탈(Fundamental) 애널리스트: 재무제표, 실적 등을 분석
- Sentiment 애널리스트: 소셜 미디어 반응, 대중 심리 분석
- News 애널리스트: 뉴스, 거시경제, 정책 변화 분석
- Technical 애널리스트: MACD, RSI, 볼린저 밴드 등 기술적 지표 분석

이 단계의 역할은 각자 전문 영역에서 짧고 구조화된 리포트를 만드는 것이다. 긴 대화를 주고 받는 것이 아니라, 이후 단계에서 재사용될 형식으로 결과를 남긴다.

**2) Researcher Team**
- Bullish Researcher: 롱쟁이
- Bearish Researcher: 숏쟁이

가 서로 토론한다. 상승 논리와 하락 논리를 충돌시켜 균형 잡힌 뷰를 확보하려는 독특한 설계이다.

**3) Trader**

트레이더는 애널리스트 리포트와 리서치 팀의 결과를 종합해 매수, 매도, 홀드와 같은 시그널을 생성한다. 추가적으로 언제, 얼마나 거래할지도 결정한다.

**4) Risk Management Team**

이 부분도 특이했는데, 리스크 팀이 Trader의 거래 규모나 포지션을 보고 검토한다. Researcher Team과 같이 (공격적인 성향 / 중립적인 성향 / 보수적인 성향) 으로 나누어서 검토한다. Risk Management Team의 의견까지 받고, 최종적으로는 Fund Manager가 승인 및 집행한다.

----

디테일 포인트
- 애널리스트는 보고서의 형태로 작성
- 리서처는 토론 후 결론을 구조화해 저장
- 트레이더는 판단 근거를 문서화
- 리스크팀은 금액 조정 논리를 정리

→ Long Context에 의한 정보 왜곡을 줄이고, 보는 사람 입장에서도 추론 경로를 점검하기 쉽다.

- 빠른 처리와 요약, 데이터 조회 같은 작업은 quick-thinking model 사용
- 의사결정, 리포트 작성, 복합 추론은 deep-thinking model 사용

→ 이는 API token 비용 등도 고려하여 설계했음을 알 수 있다.

----

## Results

2024년 1월 1일부터 3월 29일까지의 백테스트로 진행되었다.
종목은 Apple, Google, Amazon 같은 대형 기술주이다.

<img src="/assets/images/trading-agents/chart.png" alt="Trading Signal Chart" style="width:100%; margin: 1.5rem 0;">

초록색은 long, 빨간색은 short이다. 꽤 고수임을 알 수 있다. (물론 3달 정도의 백테스트이고, cherry picking한 결과이긴 하겠지만)

B&H = Buy and Hold로 그냥 장투를 의미한다.
MACD, KDJ&RSI, ZMR, SMA는 Rule 기반 투자 방식이다.

**AAPL**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | -5.23 | -5.09 | -1.29 | 11.90 |
| Rule-based | MACD | -1.49 | -1.48 | -0.81 | 4.53 |
| Rule-based | KDJ&RSI | 2.05 | 2.07 | 1.64 | 1.09 |
| Rule-based | ZMR | 0.57 | 0.57 | 0.17 | 0.86 |
| Rule-based | SMA | -3.20 | -2.97 | -1.72 | 3.67 |
| **Ours** | **TradingAgents** | **26.62** | **30.50** | **8.21** | **0.91** |
| | Improvement | +24.57 | +28.43 | +6.57 | — |

**GOOGL**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | 7.78 | 8.09 | 1.35 | 13.04 |
| Rule-based | MACD | 6.20 | 6.26 | 2.31 | 1.22 |
| Rule-based | KDJ&RSI | 0.40 | 0.40 | 0.02 | 1.58 |
| Rule-based | ZMR | -0.58 | 0.58 | 2.12 | 2.34 |
| Rule-based | SMA | 6.23 | 6.43 | 2.12 | 2.34 |
| **Ours** | **TradingAgents** | **24.36** | **27.58** | **6.39** | **1.69** |
| | Improvement | +16.58 | +19.49 | +4.26 | — |

**AMZN**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | 17.10 | 17.60 | 3.53 | 3.80 |
| Rule-based | KDJ&RSI | -0.77 | -0.76 | -2.25 | 1.08 |
| Rule-based | ZMR | -0.77 | -0.77 | -2.45 | 0.82 |
| Rule-based | SMA | 11.01 | 11.60 | 2.22 | 3.97 |
| **Ours** | **TradingAgents** | **23.21** | **24.90** | **5.60** | **2.11** |
| | Improvement | +6.10 | +7.30 | +2.07 | — |

TradingAgents가 기존의 알고리즘 대비 CR(누적 수익률), ARR(연간 수익률), SR(샤프 지수)에서 우수한 모습을 보여주었다. MDD(최대 낙폭) 역시 준수한 수준이다.

<img src="/assets/images/trading-agents/cumulative return.png" alt="Cumulative Returns Comparison" style="width:100%; margin: 1.5rem 0;">

날짜에 따른 누적 수익이다.

## Conclusion

전반적인 framework 설계 자체가 꽤 재밌었다. 구현하는 디테일도 많은 생각을 하면서 만든 것 같다. 어렵지 않은 paper이니 관심이 있다면, 직접 읽어보길 추천한다.

</div>

<div class="lang-en" markdown="1">

The keyword "Agent" is dominating the recent LLM landscape. There have been consistent efforts to use LLMs for financial trading. This paper proposes a framework that does not rely on a single LLM for all decisions, but instead creates multiple specialist agents that engage in discussion and verification before making trading decisions — effectively **mimicking a real brokerage firm**.

---

## Background

Most existing Financial AI research has been limited to:
- Single-agent approaches
- Simple agent pipelines that merely collect and utilize data

The authors point out two key limitations of existing financial multi-agent systems:
1. They fail to reflect real-world organizational structures (like brokerage firms).
2. Agents communicate only through natural language.

In real organizations, teams with diverse perspectives gather data, raise opposing views, review risks, and ultimately execute decisions. Existing research has not captured this structure.

When communicating solely through natural language, information is easily lost. As messages grow longer, key points get buried and prior context becomes distorted. This paper introduces structured document (report)-based communication alongside natural language dialogue.

## Overview

This paper organizes agents like a real trading organization.

Analysts perform their analysis, researchers debate bull and bear cases, traders make trading decisions, risk managers adjust for risk, and the manager gives final approval.

Rather than focusing on whether LLMs can accurately predict stock movements, this paper asks: **how should LLM agents be organized so that each decision becomes more explainable?**

----

## TradingAgents

<img src="/assets/images/trading-agents/overview.png" alt="TradingAgents Framework Overview" style="width:100%; margin: 1.5rem 0;">

TradingAgents composition:

**1) Analyst Team**
- Fundamental Analyst: Analyzes financial statements, earnings, etc.
- Sentiment Analyst: Analyzes social media reactions and public sentiment
- News Analyst: Analyzes news, macroeconomic conditions, and policy changes
- Technical Analyst: Analyzes technical indicators like MACD, RSI, and Bollinger Bands

The role of this stage is for each specialist to produce short, structured reports. Rather than engaging in long conversations, they leave results in a format that can be reused in subsequent stages.

**2) Researcher Team**
- Bullish Researcher: Argues the long case
- Bearish Researcher: Argues the short case

They debate each other. This is a unique design that aims to secure a balanced view by colliding bullish and bearish arguments.

**3) Trader**

The trader synthesizes the analyst reports and research team's results to generate signals such as buy, sell, or hold. Additionally, the trader decides when and how much to trade.

**4) Risk Management Team**

Interestingly, the risk team reviews the trader's position sizes and holdings. Similar to the Researcher Team, they are divided into (aggressive / neutral / conservative) perspectives for review. After receiving the Risk Management Team's input, the Fund Manager gives final approval and execution.

----

Key design details:
- Analysts produce structured reports
- Researchers save structured conclusions after debate
- Traders document their decision rationale
- The risk team formalizes position adjustment logic

→ This reduces information distortion from long contexts and makes it easier to audit the reasoning chain.

- Quick-thinking models handle fast processing, summarization, and data retrieval
- Deep-thinking models handle decision-making, report writing, and complex reasoning

→ This shows the framework was designed with API token costs in mind.

----

## Results

Backtested from January 1 to March 29, 2024.
Stocks tested include large-cap tech names like Apple, Google, and Amazon.

<img src="/assets/images/trading-agents/chart.png" alt="Trading Signal Chart" style="width:100%; margin: 1.5rem 0;">

Green indicates long positions, red indicates short positions. The signals look quite skilled. (Though of course this is only a 3-month backtest and likely cherry-picked results.)

B&H = Buy and Hold, meaning simple long-term holding.
MACD, KDJ&RSI, ZMR, and SMA are rule-based trading strategies.

**AAPL**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | -5.23 | -5.09 | -1.29 | 11.90 |
| Rule-based | MACD | -1.49 | -1.48 | -0.81 | 4.53 |
| Rule-based | KDJ&RSI | 2.05 | 2.07 | 1.64 | 1.09 |
| Rule-based | ZMR | 0.57 | 0.57 | 0.17 | 0.86 |
| Rule-based | SMA | -3.20 | -2.97 | -1.72 | 3.67 |
| **Ours** | **TradingAgents** | **26.62** | **30.50** | **8.21** | **0.91** |
| | Improvement | +24.57 | +28.43 | +6.57 | — |

**GOOGL**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | 7.78 | 8.09 | 1.35 | 13.04 |
| Rule-based | MACD | 6.20 | 6.26 | 2.31 | 1.22 |
| Rule-based | KDJ&RSI | 0.40 | 0.40 | 0.02 | 1.58 |
| Rule-based | ZMR | -0.58 | 0.58 | 2.12 | 2.34 |
| Rule-based | SMA | 6.23 | 6.43 | 2.12 | 2.34 |
| **Ours** | **TradingAgents** | **24.36** | **27.58** | **6.39** | **1.69** |
| | Improvement | +16.58 | +19.49 | +4.26 | — |

**AMZN**

| Category | Model | CR% | ARR% | SR | MDD% |
|----------|-------|-----|------|----|------|
| Market | B&H | 17.10 | 17.60 | 3.53 | 3.80 |
| Rule-based | KDJ&RSI | -0.77 | -0.76 | -2.25 | 1.08 |
| Rule-based | ZMR | -0.77 | -0.77 | -2.45 | 0.82 |
| Rule-based | SMA | 11.01 | 11.60 | 2.22 | 3.97 |
| **Ours** | **TradingAgents** | **23.21** | **24.90** | **5.60** | **2.11** |
| | Improvement | +6.10 | +7.30 | +2.07 | — |

TradingAgents demonstrated superior performance over existing algorithms in CR (Cumulative Return), ARR (Annualized Return Rate), and SR (Sharpe Ratio). MDD (Maximum Drawdown) was also at a respectable level.

<img src="/assets/images/trading-agents/cumulative return.png" alt="Cumulative Returns Comparison" style="width:100%; margin: 1.5rem 0;">

Cumulative returns over time.

## Conclusion

The overall framework design itself was quite interesting. The implementation details show a lot of thoughtful design. It's not a difficult paper, so if you're interested, I recommend reading it directly.

</div>
