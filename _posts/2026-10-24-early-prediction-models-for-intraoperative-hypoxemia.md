---
title: "[KoSAIM] Analyzing Early Prediction Models for Intraoperative Hypoxemia — 포스터 심층 분석"
date: 2026-10-24 10:08:03 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Intraoperative Hypoxemia","Pediatric Anesthesia","Physiological Signals","iTransformer","Physiology-Informed AI","Alarm Fatigue","Event-Level Evaluation","KoSAIM 2026"]
description: "소아 수술 중 저산소혈증을 60초 전에 예측하는 모델에서, SpO₂만 쓰던 기존 방식에 환기 신호(EtCO₂, FiO₂, TV, PIP)를 더하면 AUPRC가 0.142 → 0.221로 오르는 것을 보였습니다. 신호 하나를 토큰 하나로 다루는 signal-axis iTransformer에 호흡 생리 지식을 attention 편향(Method 1) 또는 보조 예측 과제(Method 2)로 주입했고, 두 방법 모두 같은 재현율에서 XGBoost 대비 수술당 오경보를 약 1/3 줄였습니다(2.7 → 1.8회). 마취 모니터의 표준 신호 5개만 쓰므로 EMR 연동 없이 배포할 수 있습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-24-kosaim-2026/poster-intraop-hypoxemia-kosaim-2026-full.jpg
  alt: "[KoSAIM] Analyzing Early Prediction Models for Intraoperative Hypoxemia — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: Analyzing Early Prediction Models for Intraoperative Hypoxemia: The Contribution of Ventilatory Signals and Where Respiratory Physiological Knowledge Should Enter
- **국문 제목**: 수술 중 저산소혈증 조기 예측 모델 분석: 환기 신호의 기여와 호흡 생리 지식의 주입 위치
  - <!-- TODO: 리본에 가려진 국문 제목 일부("조기 예측 모델")는 영문 제목으로부터 추정, 확인 필요 -->
- **수상**: **Poster Award (Certificate of Merit)**
- **저자 및 소속**: **전현지**¹, 김희수², 박정빈², 김영곤³
  - ¹서울대학교 의과대학 의료정보학, ²서울대학교 의과대학 마취통증의학과, ³서울대학교병원 융합의학과
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회)
- **핵심 키워드**: `Intraoperative Hypoxemia`, `Pediatric Anesthesia`, `Physiological Signals`, `iTransformer`, `Physiology-Informed AI`, `Alarm Fatigue`, `Event-Level Evaluation`

