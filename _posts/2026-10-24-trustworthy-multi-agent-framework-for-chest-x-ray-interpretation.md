---
title: "[KoSAIM] A Trustworthy Multi-Agent Framework for Chest X-ray Interpretation with Image-Grounded Verification — 포스터 심층 분석"
date: 2026-10-24 10:06:30 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Chest X-ray","Multi-Agent","Retrieval-Augmented Generation (RAG)","Vision-Language Model (VLM)","Hallucination","Evidence Isolation","MIMIC-CXR","KoSAIM 2026"]
description: "RAG 기반 흉부 X-ray 판독에서 VLM이 다른 환자의 판독문 소견을 영상 근거 없이 복사하는 문제를 막기 위해, 검색된 텍스트는 '질문 생성' 단계에만 쓰고 영상 판독·검증·최종 결정 에이전트는 다른 환자의 텍스트를 전혀 보지 못하게 격리한 4-에이전트 프레임워크입니다. MIMIC-CXR에서 Qwen3-VL-8B 단독 대비 정밀도를 유지하면서 Recall +0.053, F1 +0.040을 얻었고, 무관한 환자 판독문 치환이나 거짓 소견 주입에도 성능이 떨어지지 않았습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-full.jpg
  alt: "[KoSAIM] A Trustworthy Multi-Agent Framework for Chest X-ray Interpretation — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: A Trustworthy Multi-Agent Framework for Chest X-ray Interpretation with Image-Grounded Verification
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: **Yujoong Kim**¹, **Sang Hyun Park**²\* (\*Corresponding author)
  - ¹Graduate School of Artificial Intelligence, POSTECH
  - ²Department of Computer Science & Engineering, POSTECH
  - <!-- TODO: 리본에 일부 가려진 제1저자 이름(Yujoong) 철자 확인 -->
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `Chest X-ray`, `Multi-Agent`, `RAG`, `VLM`, `Hallucination`, `Evidence Isolation`, `MIMIC-CXR`

