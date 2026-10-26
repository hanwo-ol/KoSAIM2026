---
title: "[KoSAIM] Fine-Tuned Small Language Models for Hierarchical KTAS Triage and Clinician–Model Collaboration — 포스터 심층 분석"
date: 2026-10-24 10:06:00 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["KTAS","Emergency Triage","Small Language Model (SLM)","LoRA","Hierarchical Classification","Clinician-AI Collaboration","Automation Bias","KoSAIM 2026"]
description: "응급실 중증도 분류(KTAS)를 Major Category → Minor Category → KTAS Level의 계층 경로로 예측하도록 오픈 웨이트 소형 언어모델(SLM)을 단계별 LoRA로 미세조정하고, 4개 숙련도 그룹의 임상의를 대상으로 무작위 교차(crossover) 평가를 수행한 연구입니다. 미세조정 후 Hari-Q3-8B의 KTAS Top-1 정확도는 0.168 → 0.682로 올랐고, SLM 보조 시 모든 그룹에서 정확도가 오르고 under-triage가 줄었지만 over-triage는 늘었습니다. KTAS 미인증 간호사 그룹에서 이득과 자동화 편향 취약성이 모두 가장 컸습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-ktas-triage-slm-kosaim-2026-full.jpg
  alt: "[KoSAIM] Fine-Tuned Small Language Models for Hierarchical KTAS Triage — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Fine-Tuned Small Language Models for Hierarchical KTAS Triage and Clinician–Model Collaboration: Development and Crossover Evaluation Study
- **포스터 번호**: P5-16 (Session 5 · 의료 LLM) — **Poster Award (Certificate of Merit)** 수상
- **저자 및 소속**: **Doyeon Kim**¹, Jeong Hyup Lim¹, Beom Sok Seo², Youn… ³, Eu… ², **Joon Bum Park**²†, **Kyu-Hwan Jung**¹·⁴† (†: Corresponding Authors)
  - ¹Sungkyunkwan University, ²Soonchunhyang University Hospital, ³(Soonchunhyang University 추정), ⁴Samsung Medical Center
  - <!-- TODO: 리본에 가려진 저자 2명(Youn…, Eu…)과 ³ 소속 확인 필요 -->
- **연구 지원**: 보건복지부 · 한국보건산업진흥원(KHIDI) 한국보건의료기술 R&D 과제 (RS-2024-00335937)
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `KTAS`, `Emergency Triage`, `Small Language Model (SLM)`, `LoRA`, `Hierarchical Classification`, `Clinician-AI Collaboration`, `Automation Bias`

