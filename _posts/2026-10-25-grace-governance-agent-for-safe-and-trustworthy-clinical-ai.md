---
title: "[KoSAIM] GRACE: Governance Agent for Safe and Trustworthy Clinical AI — 포스터 심층 분석"
date: 2026-10-25 21:00:00 +0900
categories: ["Medical AI","KoSAIM 2026"]
tags: ["Clinical AI Governance","GRACE","Minimum Common Dataset (MCDS)","Clinician-AI Co-Governance","Out-of-Distribution (OOD)","Thyroid FNAC","KoSAIM 2026"]
description: "배포 이후 임상 AI 시스템에서 발생하는 분포 외(OOD) 오분류와 감독 공백을 해결하기 위해, 임상의가 정의한 최소 공통 데이터셋(MCDS) 루브릭 기반의 거버넌스 에이전트 아키텍처인 GRACE를 제안합니다. 갑상선 세침흡인세포검사(Thyroid FNAC) 합성 케이스 20건을 통한 개념 증명(PoC)에서 10/10건의 정확한 거버넌스 분류와 임상의 승인 기반 지식 축적(Knowledge Accumulation)을 실증했습니다."
math: true
pin: false
image:
  path: /assets/img/posts/2026-10-25-kosaim-2026/poster-grace-kosaim-2026-full.jpg
  alt: "[KoSAIM] GRACE: Governance Agent for Safe and Trustworthy Clinical AI — 포스터 심층 분석"
---

## 포스터 개요 (Overview)

- **논문/포스터 제목**: GRACE: Governance Agent for Safe and Trustworthy Clinical AI
- **부제**: A Clinician–AI Co-Governance Architecture Powered by the Minimum Common Dataset (MCDS)
- **저자 및 소속**: **Minjin Lee, M.D., Ph.D.** (MedrixLab Inc., Korea) · **Jong Chul Ye, Ph.D.** (Graduate School of AI, KAIST, Korea)
- **발표 학회**: KoSAIM 2026 (대한의료인공지능학회) (Oct 23–24, 2026, Seoul, Korea)
- **핵심 키워드**: `Clinical AI Governance`, `GRACE`, `Minimum Common Dataset (MCDS)`, `Clinician-AI Co-Governance`, `Out-of-Distribution (OOD)`, `Thyroid FNAC`, `KoSAIM 2026`

![GRACE: Governance Agent for Safe and Trustworthy Clinical AI 전체 포스터](/assets/img/posts/2026-10-25-kosaim-2026/poster-grace-kosaim-2026-full.jpg)
*KoSAIM 2026 (대한의료인공지능학회) 발표 포스터 원본: GRACE: Governance Agent for Safe and Trustworthy Clinical AI*

> **Executive Summary**  
> 배포 이후 임상 AI 시스템에서 발생하는 분포 외(OOD) 오분류와 감독 공백을 해결하기 위해, 임상의가 정의한 최소 공통 데이터셋(MCDS) 루브릭 기반의 거버넌스 에이전트 아키텍처인 GRACE를 제안합니다. 갑상선 세침흡인세포검사(Thyroid FNAC) 합성 케이스 20건을 통한 개념 증명(PoC)에서 10/10건의 정확한 거버넌스 분류와 임상의 승인 기반 지식 축적(Knowledge Accumulation)을 실증했습니다.

---

## 1. 연구 배경 및 임상적 간극 (Background & Problem Gap)

### 1-1. 임상 AI의 거버넌스 격차 (The Governance Gap in Clinical AI)

의료 AI 모델이 실제 임상 현장에 배포된 이후(Post-deployment), 안전한 자율성을 보장하기 위해 해결해야 할 3가지 핵심 공백이 존재합니다.

1. **조용한 오분류 (Silent Misclassification)**
   - 폐쇄형(Closed-set) AI 시스템은 학습 데이터 분포를 벗어난(Out-of-Distribution, OOD) 입력이 들어왔을 때 이를 인지하지 못하고 높은 확신도로 오분류할 위험이 있습니다.
2. **표준화된 참조 데이터 인프라 부재 (Lack of Standardized Reference Data)**
   - AI의 임상적 결정을 감사(Audit)하고 감독(Supervision)하기 위한 공통 참조 데이터 인프라가 부재합니다.
