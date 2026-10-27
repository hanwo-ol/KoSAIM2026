---
title: "[KoSAIM] Preoperative Prediction of Lung Metastasis in Soft-Tissue Sarcoma Using Foundation-Model Analysis of FDG-PET and MRI — 포스터 심층 분석"
date: 2026-10-24 10:08:14 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Soft-Tissue Sarcoma","Lung Metastasis","FDG-PET","MRI","Foundation Model","Frozen Encoder","LOOCV","Survival Analysis","KoSAIM 2026"]
description: "연부조직육종(STS)은 주로 폐로 재발하는데, 기존 위험 지표인 조직학적 등급은 폐전이를 잘 구별하지 못합니다(AUC 0.634). 이 연구는 공개 TCIA 코호트 51명의 치료 전 FDG-PET과 MRI를 고정된(frozen) 파운데이션 모델 인코더(CT-FM, Merlin)로 임베딩하고 단순 로지스틱 회귀로 예측해, PET+MRI AUC 0.913을 얻었습니다. 영상 고위험군의 폐전이 위험비는 16.1로 등급(1.9, 비유의)보다 훨씬 컸고, 위험을 이끈 것은 종양의 대사 이질성(SUV heterogeneity)이었습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-full.jpg
  alt: "[KoSAIM] Preoperative Prediction of Lung Metastasis in Soft-Tissue Sarcoma — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Preoperative Prediction of Lung Metastasis in Soft-Tissue Sarcoma Using Foundation-Model Analysis of FDG-PET and MRI: Beyond Histologic Grade
  - <!-- TODO: 리본에 가려진 제목 일부("of Lung", "Analysis of FDG")는 문맥으로 추정, 확인 필요 -->
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: **Se Hyun Lee**¹, Ji-yune…², Jun-S… Kwon², Yeon-Hee Han³, **Young-Jae Moon**²·⁴\*, **Hyun Ho Kim**⁵\* (\*Corresponding authors)
  - ¹전북대학교 의과대학, ²전북대학교병원 정형외과, ³전북대학교병원 핵의학과, ⁴전북대학교 의과대학 생화학교실, ⁵가톨릭대학교 서울성모병원 소아청소년과
  - <!-- TODO: 리본에 가려진 공저자 2명(Ji-yune…, Jun-S… Kwon)의 이름 확인 필요 -->
- **연구 지원**: 한국연구재단(NRF) 과학기술정보통신부 (RS-2025-00553891), 2026 보건장학회(Bogeon Scholarship Foundation) 연구비
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `Soft-Tissue Sarcoma`, `Lung Metastasis`, `FDG-PET`, `MRI`, `Foundation Model`, `Frozen Encoder`, `LOOCV`, `Survival Analysis`

