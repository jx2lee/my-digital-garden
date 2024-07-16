---
{"dg-publish":true,"permalink":"/data/kafka/__/schema-registry/","tags":["kafka","schema registry"],"noteIcon":"","created":"2024-06-30T00:39:32.604+09:00"}
---


### overview


> Schema Registry provides a centralized repository for managing and validating schemas for topic message data, and for [serialization](https://docs.confluent.io/platform/current/_glossary.html#term-serializer) and [deserialization](https://docs.confluent.io/platform/current/_glossary.html#term-deserializer) of the data over the network. [Producers](https://docs.confluent.io/platform/current/_glossary.html#term-producer) and [consumers](https://docs.confluent.io/platform/current/_glossary.html#term-consumer) to Kafka topics can use schemas to ensure data consistency and compatibility as schemas evolve. Schema Registry is a key component for [data governance](https://docs.confluent.io/cloud/current/stream-governance/index.html#stream-governance-on-ccloud), helping to ensure data quality, adherence to standards, visibility into data lineage, audit capabilities, collaboration across teams, efficient application development protocols, and system performance.


- 스키마 레지스트리는 토픽 메시지의 스키마를 관리 및 검증하고 네트워크를 통해 데이터를 직렬화 및 역직렬화하기 위한 중앙 집중식 저장소
- Kafka 토픽의 생산자와 소비자는 스키마를 사용해 스키마 변경(혹은 진화, evolution)함에 따라 데이터 일관성과 호환성을 보장한다.
- 데이터 거버넌스의 핵심 구성 요소로, 데이터 품질, 표준 준수, 데이터 계보에 대한 가시성, 감사 기능, 팀 간 협업, 효율적인 애플리케이션 개발 프로토콜, 시스템 성능을 보장하는 데 도움이 됨

![](https://i.imgur.com/56VsdS0.png)

> Schema Registry provides several benefits, including data validation, compatibility checking, versioning, and evolution. It also simplifies the development and maintenance of data pipelines and reduces the risk of data compatibility issues, data corruption, and data loss.
> 
> Schema Registry enables you to define schemas for your data formats and versions, and register them with the registry. Once registered, the schema can be shared and reused across different systems and applications. When a producer sends data to a message broker, the schema for the data is included in the message header, and Schema Registry ensures that the schema is valid and compatible with the expected schema for the topic.

- 데이터 유효성 검사, 호환성 검사, 버전 관리 및 진화(evolution)를 비롯한 여러 가지 이점을 제공한다. 또한, 데이터 파이프라인의 개발과 유지 관리를 간소화하고 데이터 호환성 문제, 데이터 손상 및 데이터 손실의 위험을 줄여준다
- 스키마 레지스트리를 통해 데이터 형식과 버전에 대한 스키마를 정의하고 레지스트리에 등록할 수 있다. 등록한 스키마는 여러 시스템과 애플리케이션에서 공유하고 재사용할 수 있습니다. 프로듀서가 메시지 브로커에 데이터를 보낼 때 데이터 스키마를 메시지 헤더에 포함하며, 스키마 레지스트리는 스키마가 유효하고 해당 토픽에 대한 스키마와 호환되는지 확인한다.

### references

- [Schema Registry Overview](https://docs.confluent.io/platform/current/schema-registry)
- [Schema Registry Github](https://github.com/confluentinc/schema-registry)