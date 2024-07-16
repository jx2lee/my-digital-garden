---
{"dg-publish":true,"permalink":"/data/datahub/docs/architecture/datahub-architecture-serving-architecture/","noteIcon":"","created":"2024-06-30T00:39:32.595+09:00"}
---

#datahub #architecture #serving-tier

**DataHub Serving Tier에 대한 개략적인 다이어그램은 아래와 같다.**

![https://datahubproject.io/assets/images/datahub-serving-0a1f223ecb2d809052fd1eed539b0b3a.png](https://datahubproject.io/assets/images/datahub-serving-0a1f223ecb2d809052fd1eed539b0b3a.png)

구성 요소는 metadata service 라고 하며 메타데이터에서 CRUD 작업을 수행하기 위한 **REST API 및 GraphQL API**를 제공한다. 또한, 검색 및 그래프 쿼리 API를 제공하여 보조 인덱스 스타일 쿼리(secondary-index style queries), 전체 텍스트 검색 쿼리 및 lineage 와 같은 관계 쿼리를 지원한다. datahub-frontend 는 GraphQL API를 제공한다.

# DataHub Serving Tier 구성요소
## Metadata Store
metadata service 는 document storage 혹은 MySQL, Postgres 또는 Cassandra 등과 같은 RDBMS에 메타데이터를 저장한다.

## Metadata Change Log Stream (MCL)
DataHub Service Tier 는 메타데이터 변경이 스토리지에 성공적으로 커밋되면, 커밋 이벤트 메타데이터 변경 로그(Metadata Change Log, MCL)를 Kafka를 통해 전송한다.

MCL 스트림은 공용 API이며 외부 시스템(예: Actions Framework)에서 구독할 수 있어 **메타데이터에서 발생하는 변경 사항에 실시간으로 반응하는 강력한 방법을 제공**한다. 예를 들어 메타데이터(예: 이전에는 누구나 읽을 수 있는 데이터 세트에 개인정보 필드가 추가됨)의 변경에 반응하는 액세스 제어 시스템을 구축한다면, **문제의 데이터 세트를 즉시 잠글 수 있다**. 메타데이터의 중복 변경 사항을 무시하기 때문에 모든 MCP-s 가 MCL 로 이어지진 않는다.

# Metadata Index Applier (mae-consumer-job)
MCP 는 mae-consumer-job([그래프](https://datahubproject.io/docs/what/graph/) 및 [검색 인덱스](https://datahubproject.io/docs/what/search-index/) 변경 사항을 반영하는 Spring job) 을 통해 소비된다. 작업은 엔터티에 독립적이며 특정 메타데이터 aspect 가 변경될 때 호출되는 해당 그래프 및 검색 인덱스 builder 를 실행한다. builder 는 메타데이터 변경에 따라 그래프 및 검색 인덱스를 업데이트하는 방법을 job 에 전달해야 한다.

메타데이터 변경을 시간순으로 처리하도록 엔터티 [URN](https://datahubproject.io/docs/what/urn/) 은 MCL 에 키를 지정한다. 즉, 특정 엔터티에 대한 모든 MAE 는 단일 스레드에 의해 순차적으로 처리된다.

> [URN(Uniform Resource Name)](https://en.wikipedia.org/wiki/Uniform_Resource_Name): DataHub 내 모든 리소스를 고유하게 정의하기 위해 선택한 URI 체계이다. `urn:<Namespace>:<Entity Type>:<ID>` 형식을 따르며 새 엔터티를 선언하는 것은 특정 URN을 모델링하는 것으로 시작된다. 기본 제공 엔터티에 대한 [기존 URN 모델](https://github.com/datahub-project/datahub/tree/master/li-utils/src/main/javaPegasus/com/linkedin/common/urn)을 참조로 사용할 수 있다. 구성요소에 대한 내용은 위 링크를 참고해보자.

# Msetadata Query Serving
document 저장소는 메타데이터에 대한 기본 키 기반 읽기(e.g `dataset-urn` 기반 데이터셋의 스키마데이터)를 라우팅한다. 메타데이터에 대한 Secondary Index 기반 읽기는 search 인덱스로 라우팅된다. 전체 텍스트 및 고급 검색(Full-text and advanced search) 쿼리는 search 인덱스로 라우팅된다. Lineage 와 같은 그래프 쿼리는 graph index 로 라우팅된다.

- search 인덱스: Secondary Index 기반 읽기, 전체 텍스트 및 고급 검색 쿼리
- graph 인덱스: lineage 그래프 쿼리
- document 스토리지: 기본키 기반 읽기