3. **실용적인 임상 감독 도구의 한계 (Limited Clinical Tools)**
   - 임상의가 진료 현장에서 AI를 지속적으로 모니터링하고 효율적으로 개입(Human oversight)할 수 있는 실무 도구가 부족합니다.

### 1-2. GRACE의 접근 방식 (Governed Reasoning for Accountable Clinical Evidence)
- **GRACE**는 임상의가 정의한 참조 지식인 **MCDS(Minimum Common Dataset)**를 활용해 임상 AI의 결정을 평가하고, 잠재적 위험을 플래그(Flag)하며, 표적화된 임상 감독(Targeted clinical oversight)을 지원합니다.

---

## 2. 제안 방법론 및 핵심 아키텍처 (Proposed Methodology)

### 2-1. MCDS: 임상의 정의 참조 루브릭 (Clinician-Defined Reference Rubric)

**Minimum Common Dataset (MCDS)**은 AI가 생성한 임상 결정을 평가·감독·거버넌스하기 위한 임상적 핵심 기준(Rubric)입니다. 기존 병원 데이터 시스템을 대체하는 것이 아니라, 그 위에 얹히는 **상호보완적 참조 레이어(Complementary reference layer)**로 작동합니다.

#### MCDS의 4대 핵심 설계 원칙 (Key Design Principles)
- **Minimal (최소성)**: 거버넌스 검증에 꼭 필요한 핵심 임상 정보만 선별
- **Automated (자동화)**: 기존 병원 데이터(CDW, OMOP CDM, FHIR 등)로부터 자동 추출 및 매핑
- **Clinically Validated (임상적 검증)**: 전문의의 표적화된 검토 및 승인을 거친 신뢰성 확보
- **Lightweight & Domain-Specific (경량화 및 도메인 특화)**: 각 임상 분과/질환 도메인에 맞춘 경량 루브릭 구성

---

### 2-2. GRACE 임상의-AI 공동 거버넌스 5단계 순환 아키텍처
1. **Step 1 — AI Decision (AI 의사결정)**: 임상 입력 및 컨텍스트를 바탕으로 임상 AI가 진단 또는 권고안을 생성
2. **Step 2 — MCDS Assessment (MCDS 기반 평가)**: GRACE 거버넌스 에이전트가 임상의 정의 참조 루브릭(MCDS)을 기준으로 AI 결정을 검증하고 판단 근거(Result and rationale) 도출
3. **Step 3 — Risk Awareness (위험 인지 및 OOD 탐지)**: 학습 분포 외(Out-of-Distribution) 입력을 식별하여 위험 인지 출력(**Continue / Flag / Escalate to human oversight**) 결정
4. **Step 4 — Clinician Review (임상의 검토)**: 에스컬레이션된 케이스에 대해 임상의가 검토·수정·승인 수행 및 피드백 제공
5. **Step 5 — Knowledge Accumulation (지식 축적)**: 임상의의 승인을 거친 케이스가 향후 거버넌스 활용을 위해 MCDS 지식베이스(KB)를 지속 업데이트

---

## 3. 주요 Figure 및 비교 표 상세 분석 (Figures & Tables Breakdown)

### Figure 1. GRACE: Clinician–AI Co-Governance Architecture

> **요약**: 임상 입력부터 AI 의사결정, MCDS 기반 거버넌스 평가, OOD 위험 감지(Continue/Flag/Escalate), 임상의 검토, 그리고 지식 축적(Knowledge Accumulation)으로 이어지는 5단계 폐루프(Closed-loop) 아키텍처 다이어그램입니다.

![Figure 1: GRACE: Clinician–AI Co-Governance Architecture](/assets/img/posts/2026-10-25-kosaim-2026/poster-grace-kosaim-2026-figure-1.png)
*Figure 1: GRACE: Clinician–AI Co-Governance Architecture*

#### Figure 1. GRACE: 임상의–AI 공동 거버넌스 아키텍처 흐름