![STS lung metastasis prediction 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Preoperative Prediction of Lung Metastasis in Soft-Tissue Sarcoma Using Foundation-Model Analysis of FDG-PET and MRI*

> **Executive Summary**  
> 연부조직육종(STS)은 주로 **폐로 재발**하고, 전이 후 생존 기간 중앙값은 11–25개월에 불과합니다. 그런데 기존 위험 지표인 **조직학적 등급**은 폐전이를 잘 구별하지 못합니다 (AUC 0.634). 이 연구는 공개 TCIA 코호트 51명의 **치료 전 FDG-PET과 MRI**를 **학습시키지 않은(frozen) 파운데이션 모델 인코더**(PET: CT-FM, MRI: Merlin)로 임베딩하고, 단순 로지스틱 회귀로 예측했습니다. **PET+MRI AUC 0.913**으로 등급(0.634)보다 크게 높았고(p < 0.001), 영상 고위험군의 폐전이 위험비(HR)는 **16.1**로 등급(1.9, 비유의)보다 훨씬 컸습니다. 위험을 가장 크게 이끈 것은 **종양의 대사 이질성(SUV heterogeneity)**이었습니다.

---

## 1. 연구 배경 (Introduction)

### 1-1. 육종은 주로 폐로 재발하고 예후가 나쁨

- **연부조직육종(STS)**은 드문 결합조직 암이며, 재발은 대부분 **폐**에서 일어납니다 [1].
- 전이 후 생존 기간 중앙값: **11–25개월** [2]
- 항암화학요법은 **고위험 환자**에게 권고되며, 가능하면 수술 전(neoadjuvant)에 시행합니다 [1].

### 1-2. 등급과 임상 변수의 위험 분류는 부정확함

- **조직학적 등급** [3]: AUC **0.634**, 생존 곡선 분리 없음 (p = 0.21)
- **모든 임상 변수**: AUC 0.678. 라디오믹스는 사람이 설계한(hand-crafted) 특징이 필요합니다 [4].

### 1-3. 목표: 치료 전 PET/MRI로 위험 예측

- 전이가 생기기 **전에** 찍은 PET/MRI로 **향후 폐전이 위험**을 예측하여, 항암치료와 폐 추적 관찰을 더 선별적으로 시행할 수 있게 하는 것

---

## 2. 연구 방법 (Methods)

### 2-1. 데이터: 공개 TCIA 연부조직육종 코호트

- **TCIA(The Cancer Imaging Archive)**: n = 51, 폐전이 19명. **등급이 있는 환자 48명** (전이 18명, 37.5%)
- 치료 전 **FDG-PET과 MRI(T1, T2/STIR)**, 종양 중심 **80 mm 정육면체** 영역

### Table 1. 폐전이 여부에 따른 코호트 특성 (n = 51; 등급은 48명)

| 특성 | 폐전이 없음 (n = 32) | 폐전이 있음 (n = 19) | p |
| :--- | :--- | :--- | ---: |
| 나이, 중앙값 (IQR) | 51.0 (34.0–62.0) | 63.0 (57.0–68.5) | **0.019** |
| 여성 | 20 (62%) | 7 (37%) | 0.091 |
| 등급 1 / 2 / 3 | 5 / 10 / 15 | 0 / 5 / 13 | 0.131 |
| 사지 발생 | 26 (81%) | 15 (79%) | 1.000 |
| 종양 직경, mm | 82 (53–101) | 93 (76–113) | **0.023** |
| SUVmax | 7.7 (5.9–11.0) | 13.4 (10.4–20.6) | **0.001** |
| SUV 이질성 (SUVsd) | 1.1 (0.7–2.0) | 2.5 (1.8–3.5) | **0.002** |

*연속형: 중앙값(IQR), Mann–Whitney U; 범주형: n(%), Fisher's exact 또는 χ². 굵은 값: p < 0.05*

### 2-2. 모델: 고정된 파운데이션 모델 + 로지스틱 회귀

### Figure 1. 파이프라인

![Figure 1: Frozen foundation-model pipeline](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-figure-1.png)
*Figure 1: 80 mm 종양 중심 PET/MRI 큐브 → 고정된 CT-FM(PET)과 Merlin(MRI) 인코더 → 특징 축소(PET은 변수 선택, MRI는 PCA) → 로지스틱 회귀 → 고정 가중 평균으로 late fusion*

```
FDG-PET ──▶ CT-FM encoder (frozen) ──▶ PET embedding (992-d) ──▶ classifier head ──▶ PET risk score ─┐
                                                                                                   ├─▶ weighted average (0.5/0.5) ──▶ 폐전이 위험
T1, T2/STIR MRI ──▶ Merlin encoder (frozen) ──▶ MRI embedding (4096-d) ──▶ classifier head ──▶ MRI risk score ─┘
                                                                                          (Leak-free LOOCV, n = 51)
```

- **고정된(frozen) 인코더**: PET은 **CT-FM** [5], MRI는 **Merlin** [6]. 인코더는 학습시키지 않고 특징만 뽑습니다.
- **Late fusion**: PET 점수와 MRI 점수를 0.5/0.5로 평균
- **누설 없는 LOOCV**(leave-one-out 교차검증). 6개 모델, 미세조정(fine-tuning), 라디오믹스와 비교

### 2-3. 평가: 다섯 가지 분석

| 분석 | 도구 | 질문 |
| :--- | :--- | :--- |
| 1. 판별력 | AUC, NRI/IDI, SHAP | 영상이 등급보다 나은가? |
| 2. 강건성 | 하위군, 시간 간격 | 시점과 항암치료에 강건한가? |
| 3. 생존 | Kaplan–Meier, Cox | 위험군별 생존이 다른가? |
| 4. 등급 vs 영상 | 환자 흐름(patient flow) | 둘은 어디서 다른가? |
| 5. 해석 가능성 | Occlusion map | 무엇이 위험을 이끄는가? |

---

## 3. 실험 결과 (Results)

### 3-1. PET+MRI가 조직학적 등급보다 우수 (AUC 0.913 vs 0.634)

### Figure 2. 판별력과 SHAP

![Figure 2: AUC comparison and SHAP](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-figure-2.png)
*Figure 2: (A) 등급, 전체 임상 변수, PET, PET+MRI의 AUC (leave-one-out, n = 48). (B) PET 대사 지표의 SHAP 값 (점 하나 = 환자 한 명)*

| 모델 | AUC (LOO, n = 48) |
| :--- | ---: |
| 조직학적 등급 | 0.634 |
| 전체 임상 변수 | 0.678 |
| **PET** | **0.872** |
| **PET + MRI** | **0.913** |

- PET 0.872, PET+MRI 0.913 vs 등급 0.634 (**p < 0.001**, DeLong)
- **NRI +1.27, IDI +0.31**. PET에 MRI를 더한 효과는 유의하지 않았습니다 (p = 0.22).
- **SHAP**: **SUV 이질성**이 가장 중요했고, 등급이 가장 덜 중요했습니다.

### 3-2. 영상은 전이보다 먼저였고, 항암치료 없이도 AUC 0.91–0.96

- 영상 촬영 시점은 폐전이보다 **최소 62일** 앞섰습니다 (중앙값 181일). 즉, 이미 생긴 전이를 본 것이 아닙니다.
- **항암치료를 받지 않은 하위군**(n = 30): PET과 PET+MRI 모델의 AUC **0.912–0.960**

### 3-3. 영상 위험군은 생존을 가르고, 등급은 가르지 못함

### Figure 3. 위험군별 무폐전이 생존 (n = 48)

![Figure 3: Lung-metastasis-free survival by risk group](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-figure-3.png)
*Figure 3: 조직학적 등급, 임상 모델, PET 모델, PET+MRI 모델별 무폐전이 생존 곡선과 log-rank p, HR(95% CI), C-index*

| 위험 분류 기준 | HR (95% CI) | p | 폐전이율 (저위험 vs 고위험) | C-index |
| :--- | :--- | :--- | :--- | :--- |
| 조직학적 등급 | 1.9 (0.7–5.4) | 0.211 | 25% vs 46% | — |
| **PET 모델** | **12.1 (3.4–42.2)** | < 0.001 | **10% vs 79%** | 0.775 |
| **PET+MRI 모델** | **16.1 (4.6–56.6)** | < 0.001 | **10% vs 88%** | 0.784 |

### 3-4. PET은 등급과 불일치한 환자 20명 중 15명을 올바르게 재분류

### Figure 4. 등급 → 영상 위험군 → 실제 폐전이

![Figure 4: Grade to imaging risk group to observed metastasis](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-figure-4.png)
*Figure 4: 왼쪽 막대는 조직학적 등급, 가운데는 모델 위험군, 오른쪽은 실제 결과 (n = 48). (A) PET, (B) PET+MRI*

- **고등급인데 전이가 없었던 15명 중 12명**을 PET은 저위험으로 분류했습니다.
- **중간/저등급인데 전이가 생긴 5명 중 3명**을 PET은 고위험으로 분류했습니다.
- 즉, 등급과 실제 결과가 어긋난 20명 중 **15명**을 PET이 올바르게 재분류했습니다.

### 3-5. Occlusion map은 차가운 영역이 아닌 고대사 영역을 강조

### Figure 5. Occlusion map

![Figure 5: Occlusion maps](/assets/img/posts/2026-10-24-kosaim-2026/poster-sts-lung-metastasis-kosaim-2026-figure-5.png)
*Figure 5: 전이 양성 3례, 음성 3례의 occlusion map (fold 모델). 각 패널은 자기 최댓값으로 정규화되어 환자 간 색 비교는 불가. 청록색 = 종양 윤곽*

- **고대사(hot) 영역과 경계(boundary)**에서 중요도가 높았습니다 (1.38 / 1.29 vs 저대사 영역 0.80).
- **전이 양성**에서 최대 중요도가 더 높았습니다 (0.016 vs 0.006, p = 0.019). 위험 점수와 상관 ρ = 0.68
- **해석**: 해당작용(glycolysis)과 저산소(hypoxia)가 **혈행성 폐전이**를 촉진할 수 있습니다.

---

## 4. 논의 및 결론 (Discussion)

1. **PET 고위험 환자 19명 중 15명**에서 폐전이가 생겼고, 저위험 29명 중에서는 **3명**뿐이었습니다. 영상 위험도는 항암치료와 폐 추적 관찰 결정에 도움을 줄 수 있습니다.
2. PET+MRI(AUC 0.913)는 **조직학적 등급(0.634)보다 우수**했고, PET은 고등급·비전이 환자 15명 중 12명을 저위험으로 분류했습니다.
3. **미세조정 없이** 고정된 파운데이션 모델 인코더와 단순한 헤드만으로 AUC 0.87–0.91에 도달했고, **SUV 이질성**이 예측 위험을 이끌었습니다.
4. **한계**: 단일 공개 코호트(n = 51), 임상적 이득은 검증되지 않음. 다음 단계로 **전북대병원(JBNU) 검증**과 **병리 분석**이 진행 중입니다.

### Take-Home Message

> **"Before treatment, PET/MRI foundation-model analysis classified lung-metastasis risk better than histologic grade (driven by metabolic heterogeneity) and may guide chemotherapy and lung surveillance once prospectively validated."**
>
> (치료 전 PET/MRI를 파운데이션 모델로 분석하면 조직학적 등급보다 폐전이 위험을 더 잘 분류할 수 있고, 그 핵심은 종양의 대사 이질성입니다. 전향적 검증을 거치면 항암치료와 폐 추적 관찰 결정에 쓰일 수 있습니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. 왜 파운데이션 모델을 "학습시키지 않고(frozen)" 썼나요?

환자가 **51명**뿐이기 때문입니다. 수백만 개 파라미터를 가진 인코더를 51명으로 미세조정하면 과적합이 거의 확실합니다.

- 대신 대규모 데이터로 미리 학습된 인코더(CT-FM, Merlin)를 **특징 추출기**로만 쓰고, 그 위에 **로지스틱 회귀**만 학습했습니다. 학습할 파라미터 수를 극단적으로 줄인 설계입니다.
- 포스터에 따르면 미세조정, 라디오믹스와도 비교했고, 고정 인코더 + 단순 헤드가 AUC 0.87–0.91에 도달했습니다.
- 소규모 의료 데이터에서 파운데이션 모델을 쓰는 **현실적인 방법**을 보여준 사례입니다.

#### Q. 통계적으로 잘한 점은?

1. **누설 없는 LOOCV**: 표본이 작을 때 가능한 한 많은 데이터를 학습에 쓰면서, 특징 선택과 PCA를 **각 fold 안에서** 수행해 정보 누설을 막았습니다.
2. **판별력 외 다양한 지표**: AUC 외에 NRI/IDI(재분류 개선), Kaplan–Meier, Cox HR, C-index까지 보고했습니다.
3. **역인과 배제**: 영상이 전이보다 최소 62일 앞섰음을 보여, "이미 생긴 미세 전이를 본 것 아니냐"는 반론에 답했습니다.
4. **교란 요인 점검**: 항암치료가 결과를 바꿀 수 있으므로, 항암치료를 받지 않은 하위군에서 따로 분석했습니다.
5. **불일치 사례 분석**: 등급과 영상이 다른 환자를 따로 추적해 "어디서 더 나은지"를 구체적으로 보였습니다.

#### Q. 비판적으로 본다면?

1. **매우 작은 표본**: n = 48–51, 사건 19건. 신뢰구간이 매우 넓습니다 (HR 16.1의 95% CI가 4.6–56.6). 저자들도 외부 검증이 필요하다고 명시했습니다.
2. **LOOCV의 분산**: 표본이 작을 때 LOOCV 추정치는 분산이 큽니다. 반복 k-fold나 부트스트랩 신뢰구간이 함께 있으면 더 좋았을 것입니다. 또 Figure 2의 PET, PET+MRI 막대에는 오차 막대가 없습니다.
3. **위험군 임계값**: 고위험/저위험을 나눈 임계값을 어떻게 정했는지(사전 고정인지, 데이터 기반인지)가 포스터에 나와 있지 않습니다. 같은 데이터로 임계값을 정하고 HR을 계산하면 HR이 부풀려질 수 있습니다.
4. **NRI +1.27**: 범주 없는(continuous) NRI는 최댓값이 2이고, 작은 표본에서 쉽게 커지는 지표라 해석에 주의가 필요합니다.
5. **교란**: 전이군이 나이가 많고 종양이 큽니다 (Table 1). 영상 점수가 나이·크기와 독립적으로 예측력을 가지는지 다변량 Cox 분석이 있으면 좋았을 것입니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- **명확한 임상 질문**: "현재 표준(조직학적 등급)보다 나은가?"라는 비교 대상이 분명합니다.
- **소규모 데이터에 맞는 설계**: 고정 인코더 + 단순 모델로 과적합을 피했습니다.
- **다각도 검증**: 판별력, 강건성, 생존, 재분류, 해석 가능성의 다섯 분석으로 결과를 입체적으로 보였습니다.
- **생물학적 해석**: 대사 이질성 → 해당·저산소 → 혈행성 전이로 이어지는 기전 가설을 제시했습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 51명이라는 작은 공개 데이터로도 설득력 있는 연구를 만들 수 있다는 점을 보여준 포스터입니다. 핵심은 **큰 모델을 학습시키는 것이 아니라, 이미 학습된 큰 모델을 "특징 추출기"로만 쓰고 통계 모형은 단순하게 유지**한 설계였습니다. 다만 HR 신뢰구간이 4.6–56.6일 만큼 넓어, 외부 검증 결과가 나와야 진짜 성능을 알 수 있습니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 소규모 신경과 영상 코호트에 "고정 인코더 + 단순 모델" 적용** ⭐ 우선순위 1

- 희귀 신경질환(예: 자가면역 뇌염, 다발성 경화증 아형, 희귀 운동질환)은 환자 수가 적어 딥러닝 학습이 어렵습니다.
- 뇌 MRI용 파운데이션 모델을 **고정된 특징 추출기**로 쓰고, 로지스틱 회귀나 Cox 모델만 학습하는 이 포스터의 설계를 그대로 적용할 수 있습니다.
- 예: 뇌졸중 환자의 치료 전 MRI로 3개월 기능 결과 예측, 치매 전단계 환자의 MRI로 전환 위험 예측

**B. 기존 임상 지표와의 비교 설계**

- 이 포스터처럼 **현재 표준 지표**(신경과에서는 NIHSS, ASPECTS, 임상 병기 등)와 직접 비교하는 설계는 임상의를 설득하는 데 효과적입니다.
- NRI/IDI, 불일치 사례 분석(표준 지표와 모델이 다르게 판단한 환자 추적)을 기본 분석 세트로 넣습니다.

**C. 소표본 예측 연구의 통계적 보완**

- 이 포스터의 약점(넓은 신뢰구간, 임계값 선택, 교란)을 보완하는 분석 틀을 연구실 표준으로 정리합니다.
  - 반복 교차검증과 부트스트랩 신뢰구간
  - 임계값은 사전에 정하거나 교차검증 안에서 정하기
  - 나이, 크기 등으로 보정한 다변량 Cox 모델로 독립적 예측력 확인

> **추천**: **A + C 조합**입니다. 연구실에 소규모 신경 영상 코호트가 있다면, 고정 인코더 설계로 빠르게 결과를 내고, C의 통계적 보완으로 이 포스터보다 한 단계 더 엄밀한 연구를 만들 수 있습니다.

---

### References (포스터 인용)

1. Gronchi A, et al. *Ann Oncol*. 2021.
2. Savina M, et al. *BMC Med*. 2017.
3. Coindre JM. *Cancer*. 2001.
4. Vallières M, et al. *Phys Med Biol*. 2015.
5. Pai S, et al. arXiv:2501.09001. 2025. (CT-FM)
6. Blankemeier L, et al. arXiv:2406.06512. 2024. (Merlin)
