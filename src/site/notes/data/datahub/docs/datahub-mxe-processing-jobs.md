---
{"dg-publish":true,"permalink":"/data/datahub/docs/datahub-mxe-processing-jobs/","noteIcon":"","created":"2024-06-30T00:39:32.595+09:00"}
---

#datahub #glossary #내가헷갈리는용어를정리하자 

> **MXE 는 datahub 에서 사용하는 MAE & MCE 를 함께 부르기 위해 가운데 문자를 X 로 표현했다.**


DataHub 는 백엔드에서 pub-sub 메세지 큐로 kafka 를 사용한다. 사용하는 kafka 토픽으로는 크게 `MetadataChangeEvent` 와 `MetadataAuditEvent` 가 있다.
- `MetadataChangeEvent`: 메타데이터 변경이 있는 모든 데이터 플랫폼 또는 크롤러(*여기서 크롤러는 적재하고자 하는 메타데이터를 전송하는 주체라고 생각하자*)에서 전송한 메세지를 갖는 토픽
	- **ingesiont framework 아키텍처의 kafka**
- `MetadataAuditEvent`: DataHub GMS 에서 변경된 메타데이터의 내용을 담은 메세지를 갖는 토픽
	- **serving-tier 아키텍처의 kafka**

위 두가지 토픽을 소비할 수 있도록 Datahub 에서는 다음 2가지 Spring Job 이 존재한다.
- MCE Consumer Job: Datahub GMS 로 적재
- MAE Consumer Job: ElasticSearch & Neo4j 로 적재