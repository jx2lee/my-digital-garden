---
{"dg-publish":true,"permalink":"/data/kafka/__/migration-to-kakfa-connect/","tags":["kafka","connect"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":"","created":"2024-06-30T00:39:32.604+09:00"}
---



> [!tldr] 
> kafka connect 를 이용한 파이프라인 마이그레이션 (kinesis & lambda to kafka connect)


### 배경


- 백엔드 회원 파트에 EDA 작업이 완료되었다. event driven architecture 로 전환하며 카프카를 활용한다.
- AWS Kinesis 로 발행하는 이벤트는 이후 deprecated 될 예정이다. Kinesis 로 발행한 메세지는 lambda 를 통해 BigQuery 에 실시간으로 적재되고 있다.
- 이에 맞춰 Kafka 메세지를 BigQuery 로 적재하도록 이사가 필요하다.


### 목적


**Kinesis & Lambda 서버로그 파이프라인을 kafka connect 로 이관한다.**


### 개선 지표


TBD


### 목표


- 회원 관련 이벤트를 kafka connect 를 이용한 파이프라인으로 이관한다.
- 적재실패 시 모니터링 & 재처리 로직을 설계한다.
    - ex. Kinesis lambda 파이프라인의 경우 적재를 시도하는 람다 실패 시 슬랙으로 알람을 발송하고 있다.


### 더보기


- [[data/kafka/__/migration-target-schema\|빅쿼리로 적재할 테이블 스키마]]
- [[data/kafka/__/migration-used-packages\|사용중인 커넥터 & 트랜스폼]]
- [[data/kafka/__/migration-error-handling\|적재 실패 메세지의 처리 방안]]
- [[data/kafka/__/migration-connector-cicd\|커넥터 CI/CD]]