![A Trustworthy Multi-Agent Framework for Chest X-ray Interpretation 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: A Trustworthy Multi-Agent Framework for Chest X-ray Interpretation with Image-Grounded Verification*

> **Executive Summary**  
> RAG 기반 흉부 X-ray 판독에서 VLM은 검색된 **다른 환자의 판독문 소견을 영상 근거 없이 그대로 복사**하는 경향이 있습니다. 이렇게 만들어진 문장은 정상 문장과 구별하기 어렵습니다. 이 연구는 검색된 텍스트가 **"무엇을 물어볼지"만 결정하고 "무엇을 보고할지"는 결정하지 못하도록** 4개 에이전트(Questioning → Perception → Verification → Decision)로 정보 흐름을 분리했습니다. MIMIC-CXR에서 Qwen3-VL-8B 단독 대비 정밀도를 유지하면서 **Recall +0.053, F1 +0.040**을 얻었고, 무관한 환자 판독문이나 거짓 소견이 섞여도 성능이 떨어지지 않았습니다.

---

## 1. 연구 배경 및 문제 정의 (Background & Problem Gap)

### 1-1. RAG가 흉부 X-ray 판독에 만드는 새로운 위험

- **Retrieval-augmented generation(RAG)**은 자동 흉부 X-ray 판독 성능을 높이는 데 널리 쓰입니다 [1, 2].
- 그런데 검색되는 판독문은 **다른 환자의 것**입니다.
- VLM은 이런 판독문의 소견을 **현재 영상에 시각적 근거가 없어도 복사**하는 경향이 있습니다.
- 복사된 문장("…small left pleural effusion…")은 문법적으로도 임상적으로도 그럴듯해서, **정상 소견 문장과 구별하기 어렵습니다**.

### 1-2. 연구 목표

- **보고되는 모든 소견이 영상 자체에 근거(grounded)하도록** 하는 멀티 에이전트 프레임워크를 구축합니다.
- 핵심 아이디어는 "Conventional RAG: *Retrieved text decides what is reported*" → "Ours: *Retrieved text only decides what to ask*"입니다.

### Figure 1. Conventional RAG vs. Image-grounded agents

![Figure 1: Conventional RAG vs. image-grounded agents](/assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-figure-1.png)
*Figure 1: (a) 기존 RAG는 검색 텍스트가 보고 내용을 결정하여 복사 텍스트가 생기고, (b) 제안 방법은 검색 텍스트가 질문만 결정하여 영상에 근거한 보고를 만듭니다.*

---

## 2. 제안 방법론 및 핵심 아키텍처 (Proposed Methodology)

네 개의 협력 에이전트가 각 검사를 여러 라운드에 걸쳐 처리합니다. **검색된 텍스트는 질문 생성 단계에만 머뭅니다.**

### 2-1. 두 개의 영역(Zone)으로 정보 흐름 분리

**(1) Retrieved-text zone: 검색 및 질문 생성**
- Query image → Report DB에서 Top-K 판독문 검색
- ① **Questioning agent**: 검색된 판독문을 **후보 소견에 대한 질문**으로 변환합니다 (예: Q1. Pleural effusion? Q2. Cardiomegaly? Q3. Pneumothorax?).
- **"Retrieved text stops here — only questions pass"**: 이 경계를 넘는 것은 질문뿐입니다.

**(2) Image-grounded zone: 검색 텍스트 없음**
- ② **Perception agent**: 영상만 보고 각 질문에 답합니다 (Q1 → Yes, Q2 → No, Q3 → Yes).
- ③ **Verification agent**: 양성 답변만 **detector로 재확인**합니다 (Q1 ✓ supported, Q3 ✗ not supported). 음성 답변(Q2)은 재확인하지 않습니다.
- ④ **Decision agent**: 근거가 확인된 소견만 받아들입니다 (Pleural effusion → **Confirmed**, Pneumothorax → **Uncertain**, Cardiomegaly → **Not reported**).

**Multi-round refinement**
- 라운드 $r$에서 확정된 소견으로 라운드 $r+1$의 검색 쿼리를 다시 만듭니다 (최대 N 라운드).
- **Final report**: Yes / No / Uncertain + 영상에 근거한 서술(grounded narrative)

### Figure 2. Multi-agent pipeline

![Figure 2: Retrieved-text zone and image-grounded zone](/assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-figure-2.png)
*Figure 2: 검색 텍스트 영역(질문 생성)과 영상 근거 영역(인지·검증·결정)의 분리, 그리고 다중 라운드 개선 구조*

```
[Query image] ─▶ Report DB ─▶ Top-K reports (다른 환자)
                                   │
                                   ▼
                    ① Questioning agent  ── "Q1. Pleural effusion?  Q2. Cardiomegaly?  Q3. Pneumothorax?"
═══════════════════ Retrieved text stops here — only questions pass ═══════════════════
                                   │ (questions only)
                                   ▼
   ② Perception (image only) ─▶ ③ Verification (detector, positives only) ─▶ ④ Decision
        Q1 Yes / Q2 No / Q3 Yes      Q1 ✓ supported / Q3 ✗ not supported      Confirmed / Uncertain / Not reported
                                   │
                                   ▼
                     Final report (Yes / No / Uncertain + grounded narrative)
                                   │
                                   └──▶ confirmed findings re-query the retriever (round r+1, up to N)
```

### 2-2. 에이전트별 입력·출력과 검색 텍스트 접근 권한

| Agent | Input | Output | Retrieved text |
| :--- | :--- | :--- | :---: |
| 1. Questioning | Retrieved reports | Targeted questions | ✓ exposed |
| 2. Perception | Image + questions | Answers (visual evidence) | ✗ isolated |
| 3. Verification | Image + positive answers | Detector support | ✗ isolated |
| 4. Decision | Answers + detector support | Confirmed / uncertain | ✗ isolated |

> 영상을 읽고 최종 보고서를 작성하는 에이전트는 **다른 환자의 텍스트를 한 번도 보지 않습니다.**

### 2-3. 세 가지 설계 원칙

1. **Evidence isolation**: 질문 에이전트만 검색 텍스트를 보고, 영상 판독 및 보고 에이전트는 보지 않습니다.
2. **Image-grounded admission**: detector의 지지가 있을 때만 소견을 확정하고, 그렇지 않으면 *uncertain*으로 표시합니다.
3. **Iterative refinement**: 한 라운드에서 확정된 소견이 다음 라운드의 검색을 안내합니다.

---

## 3. 실험 설계 및 결과 (Experiments & Results)

### 3-1. 실험 설정

- **Data**: MIMIC-CXR 흉부 X-ray 검사 [3]
- **Backbone**: 모든 에이전트에 **Qwen3-VL-8B**
- **Baseline**: 같은 backbone이 **검색 없이** 영상만 판독
- **Metric**: 소견 단위(finding-level) precision, recall, F1
- **Robustness**: (i) 검색 판독문을 **무관한 환자**의 것으로 치환, (ii) **영상에 없는 소견**을 주장하는 문장 주입

### Figure 3. Finding-level agreement on MIMIC-CXR

![Figure 3: Finding-level agreement on MIMIC-CXR](/assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-figure-3.png)
*Figure 3: (a) Table 1 — 소견 단위 일치도, (b) Image-only baseline vs. proposed multi-agent framework*

| Variant | Precision | Recall | F1 |
| :--- | ---: | ---: | ---: |
| Qwen3-VL-8B, image only | 0.265 | 0.157 | 0.197 |
| **Proposed multi-agent** | **0.272** | **0.210** | **0.237** |
| Δ (Proposed − Baseline) | +0.007 | **+0.053** | **+0.040** |

- 정밀도를 유지하면서 Recall과 F1이 올랐습니다. 즉, 추가로 찾은 소견들이 **근거 없는 소견을 끌어들이지 않고** 회복되었습니다.

### Figure 4. Robustness to corrupted retrieved reports

![Figure 4: Robustness to corrupted retrieved reports](/assets/img/posts/2026-10-24-kosaim-2026/poster-cxr-multi-agent-kosaim-2026-figure-4.png)
*Figure 4: (i) 무관한 환자 판독문 치환, (ii) 거짓 소견 주입 — 두 경우 모두 성능 저하 없음*

| 오염 시나리오 | 조작 내용 | 결과 |
| :--- | :--- | :--- |
| (i) Unrelated-patient reports | 검색 판독문을 무관한 증례의 것으로 교체 | ✓ No degradation |
| (ii) Injected false finding | 영상에 없는 소견을 주장하는 문장 주입 | ✓ No degradation |

- 검색 텍스트는 질문만 만들고, 답은 영상과 detector가 결정합니다. 그래서 텍스트가 틀려도 **틀린 질문이 하나 늘어날 뿐** 틀린 소견으로 이어지지 않습니다.

---

## 4. 결론 및 의의 (Conclusion & Significance)

1. **인지(perception), 검증(verification), 결정(decision)을 분리**하고, **영상에서 확인된 소견만** 받아들이면 더 신뢰할 수 있는 판독문이 만들어집니다.
2. 정밀도 손실 없이 더 많은 실제 소견을 찾았습니다 (**Recall +0.053, F1 +0.040**).
3. **에이전트 간 근거(evidence)의 흐름을 통제하는 것**이 임상적 신뢰성의 핵심입니다.

### Take-Home Message

> **"Controlling how evidence flows across agents is central to clinical trustworthiness."**
>
> (에이전트 사이에서 근거가 어떻게 흐르는지를 통제하는 것이 임상적 신뢰성의 핵심입니다. 검색된 다른 환자의 텍스트는 "무엇을 확인할지"만 정하고, "무엇이 있다"는 판단은 오직 현재 환자의 영상으로만 내립니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. 이 포스터의 핵심 아이디어를 한 문장으로 말하면?

**"다른 환자의 판독문은 체크리스트로만 쓰고, 판정은 이 환자의 영상으로만 한다."**

RAG의 장점(놓치기 쉬운 소견을 떠올리게 함)은 살리고, 단점(다른 환자 소견을 복사함)은 정보 흐름 설계로 차단했습니다. 모델을 새로 학습시키지 않고 **파이프라인 구조만으로** 신뢰성을 높였다는 점이 특징입니다.

#### Q. F1 0.237이면 성능이 낮은 것 아닌가요?

절대값은 낮습니다. 다만 이 연구의 비교 대상은 SOTA 판독 모델이 아니라 **같은 backbone(Qwen3-VL-8B)의 영상 단독 판독**입니다. 8B 범용 VLM을 추가 학습 없이 쓴 설정이라 절대 성능보다 **같은 모델 대비 개선 폭과 강건성**을 봐야 합니다.

#### Q. 이 포스터를 비판적으로 본다면?

1. **기존 RAG와의 직접 비교가 없습니다.** 문제 제기는 "기존 RAG가 소견을 복사한다"인데, 비교 대상은 검색을 쓰지 않는 baseline뿐입니다. 기존 RAG의 precision이 실제로 얼마나 떨어지는지를 보여줘야 주장이 완성됩니다.
2. **불확실성 표시가 없습니다.** Precision +0.007이 "유지"인지 판단하려면 신뢰구간이나 검정이 필요합니다.
3. **강건성 실험의 수치가 없습니다.** "No degradation"이라는 정성적 표시만 있습니다.
4. **Verification agent의 detector가 무엇인지 명시되지 않았습니다.** 최종 판정의 상당 부분이 이 detector에 달려 있어서, detector 성능이 전체 성능의 상한이 될 수 있습니다.
5. **"Uncertain" 비율이 보고되지 않았습니다.** 확정을 보류한 소견이 많다면 임상적 유용성이 줄어듭니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- 문제 정의가 날카롭습니다. "RAG가 성능을 올린다"는 통념 뒤에 숨은 **다른 환자 텍스트 오염**이라는 위험을 짚었습니다.
- 해결책이 단순하고 설명 가능합니다. 각 에이전트의 입력과 출력, 검색 텍스트 접근 권한이 표 하나로 정리됩니다.
- 거짓 소견 주입 같은 **적대적 강건성 테스트**를 설계해, 주장(검색 텍스트가 판정에 영향을 주지 않음)을 직접 검증하려 했습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> KTAS 포스터가 "AI가 사람의 판단을 어떻게 바꾸는가"를 다뤘다면, 이 포스터는 **"검색된 정보가 AI의 판단을 어떻게 오염시키는가"**를 다룹니다. 두 연구 모두 결론이 "더 큰 모델"이 아니라 **정보가 흐르는 경로를 설계하는 것**이라는 점이 흥미롭습니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 신경과 EMR 기반 LLM 파이프라인에 "증거 격리" 원칙 적용** ⭐ 우선순위 1

- 유사 환자 기록이나 가이드라인을 RAG로 붙이는 신경과 LLM 과제(예: 퇴원 요약 작성, 뇌졸중 레지스트리 항목 추출)에서, **최종 판정 단계에는 현재 환자의 기록만** 들어가도록 파이프라인을 설계합니다.
- 다른 환자의 텍스트는 "확인할 항목 목록"을 만드는 데만 씁니다.

**B. 오염 주입(perturbation) 기반 강건성 평가 설계**

- 이 포스터의 강건성 실험을 통계적으로 확장합니다.
  - 검색 문서에 거짓 소견을 **주입하는 비율을 0%–100%로 바꿔 가며** 성능 곡선을 그리고, 기울기로 오염 민감도를 정량화합니다.
  - 증례를 무작위효과로 둔 혼합모형으로 "오염 주입 여부"의 효과를 추정합니다.
- 어떤 RAG 시스템에도 쓸 수 있는 **평가 프로토콜**로 정리할 수 있습니다.

**C. 뇌 영상 판독 보조에 Perception → Verification 구조 적용**

- 뇌 MRI/CT 판독에서 VLM이 소견을 답하면, **전용 detector**(예: 출혈, 허혈 병변, 백질 고신호 분할 모델)로 양성 소견만 재확인하는 구조입니다.
- 뇌졸중 영상처럼 검증된 분할 모델이 이미 있는 영역이라 Verification agent를 구성하기 쉽습니다.
- Confirmed / Uncertain / Not reported 3단계 출력은 임상의 검토 우선순위를 정하는 데에도 쓸 수 있습니다.

> **추천**: **A + B 조합**입니다. 연구실의 기존 EMR 텍스트 과제에 증거 격리 파이프라인을 적용하고, B의 오염 주입 실험으로 효과를 정량적으로 보여주는 구성입니다. 영상 모델 없이 텍스트만으로 시작할 수 있어 진입 장벽이 낮습니다.

---

### References (포스터 인용)

1. Li C, et al. LLaVA-Med. *NeurIPS* 2023.
2. Xia P, et al. MMed-RAG. *ICLR* 2025.
3. Johnson AEW, et al. MIMIC-CXR-JPG. *arXiv* 2019.
4. Irvin J, et al. CheXpert. *AAAI* 2019.
