---
title: "[KoSAIM] SCHERA: an Autonomous EHR Agent on the OMOP CDM — 포스터 심층 분석"
date: 2026-10-24 10:06:40 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["EHR Agent","OMOP CDM","Text-to-SQL","Tool Use","Local Open Model","EHRSQL","MIMIC-IV","KoSAIM 2026"]
description: "LLM이 사람 대신 직접 EHR 데이터베이스를 조회하는 에이전트 SCHERA를 OMOP CDM 위에 구축했습니다. 도구(SQL, 표준 개념 검색, CDM 문서)를 CDM의 표준 스키마와 어휘에만 의존하도록 만들어 CDM을 쓰는 어느 기관에서나 재사용할 수 있게 했습니다. MIMIC-IV OMOP 기반 139문항 벤치마크에서 로컬 오픈 모델 Gemma-4-31B-IT의 정확도가 0.460 → 0.597로 올라 상용 모델과의 격차가 24.5 → 13.0%p로 줄었고, SQL 오류는 1.78 → 0.91회로 감소했습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-schera-omop-ehr-agent-kosaim-2026-full.jpg
  alt: "[KoSAIM] SCHERA: an Autonomous EHR Agent on the OMOP CDM — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: SCHERA: Structured Clinical History with Episodic Retrieval and Agentic Exploration — an autonomous EHR agent on the OMOP CDM
  - <!-- TODO: 리본에 가려진 제목 일부("Episodic Retrieval and Agentic")는 약어 SCHERA로부터 추정, 확인 필요 -->
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: **Minseong Kim**¹, Hanjae Kim¹·², Younwoo Ki¹·³, Chang Hoon Han¹·³, Ji Woo Song⁴, Hyo Young Lee¹·⁵, Yiju Park¹·³, **Seng Chan You**¹·²·³·⁶
  - ¹연세대학교 의과대학 의생명시스템정보학교실, ²연세대 AI사회혁신연구원, ³연세대 디지털헬스연구원, ⁴세브란스병원, ⁵연세암병원 방사선종양학과, ⁶연세대 의과대학 대학원 의생명시스템정보학 (BK21)
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `EHR Agent`, `OMOP CDM`, `Text-to-SQL`, `Tool Use`, `Local Open Model`, `EHRSQL`, `MIMIC-IV`

