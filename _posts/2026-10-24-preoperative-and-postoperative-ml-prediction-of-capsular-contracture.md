---
title: "[KoSAIM] Preoperative and Postoperative Machine Learning Prediction of Capsular Contracture after Implant-Based Breast Reconstruction — 포스터 심층 분석"
date: 2026-10-24 10:08:07 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Capsular Contracture","Breast Reconstruction","Clinical Prediction Model","Data Leakage","Out-of-Fold Prediction","L1 Logistic Regression","Decision Curve Analysis","TRIPOD+AI","KoSAIM 2026"]
description: "유방 보형물 재건 후 구형구축(CC)의 가장 강한 위험 인자는 유방절제술 후 방사선치료(PMRT)인데, 수술 전에는 PMRT 여부를 알 수 없습니다. 이 연구는 먼저 PMRT 확률을 예측하는 모델을 만들고, 정보 누설이 없도록 out-of-fold 방식으로 그 확률을 수술 전 CC 예측 모델에 연결(bridging)했습니다. 1,001개 유방에서 수술 전 모델 AUROC 0.719, 수술 후 모델 0.778을 얻었고, 두 모델 모두 고위험인 경우 CC 발생률이 27.8%로 둘 다 저위험(3.0%)보다 9배 높았습니다. CC가 생긴 환자는 BREAST-Q 유방 만족도가 유의하게 낮았습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-full.jpg
  alt: "[KoSAIM] Preoperative and Postoperative ML Prediction of Capsular Contracture — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Preoperative and postoperative machine learning prediction of capsular contracture after implant-based breast reconstruction
- **부제**: Leakage-controlled bridging of anticipated post-mastectomy radiotherapy
- **포스터 번호**: P4-10 — **Poster Award (Certificate of Merit)** 수상
- **저자 및 소속**: **Jae Heon Jeong**¹·²·⁵\*, **Joseph Kyu-hyung Park**²\*, Yujin Myung², Jae Hoon Jeong², Hee-Chul Shin³, Kyung-Hwak Yoon³, Jung Hee Shim¹·²·⁵, Minsung Park¹·⁴, Ji Min Lee², Jin-Woo Kim², Sodam Yi², **Eun-Kyu Kim**³#, **Chan Yeong Heo**²·⁵# (\*Co-first authors, #Co-corresponding authors)
  - ¹분당서울대학교병원 의생명연구원, ²분당서울대병원·서울대 의대 성형외과, ³분당서울대병원·서울대 의대 외과, ⁴서울대 융합과학기술대학원 헬스사이언스학과, ⁵H&BIO Corporation R&D Center
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **출판 상태**: *Computers in Biology and Medicine* (under revision)
- **핵심 키워드**: `Capsular Contracture`, `Breast Reconstruction`, `Clinical Prediction Model`, `Data Leakage`, `Out-of-Fold Prediction`, `L1 Logistic Regression`, `Decision Curve Analysis`, `TRIPOD+AI`

