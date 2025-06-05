---
{"dg-publish":true,"permalink":"/data/kafka/__/cdc-research/","dgPassFrontmatter":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


BigQuery 실시간 파이프라인의 가능한 CDC 도구를 살펴보고 비교해본다.

### CDC (Change Data Capture)
#### overview
> Change data capture (CDC) refers to the tracking of all changes in a data source (databases, data warehouses, etc.) so they can be captured in destination systems. In short, CDC allows organizations to achieve data integrity and consistency across all systems and deployment environments. Additionally, it allows organizations to use the right tool for the right job by moving data from legacy databases to purpose-built data platforms, such as document or search databases and data warehouses.

- 데이터 소스(데이터베이스, 데이터 웨어하우스 등)의 모든 변경 사항을 추적하여 캡처할 수 있도록 하는 것을 말함
- CDC를 통해 조직은 모든 시스템과 배포 환경 전반에서 데이터 무결성과 일관성을 달성할 수 있음
	- 또한, 데이터를 document or search 데이터베이스와 데이터 웨어하우스와 같은 데이터 플랫폼으로 이동하여 업무에 적합한 도구를 사용할 수 있도록 지원함
- ![Why Change Data Capture?-confluent](https://i.imgur.com/bwN2IAG.png)
	- [confluent document](https://images.ctfassets.net/8vofjvai1hpv/1ZfXCGSQCX4bEXcMrc64Sp/9f6a2dce723e4af053955d51eab7e2b3/1-narrow.png?w=851&h=431&q=100&fm=webp&bg=transparent)
- CDC 를 왜 사용하는가
	- 처음 CDC는 ETL 을 위해(to 데이터 웨어하우스) 데이터 복제의 대체 솔루션으로 인기를 얻음
	- 최근 CDC는 클라우드로 마이그레이션하는 `de facto`
- 동작 원리
	- 소스 데이터베이스(일반적으로 MySQL, Microsoft SQL, Oracle 또는 PostgreSQL 와 같은 관계형 데이터베이스)에서 데이터가 변경되면(insert, update or delete) 캐시, 검색 인덱스, 데이터 웨어하우스 또는 데이터 레이크와 같은 시스템으로 전파되어야 함
	- 일반적으로 CDC 는 push 와 pull 두 가지 방식이 존재
	- 소스 데이터베이스가 다운스트림 서비스 및 애플리케이션에 업데이트를 푸시하거나, 다운스트림 서비스 및 애플리케이션이 고정된 간격(fixed interval)으로 폴링하여 업데이트된 데이터를 가져옴
	- 각 방식에는 고유한 장단점이 있기 때문에 어떤 방식을 사용할 지 선택하는 것은 중요함

#### change data capture patterns
**Timestamp-based**
![|600](https://i.imgur.com/NEMNhyd.png)
- 테이블 가장 최근 변경된 시간을 반영하는 열을 추가할 수 있음
- LAST_MODIFIED, LAST_UPDATED 등으로 불리며 이 필드를 이용해 마지막 실행 시간 이후 업데이트된 레코드를 가져올 수 있음

**Trigger-based**
![|600](https://i.imgur.com/NEU7ueT.png)
- 테이블당 각 작업의 트리거를 이용함
	- 대부분의 데이터베이스는 트리거 함수를 지원
	- 트리거 함수는 테이블에서 레코드 삽입, 업데이트 또는 삭제와 같은 특정 이벤트가 발생하면 자동으로 실행되는 저장 프로시저
- 별도의 테이블(shadow table or event table)에 저장
- 메시징 시스템을 이용하여 데이터 변경 사항을 관련 대상 시스템이 구독(subscribe)하는 대기열에 publish 할 수 있음


### reference
- [confluent document](https://www.confluent.io/learn/change-data-capture/)