```
[Clinical Input & Context]
        │
        ▼
┌───────────────────────────────────────────────────────────┐
│ ① AI Decision                                             │
│   Clinical AI generates a diagnosis or recommendation     │
└─────────────────────────────┬─────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────┐
│ ② MCDS Assessment (GRACE Governance Agent)                │
│   Evaluate using clinician-defined reference rubric       │
└─────────────────────────────┬─────────────────────────────┘
                              │ (Result & Rationale)
                              ▼
┌───────────────────────────────────────────────────────────┐
│ ③ Risk Awareness                                          │
│   Identify Out-of-Distribution (OOD) inputs               │
│   ──▶ Risk-aware Output: Continue / Flag / Escalate       │
└─────────────────────────────┬─────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────┐
│ ④ Clinician Review                                        │
│   Review, modify, or approve when needed                  │
└─────────────────────────────┬─────────────────────────────┘
                              │ (Guidance & Feedback)
                              ▼
┌───────────────────────────────────────────────────────────┐
│ ⑤ Knowledge Accumulation                                  │
│   Approved cases update MCDS for future use               │
└───────────────────────────────────────────────────────────┘
```

---

### Figure 2. MCDS vs. Existing Healthcare Data Systems

> **요약**: FHIR, CDW, OMOP CDM, openEHR 등 기존 의료 데이터 표준/시스템과 AI 거버넌스를 위한 MCDS의 목적, 범위, 표준화 방식 및 GRACE 내 역할을 비교한 핵심 표입니다.

![Figure 2: MCDS vs. Existing Healthcare Data Systems](/assets/img/posts/2026-10-25-kosaim-2026/poster-grace-kosaim-2026-figure-2.png)
*Figure 2: MCDS vs. Existing Healthcare Data Systems*

#### Figure 2. MCDS vs. 기존 의료 데이터 시스템 비교 (Markdown Table)

| Aspect (구분) | FHIR (Data Exchange) | CDW (Data Warehouse) | OMOP CDM (Observational Model) | openEHR (Information Model) | **MCDS (for AI Governance)** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Primary Purpose (주 목적)** | Interoperability and information exchange (상호운용성 및 정보 교환) | Integrated storage and analysis (통합 저장 및 분석) | Standardized observational data (표준화된 관찰 연구 데이터) | Clinical information modeling (임상 정보 모델링) | **Reference dataset for governance assessment (거버넌스 평가를 위한 참조 데이터셋)** |
| **Main Focus (핵심 초점)** | Data format and exchange protocols | Data aggregation and management | Common data structure and vocabulary | Semantically structured clinical content | **Clinically relevant criteria for AI assessment ("scoring rubric")** |
| **Data Scope (데이터 범위)** | Clinical data (broad) | Institutional data (large-scale) | Observational data (analysis-ready) | Clinical data with detailed archetypes | **Minimum essential clinical information (domain-specific)** |
| **Standardization Approach (표준화 방식)** | Resource-based (HL7 FHIR) | Institution-specific model | Common Data Model (OMOP CDM) | Archetype-based model (openEHR) | **Clinician-defined reference framework (flexible & lightweight)** |
| **Key Users (주요 사용자)** | Healthcare systems, EHR vendors | Hospitals, researchers | Researchers, multi-institutional networks | Clinicians, EHR vendors, standards groups | **Clinicians, AI governance systems (e.g., GRACE)** |
| **Role in GRACE (GRACE 내 역할)** | Enables data exchange for MCDS construction | Provides source data for MCDS | Provides standardized data that can be mapped to MCDS | Informs domain-specific clinical modeling for MCDS | **Provides reference criteria for governance assessment in GRACE** |

---

## 4. 실험 설계 및 개념 증명 결과 (Proof-of-Concept & Results)

### 3. 갑상선 세침흡인세포검사(Thyroid FNAC) 합성 케이스 개념 증명 (Proof-of-Concept)

지식 기반 거버넌스 평가(Knowledge-based governance assessment)의 유효성을 검증하기 위해 갑상선 FNAC 합성 증례를 활용한 실험을 수행했습니다.

- **실험 설계 (Case Design)**
  - 총 **20건의 합성 갑상선 FNAC 케이스** (20 synthetic thyroid FNAC cases) 구성
  - **초기 참조 지식베이스 (Initial Reference KB)**: 10 케이스
  - **블라인드 평가 (Blind Evaluation)**: 10 케이스
  - 임상의가 정의한 MCDS 기준 적용 및 거버넌스 의사결정/KB 등록 적격성 평가