![Capsular contracture prediction 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Preoperative and postoperative machine learning prediction of capsular contracture after implant-based breast reconstruction*

> **Executive Summary**  
> 유방 보형물 재건 후 **구형구축(capsular contracture, CC)**의 가장 일관된 위험 인자는 **유방절제술 후 방사선치료(PMRT)**입니다. 그런데 PMRT 시행 여부는 최종 병리 결과를 봐야 결정되므로 **수술 전에는 알 수 없습니다**. 이 연구는 2단계 구조로 이 문제를 풀었습니다.
> 1. **Stage 1**: 수술 전 정보로 **PMRT 확률을 예측**하는 모델을 만듭니다.
> 2. **Stage 2**: 그 확률을 **out-of-fold 방식으로**(자기 데이터가 자기 확률 계산에 쓰이지 않게) 수술 전 CC 모델에 넣습니다.
>
> 1,001개 유방(864명)에서 **수술 전 AUROC 0.719, 수술 후 AUROC 0.778**을 얻었습니다. 두 모델 모두 고위험인 경우 CC 발생률은 **27.8%**로, 둘 다 저위험인 경우(**3.0%**)보다 **9배** 높았습니다.

---

## 1. 연구 배경 (Background)

### 1-1. 핵심 수치 (Key numbers)

| 항목 | 값 |
| :--- | :--- |
| 코호트 | **1,001개 유방 (864명)**, CC 사건 103건 (Baker ≥ III, 10.3%) |
| 수술 전 모델 AUROC | **0.719** (95% CI 0.668–0.768) |
| 수술 후 모델 AUROC | **0.778** (95% CI 0.726–0.834) |
| 관찰된 CC 발생률 | 두 모델 모두 고위험 **27.8%** vs 두 모델 모두 저위험 **3.0%** |

### 1-2. 임상적 배경

- **구형구축(CC)**은 보형물 기반 재건의 흔하고 중대한 합병증입니다. 재수술을 해도 **재발**하는 경우가 많아, 위험은 **첫 재건 전에** 예측하는 것이 가장 좋습니다.
- **PMRT**는 가장 일관된 위험 인자입니다. 하지만 PMRT 시행 여부는 최종 병리 결과에 따라 결정되므로, **수술 전 결정 시점에는 알 수 없습니다**.
- 따라서 **실제 PMRT 정보가 필요한 모델은 수술 전 상담에 쓸 수 없습니다.**

> **KEY QUESTION**  
> 아직 관찰되지 않은 지배적 노출(PMRT)을 **미래 정보 누설 없이** 수술 전 예측에 넣을 수 있는가?

---

## 2. 연구 방법 (Methods)

- **코호트**: 단일 기관(분당서울대병원), ADM을 사용한 즉시 direct-to-implant 재건, 2018년 11월–2024년 12월, 추적 기간 중앙값 29.9개월
- **결과 변수**: Baker grade ≥ III. 유방외과와 성형외과가 **일치하여 기록**한 경우 (κ = 0.852)
- **2단계 프레임워크**
  - **Stage 1**: PMRT 예측 모델 (1,088개 유방, AUROC 0.751)
  - **Out-of-fold bridging**: 어떤 환자의 데이터도 **그 환자 자신의 PMRT 확률** 계산에 쓰이지 않습니다.
  - **Stage 2**: **수술 전** CC 모델과 **수술 후** CC 모델
- **모델링**: 분류기 7종, **10 × 5 nested CV**(환자 단위 그룹화), **L1 로지스틱 회귀** 최종 선택
- **평가와 보고**: 보정(calibration), 결정곡선분석(DCA), SHAP, **TRIPOD+AI** 지침 준수

### 최종 모델의 예측 변수 (sparse L1)

| 수술 전 모델 | 수술 후 모델 |
| :--- | :--- |
| **Bridged PMRT probability** (후보 31개 중 유일하게 0이 아닌 계수) | 실제 PMRT 시행 여부 · 절제 림프절 수 · 수술 후 NLR · 병리 T 병기 |

### Figure 1. 코호트 구성과 모델 파이프라인

![Figure 1: Cohort derivation and two-stage pipeline](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-figure-1.png)
*Figure 1: (a) 코호트 구성, (b) out-of-fold PMRT bridging을 포함한 2단계 누설 방지 파이프라인*

- **PMRT 코호트**: 1,088개 유방 (944명), PMRT 311건 (28.6%)
- **CC 코호트**: 1,001개 유방 (864명), CC 103건 (Baker ≥ III, 10.3%)

```
[Stage 1] PMRT 예측 모델 (수술 전 변수만 사용)
              │
              ▼
[OOF Bridging] CC 코호트를 fold로 나눔
   ─ fold k의 PMRT 확률은 fold k를 뺀 나머지로 학습한 모델이 계산
   ─ 어떤 환자도 자기 자신의 PMRT 확률 계산에 기여하지 않음
              │
              ▼
[Stage 2] 수술 전 CC 모델 (bridged PMRT probability 포함)
          수술 후 CC 모델 (실제 PMRT + 병리 정보)
```

---

## 3. 실험 결과 (Results)

### 3-1. 수술 전 bridging이 신호를 더함

| 지표 | 값 |
| :--- | :--- |
| Bridged PMRT 확률을 더한 AUROC 향상 | **+0.033** (0.686 → 0.719, 단측 P = 0.006) |
| 실제 PMRT를 넣었을 때 향상(+0.055) 대비 회복 비율 | **약 60%** (PMRT를 알기 **전에** 회복) |
| 누설 부풀림 여부 | 없음: fully nested refit 0.716 vs 0.718 (P = 0.600) |
| 보정 | slope **0.98 / 0.99** (수술 전 / 수술 후) |
| 순편익 | 임계값 0.05–0.30에서 treat-all과 treat-none보다 높음 |

### Figure 2. 수술 전·수술 후 모델의 ROC, 보정, 결정곡선

![Figure 2: ROC, calibration, decision curve analysis](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-figure-2.png)
*Figure 2: 수술 전(왼쪽)과 수술 후(오른쪽) 모델의 (a, b) ROC, (c, d) 보정 곡선, (e, f) 결정곡선분석*

- ROC: 수술 전 PMRT 확률 포함 AUC 0.719 [0.668–0.768] vs 미포함 0.686 [0.630–0.740]; 수술 후 0.778 [0.726–0.834]
- DCA에서 표시된 운영 임계값은 수술 전 t = 0.09, 수술 후 t = 0.11입니다 (민감도 ≥ 0.70 기준).

### 3-2. 두 시점, 상호보완적 위험

- 수술 전 점수와 수술 후 점수는 **중간 정도로만 상관**되었습니다 (r = 0.47).
- 두 모델 모두 고위험: CC **27.8%** vs 두 모델 모두 저위험: CC **3.0%** → **9배 차이**

### Figure 3. 예측 위험 구간별 관찰 CC 발생률과 수술 전·후 위험의 관계

![Figure 3: Observed CC rate by predicted-risk bin and paired risk](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-figure-3.png)
*Figure 3: (a) 예측 확률 구간별 관찰 CC 발생률(점선: 전체 유병률 10.3%), (b) 같은 유방의 수술 전 vs 수술 후 예측 위험*

- (b)의 네 구역: 둘 다 고위험 27.8%, 수술 후만 고위험 23.8%, 수술 전만 고위험 17.9%, 둘 다 저위험 3.0%
  - <!-- TODO: 그림 범례 수치(23.8%, 17.9%)가 작아 판독이 불확실함, 확인 필요 -->

### 3-3. 각 시점의 위험을 이끄는 요인

- **수술 전**: bridged PMRT 확률이 **유일한** 비영(non-zero) 예측 변수였습니다. 즉, 수술 전 위험 ≈ **예상되는 PMRT**입니다.
- **수술 후**: PMRT, 절제 림프절 수, 수술 후 NLR, 병리 T 병기. 방사선치료 **이상의 정보**를 더합니다 (PMRT 단독 0.705 vs 전체 0.778, P = 0.002).

### Figure 4. SHAP 요약

![Figure 4: SHAP summary](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-figure-4.png)
*Figure 4: (a) 수술 전 모델과 (b) 수술 후 모델의 SHAP 요약, (c) 수술 후 모델의 유방별 SHAP heatmap*

### 3-4. CC는 환자에게 중요한 결과임

- BREAST-Q 응답: 864명 중 707명 (81.8%), PMRT·나이·BMI·설문 시점으로 보정

| BREAST-Q 영역 | CC 양성 − CC 음성 (보정 평균 차이) | P |
| :--- | ---: | ---: |
| 유방 만족도 (Breast satisfaction) | **−0.38** (−0.55 to −0.21) | < 0.001 |
| 성적 안녕 (Sexual well-being) | **−0.30** (−0.56 to −0.03) | 0.027 |
| 유방 만족도 (덜 만족한 쪽) | −0.24 (−0.48 to −0.00) | 0.050 |
| 흉부 증상 (Chest symptoms, 높을수록 나쁨) | **+0.16** (+0.05 to +0.27) | 0.003 |
| 흉부 심리사회적 안녕 | −0.14 (−0.39 to +0.11) | 0.268 |

*하단 세 행의 신뢰구간은 사진 판독이 어려워 근사값입니다.*

### Figure 5. CC 여부에 따른 수술 후 BREAST-Q 점수

![Figure 5: Postoperative BREAST-Q scores by CC status](/assets/img/posts/2026-10-24-kosaim-2026/poster-capsular-contracture-kosaim-2026-figure-5.png)
*Figure 5: (a) CC 여부별 BREAST-Q 점수, (b) 보정 평균 차이 (CC 양성 − CC 음성)*

---

## 4. 결론 (Conclusions)

1. **누설 없는 bridging** 절차로, 아직 관찰되지 않은 지배적 노출(PMRT)을 **수술 전** CC 예측에 넣을 수 있습니다.
2. **수술 전 위험은 예상 PMRT가 이끌고**, 수술 후 모델은 방사선치료 이상의 정보를 더합니다.
3. **두 시점별 모델**은 수술 전 상담과 수술 후 위험 기반 추적 관찰을 각각 지원합니다.
4. **한계**: 단일 기관, 후향적 연구, 내부 검증만 수행 → 외부 검증 필요

### Take-Home Message

> **"A leakage-safe bridging procedure lets a dominant but not-yet-observed exposure enter a preoperative prediction."**
>
> (가장 중요한 위험 인자를 수술 전에는 알 수 없다면, 그 인자 자체를 예측해서 넣으면 됩니다. 단, 정보가 새지 않도록 out-of-fold로 해야 합니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. "Bridging"과 "누설(leakage)"이 무슨 뜻인가요?

**Bridging**은 "수술 전에는 모르는 변수(PMRT)를 수술 전 정보로 예측한 확률로 대신 넣는 것"입니다. 문제는 그 확률을 어떻게 계산하느냐입니다.

- **잘못된 방법**: 전체 데이터로 PMRT 모델을 학습한 뒤, 같은 환자들에게 확률을 계산해서 CC 모델에 넣습니다. 그러면 각 환자의 PMRT 확률에 **그 환자의 실제 PMRT 정보가 이미 섞여** 있습니다. CC 모델은 사실상 실제 PMRT를 간접적으로 보게 되고, 성능이 부풀려집니다.
- **이 연구의 방법(out-of-fold)**: 데이터를 fold로 나누고, 각 fold의 PMRT 확률은 **그 fold를 제외한 데이터로 학습한 모델**로 계산합니다. 어떤 환자도 자기 확률 계산에 기여하지 않습니다.
- 저자들은 누설이 없다는 것을 **fully nested refit**(CV 안쪽에서 PMRT 모델까지 다시 학습)과 비교해 보였습니다 (0.716 vs 0.718, P = 0.600).

이는 통계학에서 **stacking**이나 **cross-fitting**과 같은 원리입니다.

#### Q. 통계적으로 잘한 점은?

1. **결과 변수의 신뢰도 확인**: Baker 등급을 두 과가 일치하여 기록한 경우만 썼고, 일치도(κ = 0.852)를 보고했습니다.
2. **환자 단위 그룹화**: 한 환자의 양쪽 유방이 서로 다른 fold에 들어가지 않게 했습니다. 양측 재건 환자가 있으므로(1,001개 유방, 864명) 필수적인 처리입니다.
3. **Nested CV**로 모델 선택과 성능 평가를 분리했습니다.
4. **복잡한 모델보다 해석 가능한 모델**: 7종을 비교한 뒤 L1 로지스틱 회귀를 골랐고, 그 결과 수술 전 모델은 변수 1개만 남았습니다.
5. **판별력 외 지표**: 보정(slope 0.98/0.99)과 DCA로 실제 임상 유용성까지 확인했습니다.
6. **환자 보고 결과(PROM)**: CC가 BREAST-Q 점수를 실제로 낮춘다는 것을 보여, "왜 이걸 예측해야 하는가"에 답했습니다.
7. **TRIPOD+AI** 보고 지침을 따랐습니다.

#### Q. 비판적으로 본다면?

1. **수술 전 모델 = PMRT 예측 모델?** 수술 전 모델에서 유일하게 남은 변수가 bridged PMRT 확률입니다. 그렇다면 수술 전 CC 모델은 사실상 **"PMRT를 받을 확률"을 다시 보정한 것**입니다. 다른 수술 전 변수(보형물 크기, BMI, 흡연 등)는 PMRT를 넘어서는 정보를 주지 못했다는 뜻이기도 합니다.
2. **단측 검정**: AUROC 향상의 P = 0.006은 단측입니다. 방향성 가설이 사전에 정해졌다면 타당하지만, 양측이었다면 약 0.012입니다.
3. **경쟁 위험과 추적 기간**: 추적 기간 중앙값 29.9개월인데 CC는 시간이 지나며 늘어나는 합병증입니다. 이진 결과 대신 **생존 분석(time-to-event)**으로 보거나, 추적 기간이 짧은 환자의 처리 방식을 명시하면 더 좋았을 것입니다.
4. **단일 기관, 내부 검증만**: 저자들도 외부 검증이 필요하다고 밝혔습니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- **임상 결정 시점에 맞춘 문제 설정**: "수술 전에 알 수 있는 정보만으로"라는 현실적 제약을 정면으로 다뤘습니다.
- **방법론적으로 정교한 누설 방지**: 흔히 놓치는 정보 누설 문제를 설계 단계에서 막고, 그것을 검증까지 했습니다.
- **임상 예측 모델의 모범적 보고**: 보정, DCA, PROM, TRIPOD+AI까지 갖췄습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 딥러닝 없이 **L1 로지스틱 회귀 하나**로 우수 포스터를 받은 연구입니다. 모델의 복잡도가 아니라 **"언제, 어떤 정보로 예측하는가"라는 설계**가 핵심이었습니다. 수술 전 모델에 변수가 하나만 남았다는 결과도, 숨기지 않고 "수술 전 위험 ≈ 예상 PMRT"라는 임상적 메시지로 정직하게 해석했습니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. "결정 시점에 아직 모르는 핵심 변수"의 bridging** ⭐ 우선순위 1

신경과에도 같은 구조의 문제가 많습니다. 결정 시점에 가장 중요한 정보를 아직 모르는 경우입니다.

| 결정 시점 | 아직 모르는 핵심 변수 | 예측 대상 |
| :--- | :--- | :--- |
| 응급실 도착 직후 | 혈관 재개통 성공 여부 | 뇌졸중 90일 기능 결과(mRS) |
| 입원 초기 | 최종 원인 분류(TOAST 아형) | 재발 위험 |
| 첫 항경련제 시작 시 | 약물 반응 여부 | 약물 난치성 뇌전증 진행 |

- 1단계로 "아직 모르는 변수"를 예측하고, 2단계에서 그 확률을 **out-of-fold로** 넣는 이 포스터의 틀을 그대로 쓸 수 있습니다.
- 통계/AI 연구원의 강점(cross-fitting, nested CV, 누설 검증)이 그대로 드러나는 과제입니다.

**B. 시점별 2단계 예측 모델**

- 이 포스터의 "수술 전 모델 + 수술 후 모델"처럼, 신경과에서도 **입원 시 모델 + 퇴원 시 모델**을 따로 만들어 각 시점의 의사결정(치료 방침 상담 vs 추적 관찰 강도)에 맞출 수 있습니다.
- 두 점수의 조합(둘 다 고위험 vs 둘 다 저위험)으로 위험군을 나누는 방식도 그대로 활용할 수 있습니다.

**C. 연구실 예측 모델 프로젝트의 누설 점검 체크리스트**

- 다단계 모델, 결측 대체, 변수 선택, 정규화 같은 전처리를 **CV 바깥에서** 하면 생기는 누설을 점검하는 표준 절차를 만듭니다.
- 이 포스터처럼 "fully nested refit과의 비교"를 누설 검증의 기본 실험으로 넣습니다.

> **추천**: **A**를 연구 과제로 추진합니다. 예를 들어 "응급실 도착 시점에 재개통 성공 확률을 bridging하여 90일 mRS를 예측"하는 연구는 이 포스터의 방법론을 신경과 핵심 질환에 직접 옮기는 것이고, 임상적 의미도 큽니다.

---

### References

포스터에 별도 참고문헌 목록은 보이지 않았습니다. 논문은 *Computers in Biology and Medicine*에서 심사 중(under revision)으로 표기되어 있습니다.
