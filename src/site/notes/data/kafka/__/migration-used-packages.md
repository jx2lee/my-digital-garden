---
{"dg-publish":true,"permalink":"/data/kafka/__/migration-used-packages/","tags":["kafka","connect"],"noteIcon":"","created":"2024-06-30T00:39:32.604+09:00"}
---



> [!tldr]
> 이관 시 사용한 connector 와 transform 을 소개한다.


### connector


**kafka-connect-bigquery**
- https://github.com/confluentinc/kafka-connect-bigquery
- confluent
    - https://docs.confluent.io/kafka-connectors/bigquery/current/overview.html
    - https://docs.confluent.io/kafka-connectors/bigquery/current/kafka_connect_bigquery_config.html
- 변경사항
    - topic2TableMap
        - 토픽:테이블 관계를 1:1 로 설정이 가능하다.
        - 이번 이관작업에서는 토픽:테이블 관계를 N:1 로 설정했다.
    - 다른 sink connector 에서도 [동일한 N:1 설정이 가능하도록 기능을 변경](https://github.com/snowflakedb/snowflake-kafka-connector/pull/459)했다.
    - ~~이에 [PR 생성](https://github.com/confluentinc/kafka-connect-bigquery/pull/361)하고 리뷰진행 중이다. 머지된 PR 날짜를 보니 패키지 의존성 문제가 아닌 경우 승인이 늦어지고 있다. (오픈된 PR 날짜도 굉장히 오래됨) 따라서 변경 후 빌드된 라이브러리만 사용해도 무방할 듯 하다.~~
        - PR 승인이 완료됐다.
        - cp-kafka-connect 차트에서 플러그인으로 설치 시 변경사항이 반영되었는지 확인할 필요가 있다. 만약 반영되어 있다면 kafka-connect-bigquery 레포는 이용하지 않아도 좋다.


### transforms


[ReplaceField](https://docs.confluent.io/platform/current/connect/transforms/replacefield.html#replacefield)
- kafka 바이너리에 포함된 디폴트 smt 패키지다.
- 특정 필드를 제외할 수 있는(exclude 옵션) 기능을 제공하여 수집하지 않아도 될 필드를 제외할 때 사용했다.


[kafka-connect-transform-tojsonstring](https://github.com/an0r0c/kafka-connect-transform-tojsonstring)
- 메시지의 key 혹은 value 값을 jsontring 으로 변환하는 smt 패키지다.
- 기능추가
    - 기존 코드에서는 메세지 key 혹은 value 값의 스키마가 없는 경우 패스한다.
    - 스키마가 없는 경우에도 transform 할 수 있도록 변경 요청하였다([PR](https://github.com/an0r0c/kafka-connect-transform-tojsonstring/pull/18)).


[kafka-connect-transform-common](https://github.com/jcustenborder/kafka-connect-transform-common)
- 컨플루언트에서 제공하는, 자주 사용되는 smt 패키지다.
- 메세지 헤더값을 field 로 추가해주는 [HeaderToField](https://jcustenborder.github.io/kafka-connect-documentation/projects/kafka-connect-transform-common/transformations/HeaderToField.html) 사용했다.


[TimestampConverter](https://docs.confluent.io/platform/current/connect/transforms/timestampconverter.html)
- 이벤트 발생 시각을 나타내느 ce_id 를 일 기준으로 파티셔닝을 설정했다.
- header 에서 string 으로 가져오기 때문에 timestamp 로 변경하는 기본 transform 을 이용했다.