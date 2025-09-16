# 만성질환(고혈압·당뇨) 환자의 **약물+영양제 병용 패턴**과 임상지표 연관성

## 프로젝트 개요
현대인의 만성질환 관리에서 **약물 치료와 건강기능식품(보충제) 병용**은 흔하지만,
실제로 혈압이나 혈당과 같은 주요 임상지표 개선 효과는 명확하지 않습니다.

- **당뇨병 가이드라인**: 비타민·미네랄 보충제를 HbA1c 개선 목적으로 **일반 권고 불가**
- **고혈압 연구 결과**: 마그네슘·칼륨 보충이 **혈압을 소폭 낮추는 신호**가 반복 보고됨
- **한계**: 연구 설계·집단·복용량이 제각각 → 실제 한국인 환자에 대한 **근거 부족**

**→** 따라서 본 프로젝트는 다음을 목표로 합니다:

> **“한국인 고혈압(메인)·당뇨(보조) 환자의 약물+보충제 병용 패턴이 임상지표(SBP/DBP, HbA1c)와 어떤 연관성을 가지는지 관찰한다.”**

---

## 상관분석 예상 질문
1. 고혈압 환자에서 약물 단독 vs 약물+보충제 병용군의 SBP/DBP 차이는?  
2. 당뇨 환자에서 동일 비교 시 HbA1c와 어떤 연관성이 나타나는가?  
3. 보충제 카테고리별(Mg/K/오메가-3/CoQ10/비타민D) 효과 신호는 어떻게 다른가?  
4. HIRA/NHIS 처방 패턴과 KNHANES 보충제 패턴은 어떻게 보완적인가?  

---

## 데이터셋
- **국민건강영양조사 (KNHANES)**: 혈압·HbA1c, 생활습관, 보충제 섭취 정보 → *주 분석 데이터*
- **HIRA/NHIS 청구데이터**: 진단(ICD-10/KCD), 처방(ATC), 다제약제 지표 → *약물 패턴 보완*
- **MFDS(식약처) 오픈 API**: 건강기능식품 원료/제품 메타데이터 → *보충제 성분 사전 구축*
- **NHANES + DSLD/DSID**: 해외 데이터 → *민감도·재현성 검증*

---

## 연구 설계
- **코호트 정의**
  - 고혈압: SBP ≥140 또는 DBP ≥90, 혹은 진단/복약 보고
  - 당뇨: HbA1c ≥6.5% 혹은 진단/복약 보고
- **노출군 분류**
  - (A) 약물만
  - (B) 약물+보충제 병용
  - (C) 보충제만
  - (D) 무복용
- **아웃컴**: SBP/DBP, HbA1c
- **공변량**: 나이, 성별, BMI, 흡연, 음주, 신체활동, 식이(나트륨/칼로리), 소득, 교육수준 등
- **한계**: 단면조사 → *인과 추론 불가, 연관성 분석으로 제한*

---

## 데이터 표준화 (FHIR 매핑)
- **혈압, HbA1c** → `Observation` (LOINC 코드 매핑)
- **질환 여부** → `Condition` (ICD-10/KCD 매핑)
- **약물 처방/복용** → `MedicationStatement` (ATC 기반)
- **보충제 섭취 이벤트** → `NutritionIntake`
- **보충제 성분/제품** → `NutritionProduct` (MFDS API 매핑)

---

## 데이터 파이프라인
- **Ingestion/Storage**: Airflow → S3/MinIO → Apache Iceberg
- **Transform**: Spark + dbt (fhir-dbt-utils)
- **Data Quality**: Great Expectations
  - 혈압/HbA1c 합리 범위 검증
  - 성분코드/참조무결성 검증
- **Metadata/Lineage**: OpenMetadata (Airflow·dbt·Trino 연동)
- **Gold marts**
  - `mart_bp_supplement` (고혈압 분석용)
  - `mart_dm_supplement` (당뇨 분석용)
  - `mart_rx_patterns` (처방 패턴 보완)

---

## 분석 방법
- 복합표본가중치 반영 **회귀 분석** (선형/로지스틱)
- **성향점수 매칭 (IPTW)** → 교란 통제
- **서브그룹 분석**: 고령(≥65), 성별, BMI, 다제약제
- **감도분석**: 부정적 대조노출/결과, 기준치 변형 (BP 130/80, HbA1c 7.0%)
- **해석 수준**: *연관성 중심 (인과 추론 아님)*

---

## LLM/RAG·Agent 활용
- **보충제 지식 RAG 챗봇**
  - 데이터: MFDS 원료 사전 + 가이드라인 + 논문 요약
  - Vector DB 색인(FAISS/Chroma)
  - LangChain/LangGraph 기반 → “질문 → 검색 → 근거 요약 → 답변”
- **Agentic Workflow**
  - 사용자 질의 → 성분 매칭 → KNHANES 결과 대조 → 해석 제공
  - MCP 기반 데이터 카탈로그 질의, GE 품질 리포트 자동 호출

---

## 결과물
- **대시보드 (Streamlit/Metabase)**
  - 고혈압: 병용 vs 비병용군 SBP/DBP 차이 (보정 추정치 + 95% CI)
  - 당뇨: HbA1c 분포/조절률 비교
- **리포트/시각화**
  - Great Expectations 데이터 품질 리포트
  - OpenMetadata 라인리지 다이어그램
  - 분석 노트북 (가중치/PSM/회귀 결과)
- **API/데모**
  - `/api/query_bp_effect` : 코호트/노출 입력 → 조정차이 반환
  - `/chat/supplement` : 보충제 관련 질의응답 (RAG 기반)

---

## 레포 구조 (초안)
```
chronic-supplement-bp-a1c/
├─ airflow/dags/
├─ infra/              # docker-compose, trino, minio, iceberg, metabase
├─ etl/
│  ├─ ingest_knhanes.py
│  ├─ ingest_hira.py
│  ├─ ingest_mfds.py
│  └─ utils_fhir_map.py
├─ dbt/
│  ├─ models/{raw,stg,silver,gold}/
│  └─ macros/quality.sql
├─ expectations/       # great expectations suite
├─ notebooks/
│  ├─ 01_weighting_design.ipynb
│  ├─ 02_psm_iptw.ipynb
│  └─ 03_regression_results.ipynb
├─ rag_agent/
│  ├─ build_index.py
│  ├─ chain_graph.py    # LangGraph agent
│  └─ api.py            # FastAPI
├─ dashboards/         # streamlit app
└─ docs/               # study protocol, DQ report, lineage
```

---

## 기대 효과
- **실무 가치**: 한국인 대상 약물+보충제 병용에 대한 실제 근거 신호 제공  
- **기술 스택**: 데이터 레이크하우스, ETL, MLOps, 품질 관리, RAG/Agent 적용  
- **도메인 결합**: 헬스케어 + 데이터 엔지니어링 + AI 서비스 기획 경험 강조  
- **기업 니즈 충족**
  - 데이터 엔지니어링/레이크하우스 구축 경험 (Saltlux·현대·두산 계열사 니즈)  
  - LLM/RAG/Agent 응용 서비스 기획 (마이리얼트립, HS애드 등 생성형 AI 서비스 기업 니즈)  
  - 금융·헬스 도메인 데이터 분석 경험 (신한, 한국평가데이터, 비씨카드 채용 니즈)  