![SCHERA 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-schera-omop-ehr-agent-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: SCHERA — an autonomous EHR agent on the OMOP CDM*

> **Executive Summary**  
> 임상에서 LLM을 쓰려면 여전히 **사람이 EHR 데이터를 뽑아서 넘겨줘야** 합니다. SCHERA는 LLM이 **OMOP CDM 데이터베이스를 직접 조회**하는 에이전트입니다. 도구(SQL, 표준 개념 검색, CDM 문서)가 CDM의 **표준 스키마·어휘·공개 명세에만** 의존하므로, CDM을 운영하는 어느 기관에서나 그대로 재사용할 수 있습니다. 139문항 벤치마크에서 로컬 오픈 모델 **Gemma-4-31B-IT(reasoning off)의 정확도가 0.460 → 0.597**로 올라 상용 모델(GPT-5.6-Sol)과의 격차가 **24.5 → 13.0%p**로 줄었고, 세션당 SQL 오류는 **1.78 → 0.91회**로 감소했습니다.

---

## 1. 연구 배경 및 문제 정의 (Background & Problem Gap)

1. **EHR 데이터 접근 문제** — 임상에서 LLM을 쓰려면 사람이 EHR 데이터를 추출해 넘겨줘야 합니다 [1]. EHR을 스스로 조회하는 에이전트가 있으면 이 부담을 없앨 수 있습니다.
2. **기존 EHR 에이전트의 한계** — 에이전트는 스키마와 임상 용어를 이해해야 하는데, 데이터 구조가 병원마다 다릅니다. 그래서 **한 기관용으로 만든 에이전트는 다른 기관으로 잘 옮겨지지 않습니다** [2–4].
3. **OMOP CDM이 해결의 기반** — 표준화된 스키마와 어휘(vocabulary)는 **기관 간 재사용 가능한 에이전트 도구**의 토대가 됩니다.
4. **로컬 오픈 모델의 필요성** — 의료 데이터는 병원 밖으로 잘 나가지 않습니다. 그래서 병원 내부에서 구동하는 **정확하면서도 빠른 오픈 모델**이 필요합니다.

---

## 2. 제안 방법론: SCHERA — Tools Built on the OMOP CDM

SCHERA는 OMOP CDM 데이터베이스를 스스로 조회합니다. 도구는 CDM의 표준 스키마, 어휘, 공개 명세에만 기반하므로 **CDM을 운영하는 어느 기관으로든 그대로 옮겨갈 수 있습니다.**

### Figure 1. Agent loop & SCHERA on the OMOP CDM

![Figure 1: Agent loop and SCHERA layers](/assets/img/posts/2026-10-24-kosaim-2026/poster-schera-omop-ehr-agent-kosaim-2026-figure-1.png)
*Figure 1: (A) 매 라운드 LLM은 텍스트로 답하거나 도구를 호출하고, 도구 결과는 컨텍스트로 돌아갑니다. (B) SCHERA의 Agent · Tool · Data 계층. 벤치마크에서는 submit_answer 도구로 답을 제출합니다.*

**(A) Agent loop**

```
Prompt ─▶ LLM ─▶ [Text or tool call?]
           ▲            │ tool                │ text
           │            ▼                     ▼
           └── result added to context ◀─ Run tool      Response
```

**(B) 3계층 구조**

| 계층 | 구성 요소 | 설명 |
| :--- | :--- | :--- |
| **Agent** | SCHERA agent | 질문을 받아 도구를 순차 호출 |
| **Tool** | **SQL** | 읽기 전용(read-only) 쿼리 실행 |
| | **Concept lookup** | 표준 개념(standard concept) 검색 |
| | **CDM docs** | CDM 문서를 필요할 때 불러옴(loaded on demand) |
| **Data** | OMOP CDM | 37개 CDM 테이블 + 표준 어휘 |
| | Documentation | OMOP CDM v5.3 공식 문서 |

- 핵심은 **도구가 특정 병원의 테이블명이나 라벨이 아니라 CDM 표준에만 의존**한다는 점입니다. 이 덕분에 기관 간 이식성이 생깁니다.

---

## 3. 벤치마크 및 실험 설정 (Benchmark & Setup)

- **Benchmark**: MIMIC-III 기반 **EHRSQL** [5] 질문 템플릿을 **MIMIC-IV OMOP CDM**용으로 다시 작성하고, 데이터베이스의 실제 환자 ID와 임상 값으로 채웠습니다.
  - 템플릿당 1문항, 총 **139문항** (답할 수 있는 문항 127개, **답할 수 없는 문항 12개**)
- **Rephrased values**: 진단명, 약물명, 검사명을 **같은 의미의 다른 표현**으로 바꿨습니다. 에이전트는 DB 라벨을 문자열로 맞추는 대신 **표준 개념을 찾아야** 합니다.
- **Ground truth**: 사람이 직접 작성한 참조 SQL의 실행 결과
- **Models**: Gemma-4-31B-IT (reasoning off / on), 상용 모델 참조로 GPT-5.6-Sol
  - GPT-5.6-Sol은 상용 API, Gemma는 로컬 GPU에서 실행했습니다.

### Table 1. 도구 구성 (Configurations)

| Configuration | Tools | Max rounds |
| :--- | :--- | :---: |
| Single-turn SQL | SQL | 1 |
| SQL | SQL | 20 |
| + Concept lookup | SQL, Concept lookup | 20 |
| + CDM documentation | SQL, Concept lookup, CDM documentation | 20 |

---

## 4. 실험 결과 (Results)

### 4-1. 도구 추가에 따른 정확도와 응답 시간

![Figure 2: Accuracy and median time per session](/assets/img/posts/2026-10-24-kosaim-2026/poster-schera-omop-ehr-agent-kosaim-2026-figure-2.png)
*Figure 2: 정확도(위)와 세션당 응답 시간 중앙값(아래), n = 139*

| Model | Single-turn SQL | SQL (multi-turn) | + Concept lookup | + CDM documentation | 응답 시간 중앙값 (all tools) |
| :--- | ---: | ---: | ---: | ---: | ---: |
| Gemma-4-31B-IT (reasoning off) | 0.137 | 0.460 | 0.475 | **0.597** | **56 s** |
| Gemma-4-31B-IT (reasoning on) | 0.079 | 0.619 | 0.604 | 0.647 | 164 s |
| GPT-5.6-Sol | 0.187 | 0.705 | 0.734 | **0.727** | 67 s |

- **Single-turn → Multi-turn**: 한 번에 SQL을 쓰는 방식은 모든 모델에서 0.2 미만이었습니다. 여러 번 시도하고 결과를 확인하는 **multi-turn 에이전트 루프 자체가 가장 큰 향상**을 만들었습니다.
- **표준화 기반 도구의 효과**: Gemma(reasoning off)는 0.460 → **0.597**로 올랐고, GPT-5.6-Sol과의 격차는 **24.5 → 13.0%p**로 줄었습니다. 반면 GPT-5.6-Sol은 거의 변하지 않았습니다 (0.705 → 0.727).
- **정확도-속도 트레이드오프**: 모든 도구를 쓸 때 reasoning on은 정확도를 +0.05 더 얻었지만 시간이 약 3배(56 s → 164 s) 걸렸습니다.

### 4-2. SQL 실행 신뢰성

![Figure 3: Mean SQL errors per session](/assets/img/posts/2026-10-24-kosaim-2026/poster-schera-omop-ehr-agent-kosaim-2026-figure-3.png)
*Figure 3: 세션당 평균 SQL 오류 수. Single-turn SQL은 쿼리를 1회만 실행하므로 제외*

| Model | SQL | + Concept lookup | + CDM documentation |
| :--- | ---: | ---: | ---: |
| Gemma-4-31B-IT (reasoning off) | 1.78 | 1.47 | **0.91** |
| Gemma-4-31B-IT (reasoning on) | 0.71 | 0.57 | 0.27 |
| GPT-5.6-Sol | 0.38 | 0.37 | 0.24 |

- 개념 검색과 CDM 문서를 함께 쓰면 Gemma(reasoning off)의 SQL 오류가 **1.78 → 0.91회**로 절반 가까이 줄었습니다. 표준화 도구가 에이전트의 **스키마 탐색과 유효한 SQL 작성**을 돕는다는 뜻입니다.

---

## 5. 결론 및 향후 과제 (Conclusion & Future Work)

1. **Reusable tools** — OMOP CDM 덕분에 기관 간 재사용 가능한 에이전트 도구를 만들 수 있고, 이 도구들은 로컬 오픈 모델의 EHR 질의 정확도를 높입니다.
2. **Practical balance** — 표준화 도구가 있으면, 추가 reasoning 없는 모델로도 정확도와 응답 시간의 균형을 맞출 수 있습니다.
3. **Future work** — 목표 지향적 자율성(goal-directed autonomy) 벤치마크, 비표준 스키마와의 비교, 다기관 검증

### Take-Home Message

> **"The OMOP CDM makes it possible to build agent tools for reuse across institutions, and these tools raise a local open model's accuracy on EHR questions."**
>
> (OMOP CDM 위에 만든 표준화 도구는 기관 간 재사용이 가능하고, 병원 내부에서 돌리는 오픈 모델의 EHR 질의 정확도를 상용 모델 수준에 가깝게 끌어올립니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. OMOP CDM이 무엇이고, 왜 이 연구에서 중요한가요?

**OMOP CDM(Observational Medical Outcomes Partnership Common Data Model)**은 OHDSI 커뮤니티가 관리하는 **의료 데이터 공통 모델**입니다. 병원마다 다른 EHR 데이터를 같은 테이블 구조(예: `person`, `condition_occurrence`, `drug_exposure`, `measurement`)와 같은 표준 용어 체계로 변환합니다.

- 국내에서도 여러 대학병원이 CDM으로 변환한 데이터를 갖추고 있어 다기관 연구에 쓰입니다.
- 이 연구의 핵심 아이디어는 "에이전트를 병원마다 새로 만들지 말고, **CDM 표준에 맞춰 한 번 만들어 어디서나 쓰자**"입니다.

#### Q. "Concept lookup" 도구가 왜 그렇게 중요한가요?

질문에 나오는 "심근경색", "MI", "급성 심근경색증"은 모두 같은 의미지만 DB 라벨과 문자열이 다릅니다. 연구진은 일부러 **값을 다른 표현으로 바꾼(rephrased) 벤치마크**를 만들어, 에이전트가 문자열 매칭이 아니라 **표준 개념 ID를 찾아야만** 맞힐 수 있게 했습니다. Concept lookup은 이 과정을 돕는 도구이고, 실제 임상 질문의 표현 다양성을 반영한 현실적인 설계입니다.

#### Q. 이 포스터를 통계적으로 비판적으로 본다면?

1. **표본 수와 불확실성** — n = 139에서 정확도 0.6의 95% 신뢰구간 반폭은 약 $\pm 1.96\sqrt{0.6 \times 0.4 / 139} \approx \pm 0.08$입니다. GPT-5.6-Sol의 0.705 → 0.734 → 0.727 같은 차이는 **잡음 범위 안**입니다. 같은 문항에 대한 구성 간 비교이므로 **McNemar 검정** 같은 paired 분석이 필요합니다.
2. **반복 실행 없음** — LLM 출력은 확률적인데, 여러 번 실행한 평균이나 분산이 보고되지 않았습니다.
3. **답할 수 없는 문항의 성능 미분리** — 12개 unanswerable 문항에서 에이전트가 "답할 수 없음"을 제대로 말했는지(abstention)가 따로 보고되지 않았습니다. 임상 안전성에서는 이것이 중요합니다.
4. **단일 데이터베이스** — 기관 간 재사용성이 핵심 주장인데, 검증은 MIMIC-IV 하나로만 했습니다. 저자들도 이를 향후 과제로 명시했습니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- **실용적 문제 설정** — "기관 간 이식성"과 "로컬 오픈 모델"이라는, 실제 병원 배포에서 가장 큰 두 장벽을 정면으로 다뤘습니다.
- **깔끔한 ablation 설계** — 도구를 하나씩 추가하며 정확도, 시간, SQL 오류를 함께 보고해 각 구성 요소의 기여를 분리했습니다.
- **현실적인 벤치마크 변형** — 값을 다른 표현으로 바꾸고 답할 수 없는 문항을 섞어, 단순 text-to-SQL보다 어려운 조건을 만들었습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 2번 포스터가 "검색된 정보가 판단을 오염시키지 않게" 정보 흐름을 설계했다면, SCHERA는 **"표준 위에 도구를 만들면 작은 모델도 큰 모델을 따라잡는다"**를 보여줍니다. 모델을 키우는 대신 **데이터 표준과 도구**에 투자하는 방향이 병원 현장에 더 현실적이라는 메시지로 읽힙니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 신경과 연구 질문용 CDM 에이전트 벤치마크 구축** ⭐ 우선순위 1

- 소속 기관의 OMOP CDM에서 신경과 연구자가 실제로 묻는 질문으로 벤치마크를 만듭니다.
  - 예: "최근 5년간 허혈성 뇌졸중으로 입원한 환자 중 퇴원 시 DOAC를 처방받은 환자 수", "레비티라세탐 단독요법 후 1년 내 약제를 추가한 뇌전증 환자 비율"
- 신경과 임상의가 질문과 참조 SQL을 검토하고, 답할 수 없는 문항도 일부러 넣습니다.
- 통계/AI 연구원은 평가 설계(paired 비교, 반복 실행, 신뢰구간)를 맡습니다. 이 포스터의 약점을 보완하는 기여가 됩니다.

**B. 코호트 정의(phenotyping) 보조 에이전트**

- 신경과 연구의 첫 단계는 대개 코호트 정의(예: 뇌졸중 환자, 치매 환자를 어떤 진단 코드와 약물로 정의할지)입니다.
- Concept lookup 도구를 확장해, 연구자가 자연어로 코호트를 설명하면 **표준 개념 세트(concept set)를 제안**하고 임상의가 검토하는 흐름을 만들 수 있습니다.
- 에이전트가 제안한 concept set과 기존 검증된 phenotype 정의의 일치도를 평가하는 연구로 이어질 수 있습니다.

**C. 실행 가능성 검토(feasibility query) 자동화**

- 연구 계획 단계의 "조건에 맞는 환자가 몇 명인가?" 같은 질문은 지금 데이터팀에 요청해야 하는 경우가 많습니다.
- 집계 결과만 반환하는 **읽기 전용 에이전트**를 원내 로컬 모델로 운영하면, 연구 계획 속도를 높이면서 개인정보 노출도 줄일 수 있습니다.

> **추천**: **A로 시작해 B로 확장**합니다. A는 연구실의 신경과 질문과 통계 평가 역량을 바로 활용할 수 있고, A에서 만든 벤치마크가 B의 평가 기반이 됩니다. 소속 기관에 OMOP CDM이 구축되어 있는지 먼저 확인하는 것이 출발점입니다.

---

### References (포스터 인용)

1. Goh E, et al. GPT-4 assistance for improvement of physician performance on patient care tasks: a randomized controlled trial. *Nat Med* 2025;31(4):1233–1238.
2. Ferber D, et al. Towards autonomous medical artificial intelligence agents. *Nature* 2026;655(8125):1282–1291.
3. Liao Y, et al. AgentEHR: Advancing autonomous clinical decision-making via retrospective summarization. arXiv:2601.13918. 2026.
4. Jiang Y, et al. MedAgentBench: A virtual EHR environment to benchmark medical LLM agents. *NEJM AI* 2025;2(9):AIdbp2500144.
5. Lee G, et al. EHRSQL: A practical text-to-SQL benchmark for electronic health records. arXiv:2301.07695.
