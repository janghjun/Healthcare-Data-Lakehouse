# 🏥 Healthcare Data Lakehouse (FHIR)

## 📌 프로젝트 개요
의료 데이터(FHIR 표준 기반)를 활용한 **데이터 레이크하우스 파이프라인 구축** 프로젝트입니다.  
데이터 품질 검증, 메타데이터 관리, BI 시각화를 포함한 **엔드투엔드(End-to-End) 데이터 엔지니어링** 사례입니다.

---

## ⚙️ 아키텍처
- Source: **Synthea** (합성 환자 데이터)
- Ingestion: Apache Airflow
- Storage: AWS S3 + Delta Lake / Iceberg
- Transformation: dbt + fhir-dbt-utils
- Data Quality: Great Expectations
- Metadata/Lineage: OpenMetadata / DataHub
- Visualization: Tableau / Power BI

![architecture](docs/architecture.png)

---

## 🚀 주요 기능
- FHIR 환자 데이터 적재 및 변환
- **Bronze → Silver → Gold** 계층 구조 설계
- GX 기반 데이터 품질 자동 검증
- OpenMetadata 기반 계보 추적 및 데이터 카탈로그
- BI 대시보드: 환자 재입원율, 진료 패턴 분석

---

## 📊 결과물
-	GX 데이터 품질 리포트 (docs/gx_report.html)
-	Tableau 대시보드 (docs/tableau_dashboard.png)
-	OpenMetadata 계보 다이어그램

## 향후 개선
- 실시간 스트리밍 데이터 연계
- 의료 데이터 De-ID(비식별화) 적용

  ---
