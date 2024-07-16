---
{"dg-publish":true,"permalink":"/data/datahub/docs/architecture/datahub-architecture-components/","noteIcon":"","created":"2024-06-30T00:39:32.595+09:00"}
---

#datahub #architecture 

DataHub 플랫폼은 아래 다이어그램에 표시된 구성 요소로 이루어져있다.

![](https://datahubproject.io/assets/images/datahub-components-ac4e0a67fb78387ac6efc0a5d66515c6.png)
# metadata store

- metadata store 는 메타데이터 그래프를 구성하는 **[엔터티 및 에스팩트](https://datahubproject.io/docs/metadata-modeling/metadata-model/)을 저장하는 역할**을 한다.
- 메타데이터 수집, primary key 로 메타데이터 수집, 엔터티 검색, 엔터티 간 관계 수집을 위한 API 도 포함한다.
- 기본 스토리지 및 인덱싱을 위한 MySQL, Elasticsearch 및 Kafka와 함께 Rest.li API 엔드포인트를 호스팅하는 Spring Java 서비스로 이루어졌다.

# metadata models

- 메타데이터 모델은 메타데이터 그래프를 구성하는 엔터티 및 에스팩트들 간 관계를 정의하는 스키마이다.
- 메타데이터 모델은 Protobuf 와 형식이 매우 유사한 모델링 언어 `PDL`을 사용하여 정의한다.
- 엔티티는 데이터세트, 대시보드, 데이터 파이프라인 등과 같은 메타데이터의 특정 클래스를 나타낸다.
	- 엔티티의 각 인스턴스는 urn 이라는 고유 식별자가 존재한다.
- 애스팩트는 엔티티 인스턴스와 연결된 관련 데이터 번들(description, tag)을 나타낸다.
	- 자세한 내용은 [링크](https://datahubproject.io/docs/metadata-modeling/metadata-model/#exploring-datahubs-metadata-model) 를 확인한다.

# ingestion framework

- Ingestion Framework 는 외부 소스(e.g Snowflake, Looker, MySQL, Kafka)에서 메타데이터를 추출하여 메타데이터 모델로 변환하고, Kafka 또는 Metadata Store Rest API 를 사용하여 DataHub 에 저장하기 위한 **확장 가능한 모듈식 Python 라이브러리**다.
- 스키마 추출, 프로파일링, 사용 정보 추출 등을 포함한 다양한 기능을 사용할 수 있도록 광범위한 [소스 커넥터](https://datahubproject.io/docs/metadata-ingestion/#installing-plugins)를 지원한다.

# graphql api

- GraphQL API 은 태그, 소유자, 링크 등을 엔티티에 추가 및 제거하기 위한 API 를 제공합니다.
	- 메타데이터 그래프를 구성하는 엔터티와의 상호 작용을 간단하게 만드는 **엔터티 지향 API**를 제공한다. 
	- 검색, 거버넌스, observability 를 활성화하기 위해 UI(아래에서 설명)에서 사용한다.

# user interface

- Datahub 는 데이터 에셋을 쉽게 검색, 관리 및 디버깅할 수 있도록 React UI 와 함께 제공한다.
