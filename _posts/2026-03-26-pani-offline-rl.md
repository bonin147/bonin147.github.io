---
layout: post
title: "PANI: Offline RL에서 가장 저평가된 알고리즘 — Diffusion 없이 이긴다"
title_en: "PANI: The Most Underrated Algorithm in Offline RL — Beating Diffusion Without Diffusion"
date: 2026-03-26
categories: [RL]
subcategory: "Paper Review"
tags: [Offline-RL, PANI, Diffusion, Noise-Injection, D4RL, IQL, TD3]
description: "Diffusion 모델의 복잡함 없이, 노이즈 한 줄로 Offline RL의 OOD 문제를 해결한 PANI를 소개합니다."
bilingual: true
---

<div class="lang-ko" markdown="1">

> **추천 점수: 5 / 5 (Strong Recommend)**
>
> *Diffusion의 핵심을 코드 3줄로 distill해냈다 — 이론, 성능, 효율을 모두 잡은 Offline RL의 숨은 보석.*

## 왜 이 논문이 저평가되었는가

Offline RL 커뮤니티는 지금 **Diffusion에 빠져 있다.** Diffusion-QL, QGPO, SRPO, DTQL — 학회마다 쏟아지는 diffusion 기반 offline RL 논문들은 점점 더 복잡한 generative model을 쌓아 올리며 성능표를 채우고 있다. 하지만 한 가지 불편한 질문을 던져보자:

> **그 복잡한 denoising process가 정말 필요한가?**

Korea University의 Oh & Lee가 발표한 **PANI (Penalized Action Noise Injection)**는 이 질문에 **"아니오"**라고 답한다. 그것도 **코드 3줄 수정**으로.

## 핵심 아이디어: Embarrassingly Simple

PANI를 한 문장으로 요약하면:

> **Dataset action에 noise를 주입하고, noise 크기만큼 Q-value에 penalty를 준다.**

수식으로 쓰면:

$$\bar{y}(s, a, s', a') = y(s, a, s') - \|a - a'\|_2^2$$

\(a'\)은 dataset action \(a\)에 noise를 주입한 perturbed action이다. 끝이다. **이게 전부다.**

기존 Q-learning objective에서 (1) noise를 주입한 action으로 Q-network를 업데이트하고, (2) 원래 action에서 멀어진 만큼 target value를 깎는다. OOD action에 대한 Q-value overestimation을 **구조적으로 억제**하는 방식이다.

## Diffusion이 실제로 하는 일

왜 이것이 작동하는지 이해하려면, **diffusion model이 offline RL에서 실제로 무슨 역할을 하는지** 뜯어볼 필요가 있다.

Diffusion model은 본질적으로 **multi-scale noise를 활용한 score matching**이다. Denoising Score Matching(DSM)은 데이터에 noise를 주입해, 데이터가 sparse한 영역에서도 학습 signal을 만들어낸다. Diffusion은 이를 여러 noise level에서 반복하여 더 안정적으로 만든 것뿐이다.

PANI의 저자들은 이 **핵심 메커니즘**을 정확히 포착했다:

1. **Noise injection** → action space 전체로 학습 signal을 전파
2. **Noise 크기에 비례한 penalty** → 데이터에서 먼 action일수록 Q-value 억제
3. **Multi-scale noise (Hybrid Distribution)** → 다양한 noise level의 장점을 결합

Diffusion이 복잡한 generative process를 통해 암묵적으로 달성하는 것을, PANI는 **명시적이고 직접적으로** 달성한다.

## 이론적 기반: Noisy Action MDP

PANI가 단순한 heuristic이 아닌 이유는 **Noisy Action MDP (NAMDP)**라는 이론적 framework 때문이다.

저자들은 PANI objective를 최소화하는 것이 **modified MDP에서의 Q-learning과 동치**임을 증명한다 (Theorem 5.3). NAMDP에서는:

- **Transition probability** \(P_\sigma\)가 noise distribution을 통해 재정의된다
- **Reward function** \(R_\sigma\)에 noise 크기만큼의 penalty가 내장된다
- Noise distribution의 support가 전체 action space를 커버하므로, **OOD 영역에서도 Q-value가 업데이트**된다

이는 CQL이나 TD3+BC 같은 기존 conservative 방법론과 근본적으로 다르다. 기존 방법들은 OOD action을 **회피**하지만, PANI는 OOD 영역에 **직접 진입하여 Q-value를 교정**한다.

## 성능: 숫자로 보자

D4RL benchmark 결과는 솔직히 **당혹스러울 정도**다.

### Gym-MuJoCo (medium dataset 평균)

| Algorithm | 평균 점수 | 비고 |
|---------|----------|------|
| IQL (baseline) | 77.0 | — |
| **IQL-AN (PANI 적용)** | **87.2** | **+10.2** |
| TD3+BC (baseline) | 72.1 | — |
| **TD3-AN (PANI 적용)** | **90.2** | **+18.1** |
| Diffusion-QL | 88.0 | Diffusion 기반 |
| QGPO | 86.6 | Diffusion 기반 |
| SRPO | 87.1 | Diffusion 기반 |
| DTQL | 88.7 | Diffusion 기반 |

**TD3에 PANI를 적용한 TD3-AN이 모든 diffusion 기반 방법을 이긴다.** IQL-AN 역시 대부분의 diffusion 방법과 동등하거나 그 이상이다.

### AntMaze

| Algorithm | 평균 점수 |
|---------|----------|
| IQL | 58.4 |
| TD3+BC | 28.8 |
| **IQL-AN** | **68.5 (+10.1)** |
| **TD3-AN** | **77.7 (+48.9)** |
| QGPO | 78.3 |

TD3+BC에서 TD3-AN으로 **+48.9** 점프는 거의 믿기 어려운 수치다. PANI 하나로 완전히 망가져 있던 알고리즘이 SOTA급으로 부활한다.

## Compute Cost: 진짜 핵심

| Algorithm | Training Time (10⁶ steps) | Overhead |
|---------|---------------------|---------|
| IQL | 9m 33s | — |
| IQL-AN | 10m 22s | **+9%** |
| TD3 | 15m 45s | — |
| TD3-AN | 16m 14s | **+3%** |

**3~9% training time 증가로 10~50점 성능 향상.** Diffusion 기반 방법들이 수 배의 학습 시간과 inference 시 denoising을 요구하는 것과 비교하면, 사실상 공짜다.

더 중요한 것은 **inference time**이다. Diffusion policy는 매번 여러 step의 denoising을 수행해야 하지만, PANI는 기존 MLP policy를 그대로 쓴다. Real-time system이나 resource 제한 환경에서 이 차이는 결정적이다.

## Hybrid Noise Distribution: 유일한 설계 선택

PANI에서 유일하게 non-trivial한 설계 선택은 **Hybrid Noise Distribution**이다.

단순한 Gaussian noise의 문제:
- **Noise가 작으면**: coverage 부족, OOD 문제 미해결
- **Noise가 크면**: mode collapse, sample inefficiency

이를 해결하기 위해 두 가지 기법을 결합한다:

1. **Uniform mixture**: \(q_t^A(a' \mid a) = \alpha(t) \cdot U(a' \mid A) + (1-\alpha(t)) \cdot q_t(a' \mid a)\)
2. **Exponential scaling**: Gaussian noise에 exponential scaling을 적용해 **leptokurtic** distribution을 유도

결과적으로 넓은 범위의 noise scale에서 **robust한 성능**을 보인다. Gaussian이나 Laplace distribution이 특정 scale에서 급격히 성능이 떨어지는 반면, Hybrid는 일관성을 유지한다.

## OOD Overestimation 억제: 실증적 증거

OOD overestimation probability \(P(Q(s, a') > Q(s, a))\) (\(a\)는 dataset action, \(a'\)은 uniform sample):

| Dataset | TD3+BC | TD3-AN | IQL | IQL-AN |
|---------|--------|--------|-----|--------|
| halfcheetah-expert | 0.014 | **0.002** | 0.014 | **0.001** |
| halfcheetah-medium | 0.112 | **0.004** | 0.211 | **0.006** |
| hopper-expert | 0.483 | **0.034** | 0.425 | **0.036** |
| walker2d-medium | 0.316 | **0.002** | 0.147 | **0.004** |

OOD overestimation probability가 **10~100배 감소**한다. 단순히 "성능이 좋다"가 아니라, PANI가 **문제의 root cause를 직접 해결**하고 있다는 강한 증거다.

## Peer Review가 인정한 것들

PANI는 ICLR 2026에 제출되었다가 저자들이 자진 철회(withdrawn)했다. 하지만 리뷰 내용을 보면, **비판적인 리뷰어들조차 PANI의 핵심 강점은 인정**하고 있다. 가장 높은 점수(6점)를 준 리뷰어의 평가:

> *"OOD 문제를 **새로운 관점**에서 접근한 점이 인상적이다. PANI가 다른 방법들과 **호환 가능하다는 점은 매우 매력적**이다."*
>
> *"NAMDP라는 이론적 framework를 형식화하고, Hybrid Noise Distribution 개념을 제안하여 **전체 방법론의 이론적 기반을 구축**한 점도 높이 평가한다."*

다른 리뷰어들도 공통적으로 인정한 장점들:

- **"직관적이고 명확한 설명"** — 방법론의 intuition과 이론이 깔끔하게 전달된다 (Reviewer nUxv)
- **"경량 drop-in 수정"** — 기존 알고리즘에 최소한의 변경만으로 통합 가능 (Reviewer mqZV)
- **"Noise와 dataset action 간 거리에 따른 penalty 조절이 직관적으로 smoother Q-value surface를 형성"** — data boundary 주변의 급격한 value 변화를 완화 (Reviewer nUxv)
- **"D4RL locomotion, Adroit, AntMaze, OGBench 등 다양한 benchmark에서 실험"** — 폭넓은 검증 (Reviewer SH7n)

주목할 점은, 철회 사유가 "방법이 안 돼서"가 아니라는 것이다. 저자들의 철회 코멘트:

> *"리뷰어들의 철저하고 통찰력 있는 피드백에 감사드립니다. 상당한 수정이 필요하다는 점을 인지하고 있으며, 모든 코멘트를 반영하여 강화된 버전을 재제출하겠습니다."*

리뷰어들의 주요 비판은 **방법론 자체보다 논문 완성도**에 집중되어 있었다: 관련 연구 비교 부족 (RORL, SAC-RND 등과의 비교), penalty term의 scale sensitivity 분석 부재, 코드 미공개, hyperparameter tuning 공정성 문제 등. **핵심 아이디어의 novelty와 실용성은 가장 비판적인 리뷰어도 부정하지 못했다.**

## 왜 이 논문에 주목해야 하는가

**1. 연구적 관점: Diffusion은 over-engineering일 수 있다**

PANI는 offline RL에서 diffusion model의 성공 요인이 "diffusion이어서"가 아니라, **"noise injection을 통한 action space coverage 확장"**에 있었음을 시사한다. 향후 연구 방향에 대한 중요한 시사점이다.

**2. 실용적 관점: 바로 적용 가능하다**

기존 IQL이나 TD3 코드에 **몇 줄만 추가**하면 된다. 새로운 architecture도, 새로운 training pipeline도, 새로운 inference 과정도 필요 없다. Q-update 단계에서 noise를 넣고 penalty를 주면 끝이다. 리뷰어 표현대로, **"기존 알고리즘의 Q-update step만 바꾸는 lightweight drop-in 수정"**이다.

**3. 범용성: 어디에든 붙일 수 있다**

PANI는 TD3, IQL뿐 아니라 diffusion 기반인 QGPO에도 적용 가능하고, 전부 성능이 오른다. 리뷰어들이 인정했듯 **"다른 방법들과의 호환성은 매우 매력적"**이며, 이는 PANI가 특정 알고리즘에 종속된 trick이 아니라 **범용적 원리**임을 뜻한다.

## 한계와 열린 질문

PANI에도 한계는 있다. 리뷰어들이 제기한 유효한 비판을 포함하면:

- **최적 noise scale 선택**이 dataset에 의존한다. Hybrid distribution이 이를 완화하지만, 완전히 해결하지는 못한다
- **Dataset의 action coverage가 극도로 낮은 경우** noise injection만으로 충분한지는 열린 질문이다. 한 리뷰어가 지적했듯, *"매우 좁은 분포의 expert dataset에서 PANI가 어떻게 작동하는지 더 상세한 논의가 필요하다"*
- **Penalty term \(\|a - a'\|_2^2\)의 scale 문제** — reward scale과 action scale이 다른 task에서 penalty의 상대적 크기가 달라질 수 있다. Scaling hyperparameter \(\alpha\)의 도입이 필요할 수 있다
- **고차원 action space**에서의 scalability와 computational efficiency는 추가 검증 필요
- RORL, SAC-RND, EPQ 등 **유사한 pessimistic/anti-exploration 접근법과의 체계적 비교** 부족

이러한 한계에도, **complexity 대비 performance 비율**에서 PANI를 능가하는 방법은 현재 없다. 논문 완성도는 개선 여지가 있지만, **핵심 아이디어의 가치는 review score와 무관하게 빛난다.**

## 결론: 단순함의 승리

PANI는 최근 몇 년간 Offline RL에서 나온 **가장 과소평가된 논문**이다. Diffusion model의 화려함에 가려져 있지만, 그 본질을 꿰뚫어 **최소한의 수정으로 동등 이상의 성능**을 달성한다.

연구자라면 NAMDP theoretical framework에 주목하라. 실무자라면 지금 당장 기존 코드에 PANI를 적용해보라. **3줄의 코드가 수천 줄의 diffusion implementation을 대체할 수 있다.**

> **"The best ideas are embarrassingly simple — the hard part is realizing they work."**

**Paper**: Oh, J. & Lee, B.-J. (2025). *Offline Reinforcement Learning with Penalized Action Noise Injection*. arXiv:2507.02356. Korea University & Gauss Labs.

</div>

<div class="lang-en" markdown="1">

> **Rating: 5 / 5 (Strong Recommend)**
>
> *Distilled the essence of diffusion into 3 lines — a hidden gem in Offline RL that nails theory, performance, and efficiency all at once.*

## Why This Paper Is Criminally Underrated

The Offline RL community is **drunk on Diffusion.** Diffusion-QL, QGPO, SRPO, DTQL — every conference cycle brings another wave of diffusion-based offline RL papers, stacking ever more complex generative models to fill performance tables. But let's ask one uncomfortable question:

> **Do you actually need that complex denoising process?**

Oh & Lee from Korea University answer this with a resounding **"No"** in their paper **PANI (Penalized Action Noise Injection)**. And they do it with **three lines of code changes**.

## The Core Idea: Embarrassingly Simple

PANI in one sentence:

> **Inject noise into dataset actions. Penalize Q-values by the noise magnitude.**

In math:

$$\bar{y}(s, a, s', a') = y(s, a, s') - \|a - a'\|_2^2$$

Where \(a'\) is a noise-perturbed version of the dataset action \(a\). That's it. **That's the whole method.**

Take the standard Q-learning objective, (1) update the Q-network using noise-perturbed actions, and (2) subtract the squared distance from the original action in the target value. This **structurally suppresses** Q-value overestimation for OOD actions.

## What Diffusion Actually Does (If You Squint)

To understand why this works, you need to decompose **what diffusion models are actually doing** in offline RL.

Diffusion models are fundamentally **multi-scale noise score matching**. Denoising Score Matching (DSM) injects noise into data to create learning signals even in sparse regions. Diffusion extends this by repeating across multiple noise levels for stability.

The PANI authors identified this **essential mechanism** precisely:

1. **Noise injection** → propagates learning signals across the entire action space
2. **Noise-proportional penalty** → suppresses Q-values for actions far from data
3. **Multi-scale noise (Hybrid Distribution)** → combines benefits of different noise levels

What diffusion achieves implicitly through a complex generative process, PANI achieves **explicitly and directly**.

## Theoretical Foundation: The Noisy Action MDP

PANI is not just a heuristic — it has a rigorous theoretical backbone in the **Noisy Action MDP (NAMDP)**.

The authors prove (Theorem 5.3) that minimizing the PANI objective is **equivalent to Q-learning in a modified MDP** where:

- **Transition probabilities** \(P_\sigma\) are redefined through the noise distribution
- **Reward function** \(R_\sigma\) has the noise penalty baked in
- The noise distribution's support covers the **entire action space**, so Q-values get updated even in OOD regions

This is fundamentally different from conservative methods like CQL or TD3+BC. Those methods **avoid** OOD actions. PANI **enters OOD regions directly and corrects Q-values there**.

## Performance: The Numbers Speak

The D4RL benchmark results are, frankly, **embarrassing** for the diffusion camp.

### Gym-MuJoCo (medium dataset average)

| Algorithm | Avg Score | Notes |
|-----------|----------|-------|
| IQL (baseline) | 77.0 | — |
| **IQL-AN (with PANI)** | **87.2** | **+10.2** |
| TD3+BC (baseline) | 72.1 | — |
| **TD3-AN (with PANI)** | **90.2** | **+18.1** |
| Diffusion-QL | 88.0 | Diffusion-based |
| QGPO | 86.6 | Diffusion-based |
| SRPO | 87.1 | Diffusion-based |
| DTQL | 88.7 | Diffusion-based |

**TD3 with PANI (TD3-AN) beats every single diffusion-based method.** IQL-AN matches or exceeds most diffusion approaches.

### AntMaze

| Algorithm | Avg Score |
|-----------|----------|
| IQL | 58.4 |
| TD3+BC | 28.8 |
| **IQL-AN** | **68.5 (+10.1)** |
| **TD3-AN** | **77.7 (+48.9)** |
| QGPO | 78.3 |

The jump from TD3+BC to TD3-AN is **+48.9 points**. That's almost unbelievable. PANI single-handedly resurrects a completely failing algorithm to SOTA-level performance.

## Compute Cost: This Is the Real Killer

| Algorithm | Training Time (10⁶ steps) | Overhead |
|-----------|--------------------------|----------|
| IQL | 9m 33s | — |
| IQL-AN | 10m 22s | **+9%** |
| TD3 | 15m 45s | — |
| TD3-AN | 16m 14s | **+3%** |

**3-9% training overhead for 10-50 point performance gains.** Compared to diffusion methods that require multiple times the training compute and multi-step denoising at inference, this is practically free.

The **inference cost** matters even more. Diffusion policies need multiple denoising steps at every inference call. PANI uses the same lightweight MLP policy as the original algorithm. For real-time systems or resource-constrained environments, this difference is decisive.

## Hybrid Noise Distribution: The One Design Choice

The only non-trivial design decision in PANI is the **Hybrid Noise Distribution**.

The problem with plain Gaussian noise:
- **Too little noise**: insufficient coverage, OOD problem unsolved
- **Too much noise**: mode collapse, sample inefficiency

The solution combines two techniques:

1. **Uniform mixture**: \(q_t^A(a' \mid a) = \alpha(t) \cdot U(a' \mid A) + (1-\alpha(t)) \cdot q_t(a' \mid a)\)
2. **Exponential scaling**: applies exponential scaling to Gaussian noise, inducing **leptokurtic** behavior

The result: **robust performance across a wide range of noise scales**, while Gaussian and Laplace distributions suffer catastrophic drops at certain scales.

## OOD Overestimation Suppression: Hard Evidence

Probability of OOD overestimation \(P(Q(s, a') > Q(s, a))\) (where \(a\) is a dataset action, \(a'\) is uniformly sampled):

| Dataset | TD3+BC | TD3-AN | IQL | IQL-AN |
|---------|--------|--------|-----|--------|
| halfcheetah-expert | 0.014 | **0.002** | 0.014 | **0.001** |
| halfcheetah-medium | 0.112 | **0.004** | 0.211 | **0.006** |
| hopper-expert | 0.483 | **0.034** | 0.425 | **0.036** |
| walker2d-medium | 0.316 | **0.002** | 0.147 | **0.004** |

OOD overestimation probability drops by **10-100x**. This isn't just "better performance" — it's hard evidence that PANI **directly solves the root cause** of the problem.

## What Peer Reviewers Acknowledged

PANI was submitted to ICLR 2026 and later withdrawn by the authors. But looking at the reviews themselves, **even the critical reviewers couldn't deny PANI's core strengths**. The highest-scoring reviewer (Rating 6/10) wrote:

> *"It is appreciated that the authors consider the OOD issue from **new perspectives**. PANI is compatible with other methods and **this merit is quite appealing**."*
>
> *"The authors formalize the framework of PANI with Noise Action MDP, as well as proposing the concept of hybrid noise distribution, which **builds the theoretical foundation for the entire methodology**."*

Strengths that reviewers unanimously acknowledged:

- **"Clear intuition and methodology"** — the paper explains both the what and the why cleanly (Reviewer nUxv)
- **"Lightweight drop-in modification"** — requires only minimal changes to the standard Q-update step (Reviewer mqZV)
- **"Adjusting the penalty according to the distance intuitively promotes smoother Q-value surfaces"** — reducing sharp value changes around the data boundary (Reviewer nUxv)
- **"Experiments on numerous datasets and tasks"** — D4RL locomotion, Adroit, AntMaze, and OGBench (Reviewer SH7n)

What's telling is that the withdrawal wasn't because the method doesn't work. The authors' withdrawal comment states:

> *"We sincerely appreciate the reviewers' thorough and insightful feedback. We fully acknowledge that substantial revisions are necessary, and we are committed to addressing all comments with great care."*

The main criticisms targeted **paper completeness, not the core idea**: missing comparisons with similar approaches (RORL, SAC-RND, EPQ), lack of penalty scale sensitivity analysis, unavailable code, and hyperparameter tuning fairness concerns. **The novelty and practicality of the core idea went unchallenged even by the harshest reviewers.**

## Why You Should Care

**1. Research implication: Diffusion may be over-engineered**

PANI suggests that diffusion models succeed in offline RL not because they're diffusion, but because of **action space coverage through noise injection**. This has profound implications for future research directions.

**2. Practical implication: Plug and play**

Add a **few lines** to your existing IQL or TD3 code. No new architecture, no new training pipeline, no new inference procedure. Inject noise in the Q-update step, apply the penalty, done. As one reviewer put it, it's **"a lightweight drop-in modification requiring only minimal changes to the standard Q-update step."**

**3. Universality: It works everywhere**

PANI works with TD3, IQL, and even the diffusion-based QGPO — improving all of them. As reviewers acknowledged, **"this compatibility merit is quite appealing"** — PANI is not an algorithm-specific trick but a **general principle**.

## Limitations and Open Questions

Let's be honest — PANI has limitations. Including valid criticisms raised by reviewers:

- **Optimal noise scale selection** is dataset-dependent. The Hybrid distribution mitigates but doesn't fully solve this
- Whether noise injection alone suffices when **dataset action coverage is extremely sparse** remains an open question. As one reviewer noted, *"a more detailed discussion on how PANI behaves with highly concentrated expert datasets would strengthen the paper"*
- **The penalty term \(\|a - a'\|_2^2\) scale issue** — different tasks have different reward and action scales, potentially making the penalty ineffective or overwhelming. A scaling hyperparameter \(\alpha\) may be needed
- **Scaling to high-dimensional action spaces** and computational efficiency need further validation
- **Systematic comparison with similar pessimistic/anti-exploration approaches** (RORL, SAC-RND, EPQ) is lacking

But even accounting for these limitations, nothing currently beats PANI's **performance-per-complexity ratio**. The paper's completeness has room for improvement, but **the core idea's value shines regardless of review scores.**

## Conclusion: Simplicity Wins

PANI is the **most underrated paper** in Offline RL in recent years. Overshadowed by the glamour of diffusion models, it cuts through to the essence and achieves **equal or superior performance with minimal modifications**.

If you're a researcher, study the NAMDP theoretical framework. If you're a practitioner, add PANI to your existing code today. **Three lines of code can replace thousands of lines of diffusion implementation.**

> **"The best ideas are embarrassingly simple — the hard part is realizing they work."**

**Paper**: Oh, J. & Lee, B.-J. (2025). *Offline Reinforcement Learning with Penalized Action Noise Injection*. arXiv:2507.02356. Korea University & Gauss Labs.

</div>
