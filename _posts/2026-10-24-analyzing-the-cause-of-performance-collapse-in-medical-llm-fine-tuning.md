---
title: "[KoSAIM] Analyzing the Cause of Performance Collapse in Medical LLM Fine-Tuning — 포스터 심층 분석"
date: 2026-10-24 10:07:35 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Fine-Tuning","Data Selection","Curriculum Learning","Hard Example Mining","QLoRA","MedMCQA","McNemar Test","KoSAIM 2026"]
description: "의료 LLM 미세조정에서 '어려운 문제'만 골라 추가 학습하면 정확도가 61.42%에서 13.70%로, 4지선다 우연 수준(25%) 아래까지 무너집니다. 이 연구는 붕괴의 원인이 난이도 자체가 아니라, 모델 예측 기반 난이도가 사실상 '모델이 틀린 문제만' 고르게 만드는 오류 조건부 선택(error-conditioned selection)에 있음을 통제 실험으로 분리했습니다. 붕괴는 무작위 손상이 아니라 '자기 예측을 피하도록 학습한 결과(avoidance)'였고, 같은 데이터라도 난이도 순서로 제시하면 28.7%p가 더 떨어졌습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-medical-llm-collapse-kosaim-2026-full.jpg
  alt: "[KoSAIM] Analyzing the Cause of Performance Collapse in Medical LLM Fine-Tuning — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Analyzing the Cause of Performance Collapse in Medical LLM Fine-Tuning: Distinguishing Difficulty from Error-Conditioned Data Selection
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: **Seri Oh** — 분당서울대학교병원 의료인공지능센터 (Center for Artificial Intelligence in Healthcare, Seoul National University Bundang Hospital)
- **연구 지원**: 보건복지부 환자중심 의료기술 최적화 연구사업단(PACEN) (RS-2025-02219103)
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회 2026 정기학술대회)
- **핵심 키워드**: `Fine-Tuning`, `Data Selection`, `Curriculum Learning`, `Hard Example Mining`, `QLoRA`, `MedMCQA`, `McNemar Test`

