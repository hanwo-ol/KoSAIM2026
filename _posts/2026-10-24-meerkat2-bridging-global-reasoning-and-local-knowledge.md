---
title: "[KoSAIM] Meerkat2: Bridging Global Reasoning and Local Knowledge for Efficient Medical AI — 포스터 심층 분석"
date: 2026-10-24 10:07:54 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Medical LLM","Knowledge Distillation","Synthetic Data","Korean Medical Law","KorMedMCQA","KMLE","Mechanistic Interpretability","KoSAIM 2026"]
description: "영어 중심 의료 LLM은 한국의 진료 지침과 의료법을 모릅니다. Meerkat2-Pro(14B/32B/30B-A3B)는 두 가지 방법으로 이를 보완합니다. 한국 의료법 조문 12개로 18.7K 법률 QA를 합성해 교사 모델 증류보다 법률 정확성을 크게 높였고(Legal fidelity 98.7 vs 79.0), 교사의 응답 스타일을 시스템 프롬프트로 넣는 Verbosity-Decoupled Distillation(VDD)으로 긴 추론 증류의 정확도는 유지하면서 출력 길이를 약 절반으로 줄였습니다. 미공개 2025년 제89회 의사국가시험에서 Meerkat2-Pro-32B가 89.7/89.0점으로 Qwen3-235B를 넘었습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-full.jpg
  alt: "[KoSAIM] Meerkat2: Bridging Global Reasoning and Local Knowledge for Efficient Medical AI — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Meerkat2: Bridging Global Reasoning and Local Knowledge for Efficient Medical AI
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: Jungwoo Park¹·², Hyeongsoon Hwang¹·², **Sunkyu Kim**², Sanghoon Lee¹·²\*, Jaewoo Kang¹·²\* (\*Corresponding authors)
  - ¹고려대학교 (Korea University), ²AIGEN Sciences
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `Medical LLM`, `Knowledge Distillation`, `Synthetic Data`, `Korean Medical Law`, `KorMedMCQA`, `KMLE`, `Mechanistic Interpretability`