![Fine-Tuned SLMs for Hierarchical KTAS Triage 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-ktas-triage-slm-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Fine-Tuned Small Language Models for Hierarchical KTAS Triage and Clinician–Model Collaboration*

> **Executive Summary**  
> 응급실 중증도 분류(KTAS)를 Major Category → Minor Category → KTAS Level의 계층 경로로 예측하도록 오픈 웨이트 SLM(4–8B)을 **단계별 LoRA 어댑터**로 미세조정했습니다. 최고 모델 Hari-Q3-8B는 KTAS Top-1 정확도 **0.168 → 0.682**, QWK **0.134 → 0.857**로 향상되었습니다. 임상의 8명(4개 숙련도 그룹)의 무작위 교차 평가에서 SLM 보조는 **모든 그룹의 정확도를 높이고 under-triage를 줄였지만 over-triage는 늘렸고**, KTAS 미인증 간호사 그룹(Group I)에서 이득과 함께 **잘못된 SLM 출력에 끌려가는 취약성(automation bias)**도 가장 크게 나타났습니다.

---

## 1. 연구 배경 및 임상적 간극 (Background & Problem Gap)

### 1-1. 응급실 중증도 분류 오류의 비대칭성

- **Under-triage(과소 분류)**는 시간 민감한 처치를 지연시키고, **Over-triage(과대 분류)**는 불필요한 자원 사용을 늘립니다.
- 5단계 **한국형 응급환자 분류도구(KTAS)** 기준으로, 두 기관 검토에서 **14.7%**의 분류가 전문가 판단과 달랐고 대부분이 under-triage였습니다 [1].

### 1-2. 기존 접근의 한계

1. **계층 구조 무시** — KTAS는 *Major Category(주호소 범주) → Minor Category(구체적 증상) → KTAS Level* 의 계층으로 중증도를 부여하지만 [2], 기존 모델 대부분은 **최종 Level만** 예측했습니다.
2. **협업 효과 미검증** — 모델 출력이 맞을 때와 틀릴 때 임상의가 어떻게 반응하는지가 불분명합니다.
3. **온프레미스 요구** — 오픈 웨이트 SLM은 환자 데이터를 외부로 보내지 않고 병원 내부에서 구동할 수 있지만, 분류 과제에서의 성능과 안전성은 평가가 필요합니다.

### 1-3. 연구 목적 및 기여 (Objective & Contributions)

- **Objective**: 계층형 KTAS 예측을 위한 SLM을 미세조정하고, SLM 보조가 임상의의 분류 결정(under-/over-triage 포함)을 어떻게 바꾸는지 검증
- **Hierarchical KTAS prediction**: Major → Minor → KTAS Level 전체 경로에 대한 **단계별(stage-specific) 미세조정**
- **Clinician–model collaboration**: 4개 숙련도 수준에 걸친 **무작위 교차(randomized crossover) 평가**, 오류 방향 및 짝지은(paired) 결정 전이 분석

---

## 2. 제안 방법론 및 핵심 아키텍처 (Proposed Methodology)

### 2-1. 임상의가 만든 합성 데이터셋 (Clinician-curated synthetic datasets)

| 데이터셋 | 규모 | 생성 방식 | 용도 |
| :--- | :--- | :--- | :--- |
| Patient-state descriptions | **441건** | 임상의 작성 + 동료 검토 | **Fine-tuning** |
| Multi-turn triage conversations | **727건** (평균 9.15 turn) | GPT-4o, GPT-5.1을 모의 정보 제공자(simulated informant)로 사용, 1,216건 중 임상의 검토 후 선별 | **모델 선택 및 평가** |

- 두 데이터셋은 **서로 독립적이고 겹치지 않으며**, 하나의 라벨 계층(**17 Major / 174 Minor / 5 KTAS Level**)과 임상의 검증 정답을 공유합니다.
- Patient-state는 *Age · ED Presentation · Relevant Clinical Status* 세 요소로 구성됩니다.

### 2-2. 계층형 KTAS 프레임워크 (Hierarchical KTAS framework)

- **모델**: 4–8B 파라미터 오픈 웨이트 SLM 4종
  - MedGemma-4B (instruction-tuned, medical)
  - Llama-3.1-8B, Qwen3-8B (general-purpose)
  - Hari-Q3-8B (Korean medical) [5]
- **Stage-specific LoRA** [3]: 단계마다 어댑터 1개. 앞 단계의 예측 라벨을 다음 단계로 넘기고, Minor Category 후보는 예측된 Major Category 안으로 제한합니다.
- **Constrained candidate scoring**: 자유 생성 대신 후보 라벨 점수화로 **임의 라벨 생성을 방지**합니다.
- **비교군**: 단일 공유 어댑터(**Hari-Q3-Shared**)를 보조 비교로 사용합니다.

### 2-3. 무작위 교차 임상의 평가 (Randomized crossover clinician evaluation)

- **평가자**: 데이터셋 구축에 참여하지 않은 **임상의 8명** (그룹당 2명)

| 그룹 | 구성 |
| :--- | :--- |
| Group I | KTAS 인증이 없는 응급실 간호사 |
| Group II | KTAS provider 인증 응급실 간호사 |
| Group III | KTAS instructor 인증 응급실 간호사 |
| Group IV | 응급의학과 전문의 |

- **평가 케이스**: SLM 예측이 맞은 경우와 틀린 경우를 모두 포함한 **150건 부분집합** (KTAS Level당 30건)
- **조건**: Only-SLM(모델 단독) / Only-Human(임상의 단독) / **SLM+Human**(SLM이 예측한 라벨만 보여줌. confidence나 rationale은 제공하지 않음)
- **교차 설계**: Sequence A는 Only-Human → (3주 washout) → SLM+Human, Sequence B는 그 역순입니다.
- **평가 지표**: Accuracy, Quadratic Weighted Kappa(QWK) [4], under-/over-triage(정답보다 덜/더 긴급), SLM 정오에 따른 paired decision transition, Decision Time
- **통계**: bootstrap 95% 신뢰구간

---

## 3. 주요 Figure 및 비교 표 상세 분석 (Figures & Tables Breakdown)

### Figure 1. Hierarchical KTAS framework and evaluation conditions

> **요약**: 상단은 Patient-state 데이터로 3개 단계의 SLM을 순차 LoRA 미세조정하는 학습 흐름이고, 하단은 Patient-conversation을 입력으로 Stage 1–3 각각에서 SLM 단독 / 임상의 단독 / SLM+임상의 조건을 비교하는 평가 흐름입니다.

![Figure 1: Hierarchical KTAS framework and evaluation conditions](/assets/img/posts/2026-10-24-kosaim-2026/poster-ktas-triage-slm-kosaim-2026-figure-1.png)
*Figure 1: Hierarchical KTAS framework and evaluation conditions*

```
[Training]   Clinician ─▶ Patient-state (Age · ED Presentation · Relevant Clinical Status)
                              │
                              ▼
             LoRA: SLM (Major) ─▶ LoRA: SLM (Minor) ─▶ LoRA: SLM (KTAS Level)

[Evaluation] Patient-conversation (GPT-simulated, multi-turn)
                              │
                              ▼
   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
   │ Stage 1      │ ─▶ │ Stage 2      │ ─▶ │ Stage 3      │   ← Hierarchical KTAS Framework
   │ Major Cat.   │    │ Minor Cat.   │    │ KTAS Level   │
   │  SLM + Group │    │  SLM + Group │    │  SLM + Group │   ← Clinician Evaluation (I–IV)
   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘
          └───────────────────┼───────────────────┘
                              ▼
          Accuracy · Decision Time · KTAS Error-direction    ← Evaluation Metrics
```

---

### Table 1. 단독 SLM 성능 (727 conversations)

> **요약**: 모든 SLM에서 미세조정이 Major, Minor, KTAS Level 정확도를 크게 끌어올렸습니다. Base 모델의 Major Top-1은 0.05–0.31 수준에 그칩니다. 세 번의 추론 평균값이며, 굵은 값은 열별 최고입니다.

| Model | Condition | Major Top-1 | Major Top-3 | Minor Top-1 | Minor Top-3 | KTAS Top-1 | KTAS QWK |
| :--- | :--- | ---: | ---: | ---: | ---: | ---: | ---: |
| MedGemma-4B | Base | 0.062 | 0.319 | 0.012 | 0.084 | 0.190 | 0.134 |
| | Fine-tuned | 0.829 | 0.911 | 0.699 | 0.851 | 0.613 | 0.829 |
| Llama-3.1-8B | Base | 0.045 | 0.312 | 0.010 | 0.105 | 0.442 | 0.008 |
| | Fine-tuned | 0.801 | 0.894 | 0.688 | 0.818 | 0.477 | 0.593 |
| Qwen3-8B | Base | 0.307 | 0.477 | 0.041 | 0.135 | 0.325 | 0.295 |
| | Fine-tuned | **0.836** | 0.913 | 0.692 | 0.842 | 0.646 | 0.814 |
| Hari-Q3-8B | Base | 0.212 | 0.433 | 0.037 | 0.105 | 0.168 | 0.134 |
| | **Fine-tuned** | **0.836** | **0.927** | **0.751** | **0.897** | **0.682** | **0.857** |

- Llama-3.1-8B Base의 KTAS Top-1(0.442)은 높아 보이지만 QWK가 0.008에 불과해, 특정 Level에 쏠린 예측일 가능성이 큽니다. 순서형 라벨에서 **정확도만 보면 안 되는 이유**를 보여주는 사례입니다.

### Table 2. 계층 경로 정확도: Stage-specific vs Shared adapter (Hari-Q3-8B)

| Hierarchical pathway | Stage-specific (3 adapters) | Shared (1 adapter) | Difference (95% CI) Shared − Stage-specific |
| :--- | ---: | ---: | :--- |
| Major–Minor Joint Top-1 | **0.711** | 0.666 | −0.045 (−0.081 to −0.011) |
| Major–Minor–KTAS Exact-Match | **0.505** | 0.459 | −0.045 (−0.085 to −0.006) |

- Exact-match 정확도(0.001 → **0.505**)는 KTAS Top-1(0.682)보다 낮습니다. **Level은 맞혔지만 경로(카테고리)는 틀린 경우**가 상당하다는 뜻입니다.
- 단계별 어댑터가 단일 공유 어댑터보다 두 경로 지표 모두에서 유의하게 우수했습니다 (CI가 0을 포함하지 않음).

---

### Figure 2. Top-1 accuracy on the 150-case subset

> **요약**: 각 단계에서 Only-SLM(검정), Only-Human(파랑), SLM+Human(주황)의 Top-1 정확도를 비교합니다. 점선은 미세조정된 단계 어댑터(Hari-Q3-Major/Minor/KTAS)의 정확도입니다.

![Figure 2: Top-1 accuracy on the 150-case subset](/assets/img/posts/2026-10-24-kosaim-2026/poster-ktas-triage-slm-kosaim-2026-figure-2.png)
*Figure 2: Top-1 accuracy on the 150-case subset*

| 조건 | (A) Major Category | (B) Minor Category | (C) KTAS Level |
| :--- | ---: | ---: | ---: |
| Hari-Q3-8B (base) | 0.233 | 0.040 | 0.233 |
| **Only-SLM (fine-tuned adapter)** | **0.887** | **0.840** | **0.753** |
| Group I — Only-Human → SLM+Human | 0.837 → 0.873 | 0.817 → 0.900 | 0.507 → **0.680** |
| Group II — Only-Human → SLM+Human | 0.887 → 0.923 | 0.850 → 0.863 | 0.767 → 0.783 |
| Group III — Only-Human → SLM+Human | 0.920 → 0.940 | 0.900 → 0.920 | 0.833 → 0.843 |
| Group IV — Only-Human → SLM+Human | 0.920 → 0.930 | 0.870 → 0.907 | 0.817 → 0.853 |

- **SLM+Human > Only-Human**: 4개 그룹, 3개 단계 모두에서 성립합니다.
- **Groups II–IV**는 SLM 보조 시 모든 단계에서 **Only-SLM을 넘어섰습니다.** 반면 **Group I**은 Minor Category에서만 넘었고, Major(0.873 < 0.887)와 KTAS Level(0.680 < 0.753)에서는 SLM 단독보다 낮았습니다.
- **최대 이득은 Group I**: KTAS Level 0.507 → 0.680 (paired difference 0.173, 95% CI 0.100–0.243), exact-match 0.360 → 0.553.

---

### Figure 3. KTAS error direction by group and condition

> **요약**: 각 그룹과 조건별 KTAS 결정을 Under-triage(빨강), Correct(초록), Over-triage(파랑) 비율로 분해한 누적 막대그래프입니다.

![Figure 3: KTAS error direction by group and condition](/assets/img/posts/2026-10-24-kosaim-2026/poster-ktas-triage-slm-kosaim-2026-figure-3.png)
*Figure 3: KTAS error direction by group and condition*

| 그룹 | 조건 | Under-triage | Correct | Over-triage |
| :--- | :--- | ---: | ---: | ---: |
| Group I | Only-Human | 42.7% | 50.7% | 6.7% |
| | SLM+Human | **20.3%** | 68.0% | 11.7% |
| Group II | Only-Human | 14.0% | 76.7% | 9.3% |
| | SLM+Human | 11.3% | 78.3% | 10.3% |
| Group III | Only-Human | 6.3% | 83.3% | 10.3% |
| | SLM+Human | 5.0% | 84.3% | 10.7% |
| Group IV | Only-Human | 8.3% | 81.7% | 10.0% |
| | SLM+Human | 3.7% | 85.3% | 11.0% |

- **Under-triage 감소**: 모든 그룹에서 줄었고, Group I에서 가장 컸습니다 (42.7% → 20.3%, 차이의 95% CI −28.3 to −16.3).
- **Over-triage 증가**: 모든 그룹에서 늘었고, Group I에서 가장 컸습니다 (6.7% → 11.7%). 단, 모든 paired 95% CI가 0을 포함합니다.
- **Group I의 paired KTAS Level 전이** (Only-Human → SLM+Human)
  - **SLM이 맞았을 때**: 틀렸던 결정의 **69.3% (70/101)**가 정답으로 바뀌었습니다.
  - **SLM이 틀렸을 때**: 맞았던 결정의 **77.8% (21/27)**가 오답으로 바뀌었습니다. Groups II–IV에서는 이 비율이 16.7%–34.5%입니다.

---

## 4. 실험 결과 요약 (Results)

1. **미세조정 효과** — 4개 SLM 모두에서 카테고리, KTAS Level, 경로 정확도가 향상되었습니다. Hari-Q3-8B가 최고 성능을 보였습니다 (exact-match 0.001 → 0.505, KTAS Top-1 0.168 → 0.682, QWK 0.134 → 0.857).
2. **경로 정확도** — exact-match가 KTAS Top-1보다 낮았고, 단계별 어댑터가 공유 어댑터보다 우수했습니다.
3. **임상의 성능** — SLM 보조가 모든 그룹, 모든 단계에서 정확도를 높였고, 이득은 숙련도가 낮을수록 컸습니다.
4. **안전성 지표** — under-triage는 줄고 over-triage는 늘었으며, 비숙련 그룹은 잘못된 SLM 출력에 가장 크게 흔들렸습니다.

---

## 5. 기대 효과 및 임상적 의의 (Discussion & Clinical Significance)

### Discussion

- **Hierarchical KTAS prediction**: 441건의 간결한 서술만으로 미세조정한 SLM이 멀티턴 대화에서도 성능이 향상되었습니다. 경로 수준 평가는 최종 Level 정확도가 놓치는 "**틀린 카테고리를 거쳐 맞힌 KTAS Level**"을 드러냈습니다.
- **Expertise-dependent complementarity**: SLM 보조 시 Groups III, IV는 exact-match에서 SLM 단독을 넘었지만, Group I은 Major와 KTAS Level에서 SLM 단독에 못 미쳤습니다. **협업의 가치는 숙련도에 따라 달라질 수 있습니다.**
- **Collaboration beyond accuracy**: Group I은 under-triage가 가장 많이 줄었지만, 잘못된 SLM 출력 뒤에 correct → incorrect 전환도 가장 많았습니다. 보조 도구는 **독립적 판단을 보존하도록** 설계되어야 합니다.
- **Limitations**: 별도 hold-out test set이 없는 합성 데이터, 그룹당 평가자 2명, 라벨만 보여주는 인터페이스
- **Future Works**: 실제 임상 데이터 검증, 임상의 의존도·환자 안전·자원 사용에 대한 전향적 다기관 연구

### Take-Home Message

> **"Stage-specific fine-tuning substantially improved hierarchical KTAS prediction by open-weight SLMs. As clinician-facing decision support, SLM assistance was associated with lower under-triage and higher accuracy, alongside higher over-triage. The largest benefit and the greatest vulnerability to incorrect output occurred among nurses without KTAS certification, consistent with automation bias [6], supporting prospective evaluation as decision support rather than autonomous triage."**
>
> (단계별 미세조정은 오픈 웨이트 SLM의 계층형 KTAS 예측을 크게 향상시켰습니다. SLM 보조는 정확도를 높이고 under-triage를 줄였지만 over-triage를 늘렸습니다. 가장 큰 이득과 가장 큰 취약성이 모두 KTAS 미인증 간호사에게서 나타났으며, 이는 자동화 편향과 일치합니다. 따라서 자율 분류가 아닌 **의사결정 지원 도구**로서의 전향적 평가가 필요합니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

```text
Q: 왜 최종 KTAS Level만 예측하지 않고 Major → Minor → Level 계층을 따르나요?
A: 실제 KTAS 절차가 주호소 범주 → 구체 증상 → 중증도 순서로 진행되기 때문입니다. 계층을 따르면
   (1) 임상의가 모델 판단 경로를 단계별로 검증할 수 있고, (2) 앞 단계 예측으로 다음 단계 후보를
   제한해 탐색 공간을 줄일 수 있습니다. 실제로 exact-match(0.505)가 KTAS Top-1(0.682)보다 낮아,
   "Level만 맞힌" 경우를 걸러내는 데 경로 평가가 유용했습니다.

Q: 왜 자유 생성(free-text) 대신 constrained candidate scoring을 썼나요?
A: 174개 Minor Category처럼 라벨 공간이 정해진 분류에서 생성형 출력은 존재하지 않는 라벨을 만들 수 있습니다.
   후보 라벨의 우도를 점수화하면 항상 유효한 라벨이 나오고, Top-k 정확도도 자연스럽게 계산됩니다.

Q: SLM+Human 조건에서 confidence나 rationale을 보여주지 않은 이유는 무엇일까요?
A: 라벨만 보여주는 가장 단순한 인터페이스에서 순수한 "제안 효과"를 측정하려는 설계로 보입니다.
   저자들도 이를 한계로 꼽았는데, 불확실성 표시가 Group I의 automation bias를 줄이는지가
   자연스러운 후속 질문입니다.
```

#### Q. KTAS가 정확히 무엇인가요?

**KTAS(Korean Triage and Acuity Scale, 한국형 응급환자 분류도구)**는 응급실 내원 환자를 긴급도에 따라 5단계로 나누는 국가 표준 도구입니다. 캐나다의 CTAS를 바탕으로 개발되었고, 2016년부터 국내 응급의료기관에서 의무적으로 사용하고 있습니다.

- **Level 1**(즉시 소생술 필요)부터 **Level 5**(비응급)까지 5단계로 분류합니다.
- 분류는 **주호소 범주(Major) → 구체적 증상(Minor) → 중증도(Level)** 순서로 진행됩니다. 이 연구가 "계층형(hierarchical)" 예측을 강조하는 이유입니다.
- 주로 트리아지 간호사가 짧은 문진으로 판단하므로 숙련도에 따라 편차가 큽니다. 인용 연구에서는 분류의 14.7%가 전문가 판단과 달랐고, 대부분이 under-triage였습니다.

#### Q. 이 포스터는 왜 중요한가요?

이 연구는 "모델이 몇 % 맞히는가"가 아니라 **"AI가 옆에 있을 때 사람의 판단이 어떻게 바뀌는가"**를 측정했습니다.

- SLM 보조로 정확도는 오르고 under-triage는 줄었지만, over-triage는 늘었습니다.
- KTAS 미인증 간호사(Group I)에서 이득이 가장 컸지만, SLM이 틀렸을 때 원래 맞았던 판단의 **77.8%를 틀린 쪽으로 바꿨습니다**(자동화 편향).
- **가장 도움이 필요한 사용자가 가장 취약하다**는 이 결과는, AI를 누구에게 어떤 인터페이스로 보여줄지 정하는 근거가 됩니다.

#### Q. 실제 환자 데이터 없이도 우수 포스터로 선정된 이유는 무엇일까요?

심사 기준은 알 수 없으므로, 포스터 내용에 근거한 추정입니다.

1. **연구 질문과 데이터가 잘 맞습니다.** 인간–AI 상호작용 연구에서는 정답이 확실한 증례가 더 중요합니다. 임상의가 작성하고 동료 검토를 거친 가상 증례(vignette)는 KTAS 교육·시험과 같은 방식입니다. 반면 실제 EMR 기록은 정답 라벨 자체가 오분류되어 있을 수 있습니다.
2. **평가 설계가 탄탄합니다.** 무작위 교차 설계, 3주 washout, 숙련도별 4개 그룹, 데이터 구축에 참여하지 않은 평가자를 사용했습니다. 특히 SLM이 맞힌 증례와 틀린 증례를 의도적으로 섞어 **자동화 편향을 직접 측정**할 수 있게 했습니다.
3. **지표가 안전성 중심입니다.** 정확도만 보고하지 않고, 오류 방향(under/over), 순서형 라벨에 맞는 QWK, SLM 정오에 따른 판단 전이까지 분해했습니다.
4. **현실적으로 배포 가능합니다.** 4–8B 오픈 웨이트 모델이라 온프레미스로 운영할 수 있고, 환자 데이터를 외부로 보내지 않아도 됩니다.
5. **한계를 솔직하게 밝혔습니다.** 합성 데이터, hold-out test set 부재, 그룹당 평가자 2명을 스스로 한계로 명시했습니다. 결론도 "자율 분류가 아닌 의사결정 지원으로서의 전향적 평가"로 절제했습니다.

> **통계적으로 보면 약점도 있습니다.** 그룹당 2명으로는 평가자 간 변동을 추정할 수 없습니다. 또 over-triage 증가는 모든 paired 95% CI가 0을 포함해 통계적으로 확인되지 않았습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 아래 블록에 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 정확도 향상만 보고하지 않고 **오류 방향(under/over-triage)**과 **SLM 정오에 따른 결정 전이**까지 분해한 점이 인상적입니다. "AI가 도움이 되는가?"라는 질문을 "**누구에게, 어떤 방향으로, 어떤 대가를 치르고** 도움이 되는가?"로 바꿔 놓았습니다. 가장 도움이 필요한 비숙련 그룹이 동시에 가장 취약하다는 결과는 의료 AI 배포 설계에서 반드시 고려해야 할 지점입니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

이 포스터에서 차용할 수 있는 요소는 세 가지입니다.

- **(a) 계층형 라벨 예측**
- **(b) 임상의 교차 평가 설계**
- **(c) 안전성 지표 분해**: 오류 방향, QWK, AI 정오별 판단 전이

신경과 연구실 안의 통계/AI 연구원이라면 세 가지를 모두 직접 해 볼 수 있습니다. (a) 모델 구축과 (c) 분석은 직접 맡고, (b)의 평가자 그룹은 연구실의 전공의·펠로우·전문의로 구성할 수 있기 때문입니다.

**A. 통계 방법론: AI 보조 판독 연구 분석 프레임워크** ⭐ 우선순위 1

- 이 포스터는 평가자를 고정 효과로 다뤘습니다. 다음과 같이 발전시킬 수 있습니다.
  - 평가자와 증례를 교차 무작위효과로 두는 **GLMM / 순서형 혼합모형** (MRMC 분석의 확장)
  - 교차 설계의 시기 효과(period)와 이월 효과(carryover) 검정
  - 오류 방향(under/correct/over)의 다항 혼합모형 분석
  - **자동화 편향 지표**: $P(\text{correct} \rightarrow \text{incorrect} \mid \text{AI wrong})$를 숙련도의 함수로 추정
- 표본 수 설계도 다룰 수 있습니다. "자동화 편향을 검출하려면 평가자와 증례가 몇 명·몇 건 필요한가"를 시뮬레이션으로 제시합니다.
- 데이터 수집 없이 시뮬레이션과 공개 데이터만으로 방법론 논문을 쓸 수 있고, 이후 신경과 AI 판독 연구의 분석 기반이 됩니다.

**B. 계층 구조가 있는 신경과 진단 분류에 적용**

| 과제 | 계층 구조 | 비고 |
| :--- | :--- | :--- |
| **두통 진단 (ICHD-3)** | 1. Migraine → 1.1 Without aura → … | 계층 코드 체계라 KTAS와 구조가 가장 비슷하고, 문진 중심이라 텍스트 증례로 만들기 쉬움 |
| **뇌전증 발작 분류 (ILAE 2017)** | 발작 시작(focal/generalized) → 의식 유지 여부 → 운동/비운동 증상 | 전공의 vs 뇌전증 전문의 교차 평가에 적합 |
| **뇌졸중 아형 (TOAST)** | 5개 아형, 근거 수준(probable/possible) | 평가자 간 불일치가 큰 것으로 알려진 과제 |

- 전공의 → 펠로우 → 전문의로 숙련도 그룹을 구성하면 이 포스터의 4개 그룹 설계를 그대로 옮길 수 있습니다.

**C. 순서형 척도 자동 추출 + QWK 평가**

- **mRS(modified Rankin Scale)**, NIHSS를 퇴원·경과 기록에서 SLM으로 추출합니다. mRS는 0–6의 순서형 척도라 QWK와 under/over 방향 분석이 그대로 적용됩니다.
- 뇌졸중 레지스트리의 수작업 입력 부담을 줄이는 실용적 목적이 있습니다.
- 원내 데이터를 다루므로 온프레미스 SLM + LoRA 조합이 필요하며, 이 포스터가 바로 그 구성을 보여줍니다.

**D. 희귀 신경질환 합성 증례 데이터셋**

- 희귀 질환은 실제 증례가 부족합니다. 임상의가 작성하고 동료 검토를 거친 가상 증례로 교육·평가용 데이터셋을 구축할 수 있습니다.
- 이 포스터의 데이터 구축 절차(1,216건 생성 → 임상의 검토 후 727건 선별)를 참고합니다.

> **추천 조합**: **B 중 하나(두통 ICHD-3)를 처음부터 끝까지 수행**합니다. 계층형 SLM 구축 → 연구실 임상의 대상 교차 평가 → A의 혼합모형 분석 순서입니다. 통계와 AI 역량을 모두 쓰면서 연구실 임상의와 바로 협업할 수 있는 구성이고, A는 이 과정에서 방법론 논문으로 따로 정리할 수 있습니다.

---

### References (포스터 인용)

1. Moon SH, Shim JL, Park KS, Park CS. Triage accuracy and causes of mistriage using the Korean Triage and Acuity Scale. *PLoS One* 2019;14(9):e0216972.
2. Park JB, Lim TH. Korean Triage and Acuity Scale (KTAS). *J Korean Soc Emerg Med* 2017;28(6):547–551.
3. Hu EJ, Shen Y, Wallis P, Allen-Zhu Z, Li Y, Wang S, et al. LoRA: Low-rank adaptation of large language models. *ICLR* 2022.
4. Cohen J. Weighted kappa: Nominal scale agreement with provision for scaled disagreement or partial credit. *Psychol Bull* 1968;70(4):213–220.
5. Healthcare AI Research Institute (HARI), Seoul National University Hospital (SNUH). Hari-Q3-8B model card. Hugging Face. 2025.
6. Khera R, Simon MA, Ross JS. Automation bias and assistive AI: Risk of harm from AI-driven clinical decision support. *JAMA* 2023;330(23):2255–2257.
