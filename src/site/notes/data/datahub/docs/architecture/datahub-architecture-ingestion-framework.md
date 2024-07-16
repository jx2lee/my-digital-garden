---
{"dg-publish":true,"permalink":"/data/datahub/docs/architecture/datahub-architecture-ingestion-framework/","noteIcon":"","created":"2024-06-30T00:39:32.595+09:00"}
---

#datahub #architecture #ingestion-framework

**DataHub 는 push/pull, 비동기 및 동기 모델을 지원하는 유연한 수집 아키텍처를 제공**한다. 아래 그림은 DataHub 의 ingestion framework 를 간략히 표현하였다.

![https://datahubproject.io/assets/images/ingestion-architecture-cd631d7c4a648ceb82908ce25b9f93b9.png](https://datahubproject.io/assets/images/ingestion-architecture-cd631d7c4a648ceb82908ce25b9f93b9.png)

# Metadata Change Proposal
Ingestion 핵심은 **datahub 의 메타데이터 변경 요청**을 의미하는 [metadata change proposal(이하 MCP)]() 이다. MCP 는 kafka 를 통해 주고 받을 수 있어 source 로 부터 확장성이 뛰어난 비동기 Publishing 이 가능하다. 또한, 동기식 성공/실패 응답을 얻기 위해 DataHub HTTP endpoint 로 직접 보낼 수도 있다.

# Pull-based Integration
DataHub 는 다양한 소스에 연결하여 메타데이터를 가져올 수 있는 [Python 기반 메타데이터 수집 시스템](https://datahubproject.io/docs/metadata-ingestion/)을 제공한다. 이 메타데이터는 **Kafka 또는 HTTP를 통해 DataHub 스토리지 계층으로 push** 한다. 메타데이터 수집 파이프라인을 Airflow와 통합하여 배치를 설정하거나 lineage 를 캡처할 수 있다. 이미 지원되는 소스를 찾지 못한 경우 직접 개발하는 환경을 제공한다.

# Push-based Integration
MCP(Metadata Change Proposal) 이벤트를 Kafka로 보내거나 HTTP를 통해 REST 호출을 한다면, 모든 시스템을 DataHub 와 통합할 수 있다. DataHub 는 시스템에 통합하여 원본 지점 메타데이터 변경(MCP-s)을 내보낼 수 있는 [Python emitter](https://datahubproject.io/docs/metadata-ingestion/#using-as-a-library) 도 제공한다.

# Internal Compoenent
## Applying Metadata Change Proposals to DataHub Metadata Service (mce-consumer-job)
MCP 를 consume 하고 `/ingest` 엔드포인트를 이용해 datahub-gms 에 기록하는 mce-consumer-job(Spring)를 함께 제공한다.