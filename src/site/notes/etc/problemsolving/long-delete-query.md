---
{"author":"jx2lee","aliases":null,"created":"2026-06-18T18:45:55.000+09:00","last-updated":"2026-06-18 18:45","tags":["redshift","problem solving"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":false,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/problemsolving/long-delete-query/","dgPassFrontmatter":true,"noteIcon":""}
---


### Background

새벽 시간대 배치는 Oracle 원천 데이터를 Glue 를 통해 parquet 형태로 s3 에 떨구고 Redshift staging 테이블에 랜딩한 뒤, target 테이블에서 겹치는 데이터를 지우고 다시 적재하는 방식으로 돌아간다.

특히 KST 자정 근처에 실행되는 주문 테이블 작업은 시간대 데이터를 빨리 반영해야 한다. 지연이 길어지면 후속 배치와 데이터 신선도에 바로 영향이 간다. (서비스에 사용하는 테이블로 어느 시간대에도 지연이 발생하지 않아야한다..)

이번 디버깅은 자정 실행 건이 평소보다 훨씬 오래 걸리면서 시작됐다. Airflow task는 정상 종료됐지만 전체 수행 시간이 너무 길었다. Airflow queue 문제인지, Redshift WLM queue인지, S3 `COPY`인지, `DELETE` 쿼리인지, 아니면 같은 시간대의 Redshift 부하인지 나눠서 확인해야 했다.

### Problem

- 처음에는 Airflow나 Redshift WLM queue 대기를 의심했다. 그런데 확인 결과 WLM queue 대기는 거의 없었고, S3에서 staging 테이블로 적재하는 `COPY`도 약 4초에 끝났다.
- 병목은 staging 테이블을 기준으로 target 테이블의 기존 데이터를 삭제하는 `DELETE FROM ... USING staging ...` 구간이었다. 해당 쿼리는 약 2TB 이상을 스캔했고, 최종적으로 약 20만 rows를 삭제했다. (정확히는 삭제하고 변경된 데이터로 삽입한다. upsert 형태로 수집한다.)
- DELETE 시 원본 테이블의 블록 프루닝을 위해 사용한 `ORDER_DTM BETWEEN min/max` 조건이 문제는 아니었다. Redshift scan 정보상 range-restricted scan은 실제로 사용됐다. 문제는 staging 데이터의 `필터 컬럼` 최솟값이 오래된 날짜까지 내려가면서 min/max 범위가 넓어진 점이었다. pruning은 됐지만, 읽어야 하는 블록이 여전히 많았다.

여기에 외부 부하도 겹쳤다. 같은 시간대에 엄청나게 큰 materialized view auto refresh가 돌고 있었다. 이 작업은 매번 full recompute 형태로 실행되며 수십 TB를 스캔했다(심지어 미사용 테이블이었다..). 그래서 ODS delete 쿼리 plan 하나의 문제라기보다, 장시간 대용량 scan과 겹치면서 Redshift storage/I/O 경합이 커진 상황으로 봤다.

### Action

다음 순서로 문제를 좁혔다.

1. Airflow task 로그와 Redshift query 이력을 비교해 실제로 오래 걸린 구간을 찾았다.
2. `stl_wlm_query`로 WLM queue 대기 여부를 확인했다.
3. `svl_query_summary`, `stl_scan`, `svl_query_report`를 통해 scan bytes, row count, slice별 처리량을 확인했다.
4. target table scan에서 range-restricted scan이 사용됐는지 확인했다.
5. 같은 시간대에 실행 중이던 장시간 Redshift query와 materialized view refresh 이력을 확인했다.

그 결과 MV 의 auto refresh가 핵심 적재 시간대와 겹치는 것이 가장 큰 리스크로 보였다. 해당 materialized view는 자동 refresh 때마다 full recompute를 반복했고, 사용 패턴에 비해 운영 클러스터에 주는 scan 부하가 컸다.

조치로 auto refresh를 비활성화하고, 필요하다면 off-peak 시간대에 명시적으로 refresh하는 방향을 검토했다.

```sql
ALTER MATERIALIZED VIEW {schema}.{table_name} AUTO REFRESH NO;
```

delete 쿼리는는 `ORDER_DTM` 조건을 유지했다. 이번 조사에서 해당 조건이 실제로 pruning을 유도한다는 점이 확인됐기 때문이다. 대신 staging 데이터의 `order_dtm` 범위가 과도하게 넓어지는 경우를 감지할 수 있도록 staging row count, `order_dtm` span, delete duration, scan bytes를 함께 모니터링하기로 했다.

### Outcome

조치 후 새벽 시간대 배치 작업(주문 테이블의 delete insert 쿼리)의 평균 수행 시간은 기존 8-9분대에서 4-5분대로 줄었다.

- 특정 SQL 한 줄을 튜닝했다기보다는, 핵심 적재 시간대에 Redshift 클러스터에서 발생하던 불필요한 대용량 scan을 분리한 효과가 컸다. 덕분에 랜딩 시간이 짧아졌고, 시간대별 변동성도 줄었다.
- 이번 경험을 통해 지연 원인도 Airflow queue나 S3 `COPY`가 아니라 Redshift 내부 delete scan과 동시에 실행되던 materialized view full refresh 부하로 좁힐 수 있었다.

### Aftermath & Lessons Learned

- WLM queue 대기가 0이라고 해서 Redshift 클러스터에 병목이 없다는 뜻은 아니다. queue에 쌓이지 않더라도 동시에 실행되는 대용량 scan 때문에 storage 또는 I/O 경합이 발생할 수 있다.
- range pruning이 동작해도 충분히 빠르다는 보장은 없다. min/max 조건이 너무 넓어지면 pruning이 적용되더라도 여전히 큰 범위를 읽게 된다. 그래서 쿼리 plan만 볼 것이 아니라 실제 staging 데이터의 시간 범위와 scan bytes를 같이 봐야 한다.
  - 더불어 min max 필터컬럼을 스칼라 서브쿼리로 사용하지 않으니 실행계획이 잘 잡히지 않았다. 옵티마이저는 똑똑해 라는 내 편견을 깼다. 타임스탬프 리터럴로 줄 수 있는 방법을 고민하다 다행히 스칼라 서브쿼리로 문제를 해결했다.
- materialized view auto refresh는 운영 편의성이 있지만, full recompute가 반복되면 핵심 배치 시간대에 큰 부하가 될 수 있다. 자동refresh 일수록 실행 시간, scan 규모, 실제 사용처를 주기적으로 점검해야 한다. 그리고 materialized view 는 제어가 필요한 테이블이다. 이런 걸 모르는 사용자들의 잘못된 패턴으로 생성한 mv 가 클러스터 리소스를 불필요하게 많이 사용할 수 있다.
- 이번 대응에서 얻은 가장 큰 교훈은 느린 쿼리 하나만 봐서는 부족하다는 점이다. 중요한 배치 시간대에는 어떤 작업들이 Redshift 리소스를 함께 쓰는지까지 같이 봐야 한다.


### References
- Factors affecting query performance - Amazon Redshift
    Dataset size와 concurrent operations가 query performance에 영향을 준다는 근거로 쓰기 좋음. (docs.aws.amazon.com
    (https://docs.aws.amazon.com/redshift/latest/dg/c-query-performance.html))
- Workload management - Amazon Redshift
    WLM이 concurrent queries와 workload resource allocation을 관리한다는 근거. 특히 장시간 cluster resource를 쓰는 query가 다른 query 성능에 영향을 줄 수
    있다는 설명이 있음. (docs.aws.amazon.com (https://docs.aws.amazon.com/redshift/latest/dg/cm-c-implementing-workload-management.html))
- Implementing manual WLM - Amazon Redshift
    query queue, slot, memory allocation, concurrency가 성능에 미치는 영향 설명. disk I/O가 performance를 degrade할 수 있다는 문구도 있음.
    (docs.aws.amazon.com (https://docs.aws.amazon.com/redshift/latest/dg/cm-c-defining-query-queues.html))
- Refreshing a materialized view - Amazon Redshift
    MV full refresh가 underlying SQL을 다시 실행한다는 점, autorefresh가 system load/resources를 고려한다는 점, 2026-02-27 이후 Auto REFRESH가 user query와
    같은 priority로 실행된다는 근거. (docs.aws.amazon.com (https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-refresh.html))