![Meerkat2 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Meerkat2: Bridging Global Reasoning and Local Knowledge for Efficient Medical AI*

> **Executive Summary**  
> 영어 중심 의료 LLM은 **한국의 진료 지침과 의료법**을 모르고, 이는 번역만으로 해결되지 않습니다. Meerkat2-Pro(14B, 32B, 30B-A3B MoE)는 두 가지 방법으로 이를 보완합니다.
> 1. **Statute-grounded QA synthesis**: 한국 의료법 조문 12개로 법률 QA 18.7K쌍을 합성했습니다. 교사 모델 증류(Legal fidelity 79.0)보다 법률 정확성이 크게 높았습니다(98.7).
> 2. **Verbosity-Decoupled Distillation(VDD)**: 교사의 응답 스타일을 시스템 프롬프트에 지시문으로 넣어, 긴 추론 증류의 정확도는 유지하면서 출력 길이를 **약 절반**으로 줄였습니다.
>
> 미공개 2025년 제89회 의사국가시험(KMLE)에서 **Meerkat2-Pro-32B가 89.7 / 89.0점(영/한)**으로 Qwen3-235B(85.2 / 88.4)를 넘었습니다.

---

## 1. 연구 배경 및 문제 정의 (Problem and Setup)

1. **현지 지식의 부재** — 영어 중심 의료 LLM은 **한국의 진료 지침**과 **의료법**을 놓칩니다. 현지화(adaptation)는 번역과 다릅니다.
2. **교사 증류의 한계** — 더 강한 교사 모델로부터의 증류(distillation)는 추론 능력은 잘 옮기지만, 법 조문처럼 **문자 그대로의 정확성(verbatim fidelity)**이 필요한 지식은 보장하지 못합니다.
3. **장황함의 비용** — 긴 추론(rationale)으로 증류하면 성능은 오르지만 학생 모델이 **장황함까지 물려받습니다**. 임상 의사결정 지원에서는 응답이 길수록 매번 시간과 비용이 듭니다.

### 기여 (Contributions)

- **Meerkat2-Pro** 모델 패밀리: **14B, 32B, 30B-A3B(MoE)**
- **Statute-grounded QA synthesis**: 의료법 문항에서 교사 증류보다 우수 → 고정밀 지식에는 **원문 기반(source-grounded) 지도 학습**이 필요함을 보임
- **Verbosity-Decoupled Distillation(VDD)**: 긴 추론 증류의 정확도를 **약 절반의 출력 길이**로 유지하고, 왜 작동하는지를 **그래디언트 수준**에서 설명

---

## 2. 방법 1: 조문 기반 의료법 QA 합성 (Statute-Grounded Law QA Synthesis)

- **문제**: 더 강한 교사 모델로부터 증류해도, 의료법 문제에서 **환각되거나 원문에 충실하지 않은 근거**가 나타납니다.
- **방법**: **한국 의료 관련 법령 12개**를 직접 수집하고, 이를 바탕으로 **법률 QA 18.7K쌍**을 합성했습니다.
- **Legal fidelity**: 합성 QA **98.7** vs 증류 79.0
  - 교사 모델은 법률 시험 점수는 높지만, 그 **근거(rationale)에는 법적 오류**가 섞여 있습니다. 그래서 증류는 이득이 없었습니다.

### Figure 1. 의료법 문항 정확도 (KorMedMCQA Doctor subset, %)

![Figure 1: Medical-law questions accuracy](/assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-figure-1.png)
*Figure 1: 의료법 문항에서 Qwen3 base, 교사 증류, 조문 기반 합성(ours)의 정확도 비교. 점선은 GPT-5.1(76.9)*

| Backbone | Qwen3 base | + Teacher distillation | **+ Statute synthesis (ours)** |
| :--- | ---: | ---: | ---: |
| Qwen3-14B | 57.7 | 57.0 | **71.2** |
| Qwen3-30B-A3B | 61.5 | 55.8 | **73.1** |
| Qwen3-32B | 63.5 | 57.7 | **84.6** |
| *(참고) GPT-5.1* | | | *76.9* |

- **교사 증류는 오히려 성능을 떨어뜨리거나 그대로**였습니다 (예: 30B-A3B 61.5 → 55.8).
- **조문 기반 합성은 세 모델 모두에서 크게 향상**되었고, 32B는 GPT-5.1(76.9)을 넘었습니다 (84.6).

---

## 3. 방법 2: Verbosity-Decoupled Distillation (VDD)

### 3-1. 아이디어

- **왜 증류하면 응답이 길어지는가?** 모델이 추론 패턴뿐 아니라 **응답 스타일(response style)까지** 배우기 때문입니다.
- **VDD**: 교사의 응답 스타일을 **시스템 프롬프트에 지시문으로** 넣고 학습합니다.
  - 그러면 "장황하게 써라"는 정보는 이미 프롬프트에 있으므로, 모델 가중치가 그 스타일을 학습할 필요가 사라집니다. 즉, **응답 스타일에 대한 그래디언트가 상쇄(cancelled)**됩니다.
  - 추론 시에는 그 지시문 없이 간결하게 답하는 것으로 해석됩니다 (포스터에는 추론 시 프롬프트 처리 방식이 명시되어 있지 않음).

### 3-2. 기계적 해석(mechanistic interpretability)으로 검증

1. **장황함 관련 attention head 찾기** (heatmap)
2. 시스템 프롬프트가 **장황함 방향의 그래디언트를 상쇄**하는지 확인 (table)

### Figure 2. Attention head 기여도와 장황함 방향 그래디언트

![Figure 2: Attention head contributions and verbosity gradients](/assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-figure-2.png)
*Figure 2: (위) 층·head별 장황함 기여도 heatmap. (아래) 학습 1스텝이 attention head를 장황한 생성 쪽(+) 또는 반대쪽(−)으로 미는 정도를 상위 1%, 10%, 전체 head에 대해 합산한 값*

| Training CoTs | Top 1% | Top 10% | All heads |
| :--- | ---: | ---: | ---: |
| Concise | −12.2 | −30.4 | −63.0 |
| Verbose | +31.1 | +89.7 | +251.2 |
| **Verbose + system prompt (VDD)** | **−7.9** | **−19.0** | **−37.1** |

- 장황한 CoT로 학습하면 head들이 강하게 장황함 쪽(+251.2)으로 밀립니다.
- 같은 장황한 CoT라도 **VDD 프롬프트를 붙이면 부호가 뒤집혀**(−37.1) 간결한 CoT로 학습한 것과 같은 방향이 됩니다.

### 3-3. VDD 결과: 짧으면서도 강한 추론

![Figure 3: VDD accuracy and output length](/assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-figure-3.png)
*Figure 3: Qwen3-14B에서 학습 방식별 정확도와 평균 출력 토큰 수*

| Qwen3-14B | MedQA acc. (avg. tokens) | KorMedMCQA En acc. (avg. tokens) | KorMedMCQA Ko acc. (avg. tokens) |
| :--- | ---: | ---: | ---: |
| Base | 76.0 (492) | 75.5 (436) | 72.4 (567) |
| + Verbose CoTs | 86.1 (757) | 84.2 (613) | 82.0 (967) |
| + Concise CoTs | 83.4 (342) | 82.5 (279) | 81.7 (530) |
| **+ VDD (ours)** | **86.5 (362)** | **84.3 (296)** | **82.7 (539)** |

- **Verbose CoT**: 정확도는 높지만 출력이 깁니다.
- **Concise CoT**: 출력은 짧지만 정확도가 떨어집니다.
- **VDD**: Verbose 수준(또는 그 이상)의 정확도를 **Concise 수준의 길이**로 달성했습니다. MedQA에서는 757 → 362 토큰으로 **약 48%** 길이였습니다.

---

## 4. 종합 평가 결과 (Evaluation & Results)

*Meerkat2-Pro = enterprise edition · Instant(non-thinking) mode · Doctor subset = 한국 의사국가시험(KMLE) 문항*

### Figure 4. 추론 효율과 벤치마크 성능

![Figure 4: Inference efficiency and benchmark results](/assets/img/posts/2026-10-24-kosaim-2026/poster-meerkat2-kosaim-2026-figure-4.png)
*Figure 4: (왼쪽) 추론 비용(PFLOPs) 대비 KorMedMCQA 정확도. (오른쪽) 모델별 벤치마크 성능*

| Model | MedQA | KorMedMCQA En all (Doctor) | KorMedMCQA Ko all (Doctor) | 89th KMLE 2025 (unseen) En / Ko |
| :--- | ---: | ---: | ---: | ---: |
| Qwen3-14B | 76.0 | 75.5 (75.6) | 72.4 (67.1) | 71.0 / 63.2 |
| **Meerkat2-Pro-14B (ours)** | **86.5** | **84.3 (89.7)** | **82.7 (87.8)** | **86.5 / 84.5** |
| Qwen3-30B-A3B | 81.2 | 77.9 (77.9) | 74.7 (72.4) | 74.2 / 70.3 |
| **Meerkat2-Pro-30B-A3B (ours)** | **86.3** | **84.7 (90.1)** | **84.0 (89.2)** | **84.5 / 87.7** |
| Qwen3-32B | 79.2 | 79.4 (82.1) | 79.8 (79.3) | 81.3 / 73.6 |
| **Meerkat2-Pro-32B (ours)** | **87.8** | **86.1 (90.3)** | **86.2 (91.3)** | **89.7 / 89.0** |
| Qwen3-235B-A22B (reference) | 89.8 | 86.7 (88.7) | 88.0 (91.0) | 85.2 / 88.4 |

- **활성 파라미터 3B**인 Meerkat2-Pro-30B-A3B가 영어 Doctor subset에서 **Qwen3-235B-A22B를 넘었습니다** (90.1 vs 88.7).
- 모든 Meerkat2-Pro 모델이 각 Qwen3 base 대비 **MedQA +5–11%p, KorMedMCQA +6–10%p** 향상되었습니다.
- 학습에 쓰이지 않은 **2025년 제89회 KMLE**에서 Meerkat2-Pro-32B는 **89.7 / 89.0**으로 Qwen3-235B(85.2 / 88.4)보다 높았습니다.
- 추론 효율 그래프에서 Meerkat2-Pro 모델들은 **낮은 추론 비용(PFLOPs)에서 높은 정확도** 영역(좌상단)에 위치합니다.

---

## 5. 결론 및 의의

1. **현지 지식은 원문에서 가져와야 합니다.** 법 조문처럼 정확성이 중요한 지식은 강한 교사 모델의 증류보다 **원문 기반 합성**이 훨씬 효과적입니다.
2. **추론 능력과 응답 스타일은 분리할 수 있습니다.** VDD는 스타일을 프롬프트로 "흡수"시켜 가중치가 추론만 배우게 합니다.
3. **작은 모델로도 큰 모델 수준의 의료 성능**을 낼 수 있어, 병원 내 배포 비용을 줄일 수 있습니다.

### Take-Home Message

> **"Adaptation is not translation — high-fidelity local knowledge needs source-grounded supervision, and reasoning can be distilled without inheriting verbosity."**
>
> (현지화는 번역이 아닙니다. 법령처럼 정확해야 하는 지식은 원문 기반 지도 학습이 필요하고, 추론 능력은 장황함을 물려받지 않고도 증류할 수 있습니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. VDD를 쉽게 설명하면?

학생에게 선배의 풀이 노트를 보여주면서 공부시키면, 학생은 **풀이 방법**과 함께 선배의 **말투(길고 장황한 서술)**도 따라 배웁니다. VDD는 노트 맨 위에 "이 노트는 일부러 자세히 쓴 것임"이라는 메모를 붙여 두는 방법입니다.

- 학생은 "장황함은 메모 때문"이라고 이해하고, **풀이 방법만** 내면화합니다.
- 시험(추론)에서 메모가 없으면 간결하게 답합니다.
- 포스터의 그래디언트 표는 이 직관이 실제 모델 내부에서도 일어난다는 증거입니다. 메모(시스템 프롬프트)를 붙이면 장황함 방향의 학습 신호가 **+251.2에서 −37.1로 뒤집힙니다.**

#### Q. 왜 교사 증류는 의료법 문제에서 효과가 없었나요?

포스터에 따르면 교사 모델은 법률 시험 점수는 높지만, 그 **근거(rationale)에 법적 오류**가 섞여 있었습니다. 증류에서는 학생 모델이 정답 하나보다 훨씬 긴 **근거 문장 전체**를 학습하므로, 틀린 근거를 그대로 배우게 된다고 해석할 수 있습니다. 그래서 오히려 성능이 떨어지기도 했습니다(30B-A3B: 61.5 → 55.8). 4번 포스터(오류 조건부 선택)와 마찬가지로, **학습 데이터의 "정확성"이 양이나 난이도보다 중요**하다는 메시지입니다.

#### Q. 비판적으로 본다면?

1. **불확실성 표시 없음** — 신뢰구간이나 반복 실행 결과가 없습니다. 특히 89회 KMLE는 문항 수가 수백 개 수준이라, 1–3%p 차이는 통계적으로 확인되지 않을 수 있습니다.
2. **Legal fidelity 측정 방법이 불명확** — 98.7 vs 79.0이 핵심 수치인데, 누가(사람 또는 LLM 평가자) 어떤 기준으로 채점했는지가 포스터에 없습니다.
3. **벤치마크 오염 가능성** — KorMedMCQA는 과거 국가시험 문항 기반입니다. 학습 데이터와의 중복 검사 여부가 명시되지 않았습니다. 미공개 89회 KMLE를 따로 보고한 것은 이를 의식한 좋은 설계입니다.
4. **상용 모델** — Meerkat2-Pro는 AIGEN Sciences의 enterprise edition이라 가중치와 학습 데이터가 공개되지 않을 수 있어, 독립적 재현이 어렵습니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- **한국 의료 현장에 특화된 문제**(의료법, KMLE)를 정면으로 다뤘습니다.
- 성능 수치만이 아니라 **기계적 해석으로 "왜 작동하는지"**까지 보였습니다.
- 작은 모델(활성 3B)로 235B 모델을 넘는 등 **배포 효율성**이 뚜렷합니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> "강한 교사를 따라 하는 것"과 "원문을 정확히 배우는 것"이 다르다는 점이 인상적입니다. 4번 포스터가 "모델 예측과 어긋나는 라벨이 모델을 망친다"를 보였다면, 이 포스터는 "**교사의 그럴듯하지만 틀린 근거가 학생을 망친다**"를 보여줍니다. 두 연구 모두 데이터의 정확성이 핵심이라는 같은 결론으로 이어집니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 신경과 진료지침 기반 QA 합성** ⭐ 우선순위 1

- 국내 뇌졸중 진료지침, 뇌전증·치매·두통 관련 학회 지침처럼 **정확성이 중요한 한국 신경과 지침**을 원문으로 삼아 QA를 합성합니다.
- 이 포스터의 방법(원문 기반 합성 vs 교사 증류)을 그대로 비교하면, "신경과 지침 지식에서도 원문 기반 합성이 우월한가"를 검증하는 연구가 됩니다.
- 연구실 신경과 임상의가 합성 QA의 정확성(legal fidelity에 해당하는 **guideline fidelity**)을 평가하고, 통계/AI 연구원은 평가자 간 일치도와 신뢰구간을 포함한 평가 설계를 맡습니다.

**B. VDD를 임상 문서 요약에 적용**

- 퇴원 요약, 판독문 요약처럼 **간결함이 중요한 생성 과제**에서, 긴 설명으로 학습하되 출력은 짧게 만드는 VDD를 적용합니다.
- 평가는 정확도와 길이를 함께 보고하고, 임상의 선호도 평가를 추가합니다.

**C. 국가시험 문항 기반 평가의 통계적 보완**

- 89회 KMLE처럼 **학습에 쓰이지 않은 최신 시험**으로 평가하는 설계는 좋은 관행입니다.
- 신경과 전문의 시험 문항이나 교육용 증례로 비슷한 "미공개 평가 세트"를 만들고, 문항 수에 맞는 신뢰구간과 모델 간 paired 비교를 표준으로 보고하는 평가 틀을 만들 수 있습니다.

> **추천**: **A로 시작**합니다. 연구실의 신경과 전문성(지침 해석, 정확성 평가)과 통계/AI 역량(합성 파이프라인, 평가 설계)을 모두 쓰면서, 한국 신경과에 특화된 기여를 할 수 있습니다.

---

### References

포스터에 별도 참고문헌 목록은 없었습니다.
