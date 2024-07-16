---
{"dg-publish":true,"permalink":"/data/datahub/docs/architecture/datahub-architecture-overview/","noteIcon":"","created":"2024-06-30T00:39:32.595+09:00"}
---

#datahub #architecture


![](https://datahubproject.io/assets/images/datahub-architecture-30b34a888241e0780c72b7f618137fe4.png)

DataHub는 최신 데이터 스택으로 구축한 데이터 검색, 협업, 거버넌스 및 종단 간 관찰 기능을 지원하는 3세대 메타데이터 플랫폼이다. 서로 다른 도구 및 시스템 간의 [상호운용성](https://ko.wikipedia.org/wiki/상호운용성) 에 중점을 둔 모델 우선 철학을 갖는다.

architetcure 는 다음 세 가지 메인 특징을 갖는다.

# 스키마 우선 접근 방식의 메타데이터 모델링 (Schema-first approach to Metadata Modeling)

- DataHub 메타데이터 모델은 [serialization agnostic language(직렬화 불가지론 언어)](https://linkedin.github.io/rest.li/pdl_schema)를 사용한다.
- REST & GraphQL API 를 모두 지원한다.
- Kafka를 통해 AVRO 기반 API 를 지원하여 메타데이터 변경 사항을 발행하고 구독한다.
- [메타데이터 모델링](https://datahubproject.io/docs/metadata-modeling/metadata-model/) 에 대한 자세한 내용은 링크를 참고한다.

# 스트림 기반 실시간 메타데이터 플랫폼 (Stream-based Real-time Metadata Platform)

- DataHub 메타데이터 인프라는 스트림 지향적이므로 메타데이터의 변경 사항을 몇 초 내에 플랫폼 내에서 전달하고 반영할 수 있다.
- DataHub 의 메타데이터에서 발생하는 변경 사항을 구독하여 실시간 메타데이터 기반 시스템을 구축할 수 있다.
	- 예를 들어, 접근제어 시스템을 구축할 수 있다.
	- 이전에 누구나 읽을 수 있는 데이터에 개인정보가 추가된다면 해당 데이터를 실시간으로 잠글 수 있다.

# 통합 메타데이터 제공 (ederated Metadata Serving)