![Analyzing Early Prediction Models for Intraoperative Hypoxemia 전체 포스터](/assets/img/posts/2026-10-24-kosaim-2026/poster-intraop-hypoxemia-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: Analyzing Early Prediction Models for Intraoperative Hypoxemia*

> **Executive Summary**  
> 소아 수술 중 저산소혈증을 **60초 전에** 예측하는 모델을 만들었습니다. 기존 연구는 SpO₂ 하나만 쓰거나 EMR 변수를 잔뜩 썼지만, 이 연구는 **마취 모니터의 표준 신호 5개**(EtCO₂, FiO₂, TV, PIP, SpO₂)만 씁니다. SpO₂에 환기 신호를 더하면 AUPRC가 **0.142 → 0.221**(p < 0.002)로 올랐습니다. 신호 하나를 토큰 하나로 다루는 **signal-axis iTransformer**에 호흡 생리 지식을 두 가지 방식으로 주입했습니다.
> - **Method 1**: 생리적으로 연결된 신호 쌍에 attention 편향을 줍니다.
> - **Method 2**: 각 신호의 미래를 예측하는 보조 과제를 학습 중에만 추가합니다.
>
> 두 방법 모두 같은 재현율에서 XGBoost보다 **수술당 오경보를 약 1/3 줄였습니다**(2.7 → 1.8회).

---

## 1. 연구 배경 및 목표 (Background & Objective)

### 1-1. 임상적 문제 (Clinical problem)

- **생명을 위협합니다**: 서맥과 심정지로 이어질 수 있습니다.
- **소아에서 더 빠르게 진행합니다**: 산소 요구량은 높고 산소 저장량은 적기 때문입니다.
- **늦게 발견됩니다**: SpO₂는 산소화가 이미 실패한 뒤에야 떨어집니다.

### 1-2. 기존 연구의 한계 (Limitations of prior work)

- **입력의 양극단**: SpO₂ 하나만 쓰거나 [1, 2], EMR과 연결한 변수를 많이 씁니다 [3–6].
- **시간축 토큰화만 사용**: 한 시점의 여러 신호를 한 토큰에 섞어서, 신호 간 관계가 흐려집니다.
- **임상 맥락 무시**: 호흡 생리학을 반영한 모델이 없었습니다.

### 1-3. 접근 방식과 기여

| 기술적 접근 (Our approach) | 임상적 기여 (Contribution) |
| :--- | :--- |
| **신호 관계 모델링**: 시간 토큰이 아니라 신호 토큰 사이의 attention | **더 이른 경보**: SpO₂가 떨어지기 전에 알려 선제 대응 가능 |
| **생리 지식 주입**: attention 편향 또는 보조 예측 | **오경보 감소**: 인공물(artifact)이 아닌 실제 저산소혈증을 목표로 학습 |
| **조기 예측**: 60초 전 예측, 실제(genuine) 사건만으로 학습 | **실시간 배포 가능**: 모니터 신호 5개만 사용, EMR 연동 불필요 |

---

## 2. 연구 방법 (Methods)

- **데이터**: 소아 수술 환자 **1,439명**, 모니터 신호 5개 (EtCO₂, FiO₂, TV, PIP, SpO₂). 라벨은 **임상의가 판정한 실제 저산소혈증 사건**입니다.
- **과제**: 60초 입력 창으로 **다음 60초 안의 저산소혈증**을 예측합니다. 사건 이전 구간만 사용합니다.
- **평가**:
  - 창(window) 단위 **AUPRC**(주 지표, 유병률 약 1%)와 AUROC
  - 사건(event) 단위: 같은 경보율 또는 같은 재현율에서의 재현율과 오경보
- **통계**: 5-seed 앙상블, **환자 단위 paired bootstrap**(B = 1,000)으로 XGBoost 대비 95% CI와 양측 p값 (\*p < 0.05)

### Figure 1. 제안 방법: Signal-axis iTransformer에 생리 지식을 주입하는 두 가지 방법

![Figure 1: Two ways to inject physiology into a signal-axis iTransformer](/assets/img/posts/2026-10-24-kosaim-2026/poster-intraop-hypoxemia-kosaim-2026-figure-1.png)
*Figure 1: Signal-axis 토큰화, Method 1(생리적 attention), Method 2(보조 예측)와 전체 학습 구조*

### 2-1. Signal-axis 토큰화 (Backbone: iTransformer [7])

- **기존(Time-axis)**: 토큰 하나 = 한 시점의 5개 신호 값
- **제안(Signal-axis)**: 토큰 하나 = **한 신호의 60초 전체**
- 신호 5개가 토큰 5개가 되므로, attention이 **"신호들이 서로 어떻게 상호작용하는지"**를 직접 학습합니다.

### 2-2. Method 1: 생리적 attention (Physiological attention)

- 생리적으로 연결된 신호 쌍에 **attention 편향 $+\alpha$**를 더합니다 (그림에서는 $+0.6$).
- 그림의 편향 행렬에서 강화된 연결은 EtCO₂ → TV, EtCO₂ → PIP, TV → PIP, PIP → TV입니다.

### 2-3. Method 2: 보조 예측 (Auxiliary prediction)

- 각 신호 토큰이 **자기 신호의 미래**를 예측하는 보조 과제를 추가합니다. 이 과제는 **학습 중에만** 쓰입니다.
  - 상류 신호(EtCO₂, FiO₂, TV, PIP): 다음 60초 평균과 직전 5초 평균의 차이 $\Delta = a - b$
  - SpO₂: 다음 60초의 최솟값
- **Task 1 (주 과제)**: SpO₂ 토큰만 읽어 60초 내 저산소혈증 예측 (L1: focal loss)
- **Task 2 (보조 과제)**: 신호별 미래 변화 회귀 (L2: MSE)
- 두 손실이 공유 인코더를 함께 업데이트합니다.

---

## 3. 실험 결과 (Results)

### Figure 2. 창 단위 평가와 사건 단위 평가

![Figure 2: Window-level and event-level evaluation](/assets/img/posts/2026-10-24-kosaim-2026/poster-intraop-hypoxemia-kosaim-2026-figure-2.png)
*Figure 2: (A) 환기 신호의 추가 가치, (B) 기존 모델 vs 생리 정보 모델, (C) 173개 실제 사건에서의 경보 부담과 FROC 곡선*

### 3-1. (A) SpO₂ 대비 환기 신호의 추가 가치

| Input signals | Test AUROC [95% CI] | Test AUPRC [95% CI] |
| :--- | :--- | :--- |
| SpO₂ only | 0.773 [0.714, 0.783] | 0.142 [0.113, 0.182] |
| **SpO₂ + ventilatory (EtCO₂, FiO₂, TV, PIP)** | **0.924 [0.901, 0.945]** | **0.221 [0.183, 0.268]** |
| Δ (adding ventilatory signals) | | **+0.080 [0.041, 0.113], p < 0.002** |

*같은 signal-axis backbone과 학습 설정에서 입력 신호만 다르게 했습니다.*

### 3-2. (B) 기존 모델 vs 생리 정보 모델 (신호 5개)

| 구분 | Model | Test AUROC [95% CI] | Test AUPRC [95% CI] |
| :--- | :--- | :--- | :--- |
| Baseline | Transformer [3, 4] | 0.920 [0.897, 0.941] | 0.184 [0.144, 0.239] |
| | LSTM [1–4, 6] | 0.922 [0.900, 0.942] | 0.202 [0.164, 0.246] |
| | XGBoost [1, 3–6] (reference) | 0.918 [0.893, 0.938] | 0.187 [0.149, 0.234] |
| **Method 1** | Signal-axis + attention bias | 0.922 [0.899, 0.943] | **0.225\* [0.184, 0.273]** |
| **Method 2** | Signal-axis + aux: SpO₂ (1) | 0.928\* [0.907, 0.947] | 0.215\* [0.179, 0.261] |
| | Signal-axis + aux: Upstream (4) | 0.928 [0.905, 0.948] | 0.224\* [0.182, 0.272] |
| | Signal-axis + aux: All (5) | **0.929\* [0.907, 0.949]** | 0.224\* [0.181, 0.273] |

*\*p < 0.05 vs XGBoost, 환자 단위 paired bootstrap*

- AUROC는 모든 모델이 0.92 안팎으로 비슷합니다. **차이는 AUPRC에서 드러납니다.**
- 생리 지식을 넣은 signal-axis 모델의 AUPRC(0.215–0.225)는 시간축 Transformer(0.184)와 XGBoost(0.187)보다 높았습니다.

### 3-3. (C) 173개 실제 사건에서의 경보 부담 (사건 단위 평가)

*기존 연구에서 거의 보고되지 않은 평가입니다. 173개 사건, 143명 환자.*

| Model | Recall (%) ↑ at 10 alarms/10 h | Alarms/10 h ↓ at 80% recall | False alarms/surgery ↓ at 80% recall |
| :--- | :--- | :--- | :--- |
| XGBoost | 59.6 [50.8, 69.1] | 19.6 [14.4, 25.8] | 2.71 [1.76, 3.81] |
| Transformer | 58.2 [48.1, 70.8] | 19.3 [13.6, 26.0] | 2.66 [1.58, 3.86] |
| **Method 1** | **68.6\* [59.4, 78.2]** | **14.5\* [10.6, 20.2]** | **1.76\* [1.07, 2.79]** |
| **Method 2** | 66.7\* [56.2, 77.6] | 14.7\* [10.7, 21.4] | 1.79\* [1.11, 2.97] |

- 같은 경보 빈도(10시간당 10회)에서 **놓치는 사건이 줄었고**, 같은 재현율(80%)에서 **경보와 오경보가 줄었습니다**.
- FROC 곡선(전체 임계값에서 경보 수 대비 사건 재현율)에서도 Method 1, 2가 기존 모델보다 위에 있습니다.

---

## 4. 결론 및 향후 과제 (Conclusion & Future work)

1. **환기 신호가 조기 예측에 기여합니다**: 모니터 신호만으로 AUPRC가 0.142 → 0.221로 올랐습니다 (p < 0.002).
2. **시간축이 아니라 신호축 모델링**: 신호 하나를 토큰 하나로 두고 생리 지식을 주입한 모델이 시간축 Transformer보다 좋았습니다 (AUPRC 0.225 vs 0.184).
3. **생리 지식을 넣는 위치**: Method 1과 2 모두 같은 재현율에서 XGBoost보다 **오경보를 약 1/3 줄였습니다** (수술당 2.7 → 1.8회).
4. **최소 입력, 배포 이점**: 표준 모니터 신호 5개만 쓰므로 **EMR 연동이나 추가 센서 없이** 기존 마취 모니터에서 작동할 수 있습니다.
5. **향후 과제**: 독립 코호트(충남대병원, CNUH)에서 외부 검증 예정, IRB 승인 진행 중

### Take-Home Message

> **"Ventilatory signals contribute to early prediction, and injecting respiratory physiology into a signal-axis model cuts false alarms by about one third at equal recall — using only five standard monitor signals."**
>
> (환기 신호는 저산소혈증 조기 예측에 기여하고, 신호축 모델에 호흡 생리 지식을 주입하면 같은 재현율에서 오경보를 약 1/3 줄일 수 있습니다. 필요한 것은 마취 모니터의 표준 신호 5개뿐입니다.)

---

## 포스터 Q&A 및 연구자 노트 (Q&A & Insights)

### 포스터 읽으며 나눈 Q&A (with Claude)

#### Q. 왜 AUROC가 아니라 AUPRC를 주 지표로 썼나요?

저산소혈증 창의 유병률은 약 **1%**입니다. 이렇게 드문 사건에서는 AUROC가 쉽게 높게 나옵니다. 음성이 압도적으로 많아서 위양성률(FPR)이 작게 보이기 때문입니다.

- 실제로 (B) 표에서 모든 모델의 AUROC는 0.918–0.929로 거의 같지만, AUPRC는 0.184–0.225로 차이가 납니다.
- AUPRC는 "경보가 울렸을 때 실제 사건일 확률(정밀도)"을 반영하므로 **오경보 부담**과 직접 연결됩니다.
- 참고로 무작위 모델의 AUPRC는 유병률과 같은 약 0.01이므로, 0.22는 무작위보다 약 **22배** 높은 수준입니다.

#### Q. "사건 단위 평가"가 왜 중요한가요?

창 단위 AUPRC는 60초 창 하나하나를 독립된 샘플로 봅니다. 하지만 임상의가 체감하는 것은 **"이번 수술에서 경보가 몇 번 울렸고, 그중 몇 번이 진짜였는가"**입니다.

- 이 연구는 10시간당 경보 수, 수술당 오경보 수처럼 **임상 현장의 언어로** 성능을 보고했습니다.
- 경보 피로(alarm fatigue)는 수술실과 중환자실에서 실제 안전 문제이므로, 이 지표가 모델 도입 여부를 결정하는 데 훨씬 직접적입니다.

#### Q. 통계적으로 잘한 점과 아쉬운 점은?

**잘한 점**
- **환자 단위 paired bootstrap**: 한 환자에게서 나온 여러 창은 서로 독립이 아닙니다. 환자 단위로 재표집하면 이 군집 구조를 반영한 신뢰구간을 얻습니다. 같은 환자에 대한 두 모델 비교이므로 paired로 한 것도 적절합니다.
- **5-seed 앙상블**로 학습 무작위성의 영향을 줄였습니다.
- **임상의가 판정한 실제 사건**만 라벨로 써서, 센서 탈락 같은 인공물을 사건으로 학습하지 않게 했습니다.

**아쉬운 점**
- **SpO₂ only의 AUROC 신뢰구간**: 0.773 [0.714, 0.783]은 점추정치가 구간 상단에 치우쳐 있습니다. bootstrap 분포가 비대칭일 수 있지만, 확인이 필요해 보입니다.
- **Method 간 차이의 검정 없음**: 모든 비교가 XGBoost 기준입니다. Method 1과 Method 2, 또는 Method와 LSTM(AUPRC 0.202) 간 차이는 신뢰구간이 크게 겹쳐 유의하지 않을 수 있습니다.
- **생리 연결과 $\alpha$의 선택 근거**: 어떤 신호 쌍을 연결할지, $\alpha = 0.6$을 어떻게 정했는지(사전 지식인지, 검증 세트로 조정했는지)가 포스터에 나와 있지 않습니다.
- **단일 기관, 소아 코호트**: 외부 검증(충남대병원)이 예정되어 있습니다.

#### Q. 이 연구가 우수 포스터로 뽑힌 이유는 무엇일까요?

심사 기준은 알 수 없으므로 추정입니다.

- **임상 질문이 명확합니다**: "SpO₂ 외 환기 신호가 도움이 되는가?"와 "생리 지식을 어디에 넣어야 하는가?"라는 두 질문에 각각 답했습니다.
- **임상 현장의 언어로 평가했습니다**: 오경보/수술처럼 마취과 의사가 바로 이해할 수 있는 지표를 썼습니다.
- **배포 가능성이 높습니다**: 표준 모니터 신호 5개만 쓰므로 EMR 연동 없이 바로 붙일 수 있습니다.

<!-- TODO: 현장에서 직접 나눈 질의응답이 있다면 추가 -->

### 개인 감상평 및 연구 인사이트

> <!-- TODO: 직접 작성 -->
> 지금까지 본 LLM 중심 포스터들과 달리, **생체신호와 도메인 지식(호흡 생리학)**을 모델 구조에 녹인 연구입니다. "AUROC는 다 비슷한데 AUPRC와 오경보에서 차이가 난다"는 결과는, 드문 사건 예측에서 **어떤 지표로 평가하느냐가 결론을 바꾼다**는 점을 잘 보여줍니다.

### 후속 연구 적용 아이디어 (Action Items) — 신경과 연구실의 통계/AI 연구원 관점

**A. 뇌파(EEG) 기반 발작 탐지·예측에 signal-axis 모델 적용** ⭐ 우선순위 1

- 다채널 EEG는 이 포스터의 "여러 모니터 신호"와 구조가 같습니다. 채널 하나를 토큰 하나로 두는 signal-axis 토큰화를 그대로 적용할 수 있습니다.
- **Method 1의 생리 지식** → 전극 배치(montage)상 인접한 채널, 같은 반구 채널 쌍에 attention 편향을 주는 방식으로 옮길 수 있습니다.
- **사건 단위 평가**는 발작 탐지 분야의 표준 지표(**24시간당 오경보 수**, 사건 단위 민감도)와 정확히 같은 철학입니다. 이 포스터의 평가 틀을 그대로 가져올 수 있습니다.

**B. 신경계 중환자실(Neuro-ICU) 악화 조기 경보**

- 두개내압(ICP), 뇌관류압(CPP), 혈압, 심박수, SpO₂ 같은 모니터 신호로 **ICP 상승이나 신경학적 악화를 미리 예측**하는 과제입니다.
- 이 포스터처럼 "EMR 없이 모니터 신호만으로" 설계하면 실시간 배포가 쉬워집니다.
- Method 2(보조 예측)는 "각 신호의 다음 변화"를 함께 학습시켜, 데이터가 적은 신경계 중환자 코호트에서 일반화를 도울 수 있습니다.

**C. 드문 사건 예측의 평가 방법론**

- 유병률이 낮은 신경과 사건(발작, 뇌졸중 후 악화, 섬망 등) 예측 연구에 대해, **AUPRC + 사건 단위 경보 부담 + 환자 단위 bootstrap**을 표준 평가 세트로 정리합니다.
- 통계/AI 연구원이 연구실의 여러 예측 모델 프로젝트에 공통으로 적용할 평가 프로토콜이 됩니다.

> **추천**: 연구실에 장시간 EEG 데이터가 있다면 **A**가 가장 직접적입니다. 모델 구조와 평가 틀을 모두 그대로 옮길 수 있습니다. 데이터 확보 전이라도 **C**는 바로 시작할 수 있습니다.

---

### References (포스터 인용)

1. Erion G, et al. *arXiv*. 2017.
2. Annapragada AV, et al. *PLOS Comput Biol*. 2021.
3. Park JB, et al. *PLOS ONE*. 2023.
4. Baek S, et al. *PLOS ONE*. 2026.
5. Lundberg SM, et al. *Nat Biomed Eng*. 2018.
6. Liu H, et al. *CIKM*. 2022.
7. Liu Y, et al. *ICLR*. 2024. (iTransformer)