- **거버넌스 평가 결과 (Governance Assessment)**
  - **10 / 10 (100%) 정확한 거버넌스 분류 달성** (Correct governance classifications)
  - 진단 부적합 검체(Non-diagnostic specimen)에 대한 정확한 식별 및 필터링 포함
- **임상의 승인 기반 지식 축적 (Clinician-Approved Knowledge Accumulation)**
  - **Initial KB (10건)** $\rightarrow$ 블라인드 10건 중 **9건 승인(Approved)** + **1건 검체 불충분(Insufficient material) 제외** $\rightarrow$ **Final KB (19건)**으로 안전하게 지식 확장

---

## 5. 기대 효과 및 임상적 의의 (Clinical Significance & Impact)

### 4. 기대 효과 및 임상적 의의 (Expected Benefits & Aim)

단순한 임상 감독 보조 도구를 넘어 **책임 있는 의료 AI 생태계(Responsible AI Ecosystem)** 구축을 목표로 합니다.

1. **임상의 관점 (For Clinicians)**
   - "잘 훈련된 전공의(a well-trained resident)" 같은 신뢰할 수 있는 감독 보조 에이전트 역할 수행
   - 인지적 부담(Cognitive burden) 경감 및 임상적으로 의미 있는 고위험 케이스에 집중 가능
   - AI 안전 경계(Safety boundaries)를 정의하는 전문가적 주도권 확보
2. **의료기관 관점 (For Healthcare Institutions)**
   - 배포 후 지속적인 AI 모니터링 및 품질 관리(Continuous AI monitoring & quality management)
   - 감사 대응이 가능한 문서화(Audit-ready documentation) 및 기관 내 거버넌스 지식 자산 축적
   - 환자 안전성 향상 및 임상적 신뢰도 제고
3. **AI 개발자 관점 (For AI Developers)**
   - 임상적 한계점에 대한 구조화된 피드백 확보로 더 빠르고 안전한 배포 가능
   - 책임 있는 AI 자율성(Responsible AI autonomy)을 위한 장기 데이터 자산 확보
4. **의료 생태계 관점 (For the Healthcare Ecosystem)**
   - 검증·승인된 임상 거버넌스 지식의 공유 및 다양한 임상 도메인으로의 확장성 확보

### Take-Home Message

> **"Clinical AI needs not only intelligence, but also governable autonomy. GRACE provides a clinician-defined reference framework (MCDS) to enable safe, trustworthy, and responsible AI deployment through human–AI co-governance. (임상 AI에게는 단순한 지능뿐만 아니라 거버넌스 가능한 자율성이 필요합니다.)"**

---

## 현장 Q&A 및 연구자 노트 (Field Notes & Insights)

### 현장에서 나눈 질의응답 (On-Site Q&A)

```text
Q: 기존 OMOP CDM이나 FHIR 데이터가 구축된 병원에서는 MCDS를 어떻게 연동하나요?
A: MCDS는 기존 데이터 표준을 대체하는 것이 아니라, FHIR(데이터 교환) 및 OMOP CDM(표준 관측 데이터)으로부터 거버넌스 평가에 필수적인 최소 임상 기준만 자동 매핑·추출하여 AI 의사결정 검증 루브릭으로 작동합니다.
```

### 개인 감상평 및 연구 인사이트

> 임상 현장에 배포된 의료 AI 모델이 학습 분포 밖(Out-of-Distribution, OOD) 입력에 대해 조용한 오분류(Silent Misclassification)를 일으키는 문제를 단순 모델 재학습이 아닌 "임상의-AI 공동 거버넌스(Co-Governance) 에이전트"와 "최소 공통 데이터셋(MCDS)" 루브릭으로 풀어낸 점이 매우 실용적이다.

### 후속 연구 적용 아이디어 (Action Items)

- 우리 연구실의 영상/병리 판독 AI 파이프라인에도 배포 후 OOD 감지 및 임상의 피드백 루프(Knowledge Accumulation) 구조를 적용할 수 있을지 검토 필요.