![Analyzing the Cause of Performance Collapse in Medical LLM Fine-Tuning 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-medical-llm-collapse-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Analyzing the Cause of Performance Collapse in Medical LLM Fine-Tuning*

> **Executive Summary**  
> 의료 LLM을 미세조정할 때 "어려운 문제"만 골라 추가 학습하면 정확도가 **61.42% → 13.70%**로, 4지선다 우연 수준(25%)보다도 낮게 무너집니다. 이 연구는 원인이 **난이도 자체가 아니라 오류 조건부 선택(error-conditioned selection)**임을 통제 실험으로 분리했습니다. 모델 예측으로 계산한 난이도로 상위 문항을 고르면 **사실상 모델이 틀린 문제만** 선택됩니다. 모델이 이미 맞힌 어려운 문제를 추가하면 정확도는 그대로였고(61.42%), 난이도와 무관한 무작위 문항의 정답을 **모델 예측과 다르게 바꾸기만 해도** 붕괴(8.61%)가 재현되었습니다. 붕괴는 무작위 손상이 아니라 **자기 예측을 피하도록 학습한 결과(avoidance)**였고, 같은 데이터도 **난이도 순서로 제시하면 28.7%p** 더 떨어졌습니다.

---

## 1. 연구 배경 및 문제 정의 (Background & Problem Gap)

1. **데이터 선택이 중요한 이유** — 의료 LLM은 도메인 데이터로 미세조정해 적응시키는데 [1], 데이터와 연산 자원이 제한되므로 **어떤 예제를 학습에 넣을지**가 중요합니다.
2. **난이도 기반 선택의 유행과 의문** — Hard-example selection과 curriculum learning [2]은 난이도를 기준으로 예제를 고르거나 순서를 정합니다. 하지만 난이도만으로 충분한지에는 의문이 제기되어 왔습니다 [3].
3. **모델 기반 난이도의 함정** — 난이도를 모델 자신의 예측으로 $\text{difficulty} = 1 - P(\text{correct answer} \mid \text{question, options})$라고 정의하면, 이 값은 문제의 본질적 난이도가 아니라 **모델의 불확실성**을 반영합니다 [4]. 그 결과 "어려운 예제"는 **모델이 틀리는 예제와 겹치게** 됩니다.
4. **연구 질문** — 미세조정 중 성능 붕괴는 **난이도 자체** 때문인가, 아니면 **오류 조건부 선택** 때문인가?

---

## 2. 연구 방법 (Methods)

### 2-1. Baseline과 데이터 분할

- **모델**: Qwen2.5-7B-Instruct (4-bit NF4), **QLoRA** [5] (r = 16, α = 32)
- **Baseline**: MedMCQA [6] 20,000문항(**INIT**)으로 미세조정
- **POOL**: 겹치지 않는 50,000문항. Baseline이 여기서 각 문항의 난이도를 계산합니다.
- **TEST**: MedMCQA 공식 validation split (**n = 4,183**)
- 세 분할은 서로 ID가 겹치지 않습니다 (id-disjoint).

### 2-2. 왜 상위 N 난이도는 오답만 고르는가 (Proposition 1)

- $k$지선다 문항에서 정답을 맞히려면 $P(\text{gold}) \ge 1/k$여야 합니다.
- 따라서 $\text{difficulty} > 1 - 1/k$이면 모델은 **반드시 틀린 상태**입니다. $k = 4$이면 임계값은 **0.75**입니다.
- POOL에서 9,647문항이 0.75를 넘었고, 이 중 맞힌 문항은 하나도 없었습니다. 8,000 < 9,647이므로, **상위 8,000개 Hard 세트는 연역적으로 전부 오답 문항**입니다 (alignment $A = 0$).

### Figure 1. 난이도 구간별 Baseline 정확도 (POOL, n = 50,000)

![Figure 1: Baseline accuracy by difficulty band in POOL](/assets/img/posts/2026-10-24-kosaim-2026/poster-medical-llm-collapse-kosaim-2026-figure-1.png)
*Figure 1: 난이도 구간별 Baseline 정확도. 구간별 문항 수는 20,250 / 6,921 / 3,283 / 7,663 / 6,646 / 5,237이며, 0.50 미만에서는 모든 문항을 맞히고 0.70 초과에서는 거의 모두 틀립니다. 어떤 구간도 POOL의 자연 정답률 0.702를 재현하지 않습니다.*

- 난이도 구간별 정확도는 **1.000, 1.000, 1.000, 0.594, 0.016, 0.000**으로 급격히 갈립니다.
- 즉, 모델 기반 난이도로 구간을 나누면 각 구간은 사실상 "맞힌 문제 묶음" 또는 "틀린 문제 묶음"이 됩니다.

### 2-3. 학습 조건 (Training Arms & Controls)

모든 조건은 Baseline에 **정확히 8,000개 POOL 문항**을 추가하며, 학습률·배치·스케줄·스텝 수가 동일합니다. **선택 규칙만** 다릅니다.

| 구분 | 조건 | 무엇을 분리하는가 |
| :--- | :--- | :--- |
| Main arms | **Random** | 기준 비교 |
| | **Hard** (최고 난이도) | 난이도 + 오류가 섞인 기존 방식 |
| | **Curriculum** (중간 → 매우 어려움, 순서대로, 하나의 optimizer) | 난이도 + 오류 + 순서 |
| Controls | **Baseline이 맞힌 고난이도 문항** | 난이도만 (오류 없음) |
| | **무작위 문항을 모델 예측과 다른 답으로 재라벨** | 오류만 (난이도 선택 없음) |
| | 무작위 문항을 균등 무작위로 재라벨 | 방향 없는 라벨 잡음 |
| | 무작위 문항을 모델 예측으로 재라벨 | 모델과 일치하는 라벨 |
| | **Capped curriculum** (난이도 상한 0.95, 쉬운 문항 25% 고정), 난이도 순서 vs 셔플 | 순서만 |

### 2-4. 평가 지표

- **학습 손실**: 답 문자 토큰 하나에 대한 loss
- **정확도**: TEST에서 Wilson 95% CI, Baseline 대비 **paired McNemar exact test**
- **Alignment** $A = P(\text{gold} = \text{model's argmax})$: 추가된 세트에서 정답이 모델의 최상위 예측과 일치하는 비율 (POOL의 자연값 0.702)
- **Avoidance index**: $AI = \dfrac{1/k - R}{1/k}$, 여기서 $R$은 TEST에서 Baseline의 예측을 유지한 비율
  - $AI = 1$이면 완전한 회피, $AI = 0$이면 무작위 표류(random drift)

---

## 3. 실험 결과 (Results)

### 3-1. 모델 성능

- 초기 미세조정으로 정확도가 **55.10%(zero-shot) → 61.42%**로 올랐습니다. 무작위 8,000문항을 더하면 62.04%였습니다 (p = 0.19).
- **Hard 선택은 13.70%, Curriculum은 10.33%**로 떨어졌습니다. 4지선다 우연 수준 25%보다 낮습니다.

### Table 1. 8,000문항 추가 후 TEST 정확도 (n = 4,183)

| 조건 | Accuracy (%) | Δ vs. Baseline (pp) |
| :--- | ---: | ---: |
| Baseline — initial fine-tuning only | 61.42 | — |
| **+8,000 items — Main arms** | | |
| Random | 62.04 | +0.62 (n.s.) |
| Hard — highest difficulty | **13.70** | −47.72 |
| Curriculum — moderate → very hard, in order | **10.33** | −51.09 |
| **+8,000 items — Controls** | | |
| High-difficulty items correctly answered by Baseline | 61.42 | ±0.00 (n.s.) |
| Random items relabeled away from the model's prediction | **8.61** | −52.81 |
| Random items relabeled uniformly at random | 30.22 | −31.20 |
| Capped curriculum, in difficulty order | 33.01 | −28.40 |
| ↳ Same 8,000 items, shuffled | 61.75 | +0.33 (n.s.) |

*Δ는 %p. n.s. = 유의하지 않음 (paired McNemar, p ≥ 0.19). 나머지 Δ는 모두 p < 10⁻¹⁵⁰. 굵은 값은 우연 수준 25% 미만. 추가 대조군 두 개도 Baseline 근처였습니다: capped + anchored Hard 61.61%, 모델 예측으로 재라벨한 무작위 문항 60.39%.*

### 3-2. 난이도·오류·순서의 분리

- **난이도만으로는 붕괴하지 않음**: Baseline이 맞힌 고난이도 문항을 추가하면 정확도가 그대로였습니다 (61.42%, p = 1.00).
- **오류만으로 붕괴 재현**: 난이도 선택 없이 무작위 문항의 정답을 모델 예측과 다르게 바꾸기만 해도 붕괴가 재현되었습니다 (**8.61%**). 반면 균등 무작위 라벨은 우연 수준 위에 머물렀습니다 (30.22%).
- **순서도 중요**: 같은 8,000문항, 같은 optimizer, 같은 스케줄인데 난이도 순서로 제시하면 33.01%, 섞으면 61.75%였습니다. **순서만으로 28.7%p** 차이가 났습니다.

### Figure 2. 추가 세트의 평균 난이도 vs alignment에 따른 정확도

![Figure 2: Accuracy by difficulty vs. by alignment of the added set](/assets/img/posts/2026-10-24-kosaim-2026/poster-medical-llm-collapse-kosaim-2026-figure-2.png)
*Figure 2: (a) 추가 세트의 평균 난이도 기준, (b) alignment A 기준. 원은 섞거나 셔플한 8개 조건, 삼각형은 난이도 순서로 제시한 2개 조건입니다.*

- **(a) 난이도는 정확도를 설명하지 못합니다.** 평균 난이도가 0.388로 똑같은 4개 조건(같은 문항, 라벨만 다름)의 정확도가 **8.6%에서 62.0%**까지 흩어집니다.
- **(b) alignment A가 정확도를 훨씬 잘 설명합니다.** A가 낮을수록(정답이 모델 예측과 어긋날수록) 정확도가 떨어집니다. 같은 8,000문항에서 **순서만 바꿔도 +28.7%p** 차이가 납니다.

### 3-3. 붕괴는 손상이 아니라 회피 (Collapse Is Avoidance, Not Damage)

- **두 가설의 예측값 비교**
  - 방향 없는 손상(undirected damage)이라면 정확도는 **25%** 쪽으로 표류해야 합니다.
  - 모델이 **자기 최상위 선택을 피하도록** 학습했다면, 정확도는 $(1 - 0.6142)/3 = 12.86\%$가 됩니다. 원래 틀린 38.58% 문항의 정답이 남은 3개 선택지 중 하나로 고르게 맞는다는 계산입니다.
- **관측값**: Hard arm은 **13.70%**로 회피 가설의 예측(12.86%)과 거의 일치했습니다. Baseline 예측을 유지한 문항은 1.05%에 불과했습니다 ($AI = 0.96$).
- **Curriculum이 더 나쁜 이유**: 커리큘럼은 오류가 가장 많은 단계(난이도 0.714, $A = 0.250$)로 끝납니다. 그래서 회피 패턴이 **마지막에 각인되고 이를 바로잡을 데이터가 남지 않습니다.**
- **배제한 대안 설명**
  - 발산(divergence): loss가 1.59 → 1.01로 매끄럽게 감소했습니다.
  - 출력 붕괴(output collapse): 예측이 A–D에 고르게 분포했습니다.
  - 데이터 결합 오류
  - 학습률 예산: optimizer 하나로 재실행해도 11.07% → 10.33%였습니다.
  - 정답 위치 편향: 정답 문자 분포가 21–27%로 고르게 퍼져 있었습니다.

---

## 4. 결론 및 의의 (Conclusion)

1. **난이도만으로는 붕괴가 일어나지 않습니다.** 모델이 이미 맞힌 고난이도 문항은 정확도를 바꾸지 않았습니다 (61.42%, p = 1.00).
2. **학습 예제가 모델의 예측과 모순될 때 심각한 성능 저하가 일어납니다.** 이는 난이도 선택 없이도 재현됩니다 (8.61%). 반면 균등 무작위 라벨은 우연 수준 위에 머물렀습니다 (30.22%).
3. **같은 예제의 제시 순서도 중요합니다** (난이도 순서 33.01% vs 셔플 61.75%).
4. **실무 권고** (모델·데이터셋·시드 각 1개라는 한계 하에서):
   - 난이도 기반 선택은 선택된 예제의 **정답 분포와 순서를 함께** 평가할 것
   - POOL의 **자연 정답률을 유지**할 것
   - **난이도 순서로 제시하지 말 것**
   - 난이도 상한(cap)은 **필요조건이지 충분조건이 아님**

### Take-Home Message

> **"Difficulty alone was not sufficient to cause collapse. Severe degradation occurred when training examples contradicted the model's predictions."**
>
> (붕괴의 원인은 어려운 문제가 아니라, **모델의 예측과 정면으로 어긋나는 예제만 모아서 학습시킨 것**입니다. 모델 기반 난이도로 "어려운 문제"를 고르면 이것이 자동으로 일어납니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. 이 포스터를 쉽게 설명하면?

학생에게 "네가 틀린 문제만 모아서 오답 노트를 만들어 줄게"라고 했는데, 그 노트를 공부한 학생이 **오히려 "내가 고르고 싶은 답은 틀린 답"이라고 배운** 상황입니다.

- 모델 기반 난이도로 "어려운 문제"를 고르면 사실상 **모델이 틀린 문제만** 골라집니다.
- 그 문제들만 학습하면 모델은 "정답을 찾는 법"이 아니라 **"내 첫 직감을 피하는 법"**을 배웁니다.
- 그 결과 원래 맞히던 문제까지 피해서 틀리게 되고, 정확도가 우연 수준 아래로 떨어집니다.

#### Q. 왜 이 연구가 통계적으로 모범적인가요?

이 포스터는 "왜 망가졌는가"를 **가설을 세우고 반증 가능하게 검증**하는 구조입니다.

1. **경쟁 가설에 대한 정량적 예측** — "손상이면 25%, 회피면 12.86%"처럼 각 가설이 예측하는 수치를 **실험 전에 계산**할 수 있게 만들고, 관측값(13.70%)과 비교했습니다.
2. **교란 요인 분리** — 난이도, 오류(라벨-예측 불일치), 순서라는 세 요인을 각각 하나씩만 바꾸는 대조군을 설계했습니다. 특히 "평균 난이도가 같은 4개 조건"은 난이도를 고정하고 라벨만 바꾼 깔끔한 비교입니다.
3. **적절한 검정** — 같은 TEST 문항에 대한 두 모델 비교이므로 독립 표본 검정이 아닌 **paired McNemar exact test**를, 비율 신뢰구간에는 **Wilson CI**를 썼습니다.
4. **대안 설명의 체계적 배제** — 발산, 출력 붕괴, 학습률, 정답 위치 편향을 하나씩 확인했습니다.

#### Q. 비판적으로 본다면?

1. **MedMCQA의 라벨 잡음 가능성** — 괜찮은 모델이 **확신을 갖고 틀리는** 문항에는 원래 **정답 라벨이 잘못된 문항**이 상대적으로 많이 섞여 있을 수 있습니다. 그렇다면 Hard 세트는 "어렵지만 맞는 문제"가 아니라 "라벨 오류가 농축된 세트"일 수 있습니다. 이는 이 연구의 결론(모델 예측과 어긋나는 라벨이 문제)과 같은 방향이지만, 해석에 한 층을 더합니다.
2. **극단적 설정** — Proposition 1 때문에 Hard 세트는 100% 오답 문항입니다. 실무에서 흔한 "상위 일부 + 무작위 일부" 혼합 설정에서 붕괴 정도가 어떻게 연속적으로 변하는지(용량-반응 곡선)가 보이면 더 유용했을 것입니다.
3. **"Relabel away"가 회피 예측보다 낮음** — 무작위 문항을 모델 예측과 다르게 재라벨한 조건은 8.61%로, 회피 가설 예측(12.86%)보다도 낮습니다. 회피 외에 추가 기전이 있는지 설명이 필요합니다.
4. **일반화 범위** — 저자도 밝혔듯 모델 1개, 데이터셋 1개, 시드 1개입니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- 흔히 쓰는 기법(hard example mining, curriculum)이 **왜 실패하는지**를 정면으로 다룬 **부정적 결과(negative result)의 원인 분석**입니다.
- 단독 저자 연구임에도 대조군 설계, 검정, 대안 설명 배제가 매우 촘촘합니다.
- 결론이 바로 쓸 수 있는 **실무 권고**(자연 정답률 유지, 난이도 순서 제시 금지)로 이어집니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 지금까지 본 포스터 중 가장 "통계적인" 포스터입니다. 새 모델이나 새 기법이 아니라, **"왜 망가졌는가"를 가설-예측-반증 구조로 끝까지 추적**합니다. "어려운 문제를 더 학습시키면 좋아질 것"이라는 직관이 모델 기반 난이도와 만나면 **오류 조건부 선택**이라는 함정으로 바뀐다는 점은, 데이터 큐레이션을 하는 모든 의료 AI 연구자가 알아야 할 내용입니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 연구실 데이터로 미세조정할 때의 데이터 큐레이션 체크리스트** ⭐ 우선순위 1

- 신경과 판독문, 경과 기록 등으로 모델을 미세조정하거나 능동 학습(active learning)으로 라벨링 대상을 고를 때, 모델 불확실성 기반 선택은 이 포스터의 함정에 그대로 빠질 수 있습니다.
- 학습 세트를 확정하기 전에 다음을 기본 진단으로 보고합니다.
  - 선택된 세트의 **alignment A**(정답과 모델 예측의 일치율)가 전체 풀의 자연값과 얼마나 다른가
  - 제시 순서가 난이도와 상관되어 있는가 (셔플 여부)
- 통계/AI 연구원이 연구실 표준 절차로 만들기 좋은 항목입니다.

**B. 평가자 간 불일치가 큰 신경과 라벨에서의 재현 연구**

- 뇌졸중 아형(TOAST), 발작 분류, 뇌파 판독처럼 **전문가 간 불일치가 큰** 과제에서는 "모델과 어긋나는 라벨"이 실제로는 **라벨 잡음**일 가능성이 큽니다.
- 다수 평가자 라벨이 있는 데이터에서 "평가자 간 불일치 문항"과 "모델 고난이도 문항"의 겹침을 분석하면, 이 포스터의 비판점 1(라벨 잡음)을 직접 검증하는 연구가 됩니다.

**C. 용량-반응(dose-response) 실험으로 확장**

- Hard 문항 비율을 0%, 10%, …, 100%로 바꿔가며 정확도와 avoidance index의 변화를 곡선으로 추정합니다.
- 실무에서 "어느 정도까지 섞어도 안전한가"라는 질문에 답할 수 있고, 통계 모형(예: 로지스틱 또는 분할 회귀)으로 임계점을 추정하는 방법론적 기여가 가능합니다.

> **추천**: **A를 바로 적용**하고, 연구실에 다수 평가자 라벨 데이터가 있다면 **B**를 연구 과제로 추진합니다. A는 비용이 거의 들지 않으면서 앞으로의 모든 미세조정 프로젝트를 보호하고, B는 이 포스터의 열린 질문에 신경과 데이터로 답하는 독창적 기여가 됩니다.

---

### References (포스터 인용)

1. Wu C, Lin W, Zhang X, Zhang Y, Xie W, Wang Y. PMC-LLaMA: toward building open-source language models for medicine. *J Am Med Inform Assoc* 2024;31(9):1833–1843.
2. Bengio Y, Louradour J, Collobert R, Weston J. Curriculum learning. In: *Proceedings of the 26th ICML*. 2009:41–48.
3. Jiang Z, Han J, Li T, Wang X, Jiang S, Meng X, et al. Difficulty is not enough: curriculum learning for LLMs fine-tuning must consider utility. *Proc AAAI Conf Artif Intell* 2026;40(37):31365–31373.
4. Swayamdipta S, Schwartz R, Lourie N, Wang Y, Hajishirzi H, Smith NA, et al. Dataset cartography: mapping and diagnosing datasets with training dynamics. In: *Proceedings of EMNLP*. 2020:9275–9293.
5. Dettmers T, Pagnoni A, Holtzman A, Zettlemoyer L. QLoRA: efficient finetuning of quantized LLMs. *Adv Neural Inf Process Syst* 2023;36:10088–10115.
6. Pal A, Umapathi LK, Sankarasubbu M. MedMCQA: a large-scale multi-subject multi-choice dataset for medical domain question answering. In: *Proceedings of CHIL*. PMLR 2022;174:248–260.
