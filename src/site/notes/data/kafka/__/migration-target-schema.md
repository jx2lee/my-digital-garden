---
{"author":"jx2lee","aliases":"빅쿼리로 적재할 테이블 스키마","created":"2024-06-30T00:39:32.000+09:00","last-updated":"2023-10-24 17:30","tags":["kafka","connect","bigquery"],"dg-publish":true,"permalink":"/data/kafka/__/migration-target-schema/","dgPassFrontmatter":true,"noteIcon":""}
---


> [!tldr] 서버로그 이벤트 메세지를 적재할 테이블을 정의한다.

### DDL


```sql
CREATE TABLE
  datasets.serverlog_kafka (
    ce_type STRING,
    ce_source STRING,
    ce_specversion STRING,
    ce_time TIMESTAMP,
    ce_id STRING,
    ce_subject STRING,
    value STRING
)
PARTITION BY
  TIMESTAMP_TRUNC(ce_time, DAY);
```


### notice

- 이벤트 헤더에 저장된 메세지 메타정보를 풀어 저장한다. (ce_ prefix columns)
    - [BigQuery Sink Connector](https://docs.confluent.io/kafka-connectors/bigquery/current/overview.html) 는 At least once delivery 를 보장한다.
    - 이벤트 중복 적재를 피하기 위해 Transform 과정에서 ce_id 를 이용할 수 있다.
- 이벤트 value 값들을 jsonstring 으로 뭉개어 저장한다.
    - Transform 과정에서 필요한 값들을 가져갈 수 있도록 한다.
- paritioned table 로 생성한다. (key: ce_time)
    - 이벤트 발생 시각을 나타낸다.
    - `_PARTITIONTIME` 보다 이벤트 발생 시각(ce_time)을 파티셔닝 키로 설정한다.
        - 왜? 빅쿼리 적재 시간(ingestion time) 보다 이벤트 발생 시간이 더 적절하다. 파이프라인이 고장난 경우 다시 살아나기까지 텀이 발생할 수 있다.
        - ingestion time 이 실제 이벤트 발생시각보다 하루 넘게 차이날 수 있다.
