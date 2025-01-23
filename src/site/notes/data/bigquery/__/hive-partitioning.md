---
{"dg-publish":true,"permalink":"/data/bigquery/__/hive-partitioning/","tags":["bigquery","partitioning","hive"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgShowInlineTitle":true,"dgEnableSearch":true,"dgLinkPreview":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


그래서 하이브 파티셔닝이 뭔데?
BigQuery external 테이블 생성 시 hive partitioning 에 대해 고민했었고 정리할 필요가 있어 여러 곳에서 하이브 파티셔닝에 대해 정리한 내용을 가져왔다.


### in [hive documents](https://cwiki.apache.org/confluence/display/hive/tutorial)


> **Partitions**: Each Table can have one or more partition Keys which determines how the data is stored. Partitions—apart from being storage units—also allow the user to efficiently identify the rows that satisfy a specified criteria; for example, a date_partition of type STRING and country_partition of type STRING. Each unique value of the partition keys defines a partition of the Table. For example, all "US" data from "2009-12-23" is a partition of the page_views table. Therefore, if you run analysis on only the "US" data for 2009-12-23, you can run that query only on the relevant partition of the table, thereby speeding up the analysis significantly. Note however, that just because a partition is named 2009-12-23 does not mean that it contains all or only data from that date; partitions are named after dates for convenience; it is the user's job to guarantee the relationship between partition name and data content! Partition columns are virtual columns, they are not part of the data itself but are derived on load.

- 파티션은 저장 단위일 뿐만 아니라 사용자가 지정된 기준을 충족하는 행을 효율적으로 식별할 수 있게 도와준다
    - 문자열 유형의 date_partition (‘2023-11-22’, ‘2023-11-23’)
    - 문자열 유형의 country_partition (uk, ko, us …)
- 파티션 키의 각 고유 값은 테이블의 파티션을 의미한다
    - "2009-12-23"의 모든 "US" 데이터가 있다고 가정해보자.
    - 2009-12-23에 대한 "US" 데이터에 대해서만 분석을 실행하는 경우, 테이블의 관련 파티션에서만 해당 쿼리를 실행할 수 있으므로 분석 속도가 크게 향상된다.
    - 그러나 파티션 이름이 2009-12-23이라고 해서 해당 날짜의 데이터만 포함한다는 아니다. 파티션은 편의상 날짜의 이름으로 설정한다. 따라서 파티션 이름과 데이터 내용 간의 관계를 확인하는 것은 사용자의 몫이다.
    - 파티션은 가상 열이며, 데이터 자체의 일부가 아니라 로드 시 이용된다.


### in [duckdb documents](https://duckdb.org/docs/data/partitioning/hive_partitioning.html)


> - Hive partitioning is a [partitioning strategy](https://en.wikipedia.org/wiki/Partition_(database)) that is used to split a table into multiple files based on **partition keys**. The files are organized into folders. Within each folder, the **partition key** has a value that is determined by the name of the folder.
> - Below is an example of a hive partitioned file hierarchy. The files are partitioned on two keys (`year` and `month`).

- 하이브 파티셔닝은 **파티션 키**를 기반으로 테이블을 여러 개 파일로 분할하는 데 사용하는 [파티셔닝 전략](https://en.wikipedia.org/wiki/Partition_(데이터베이스))이다
    - 파일은 폴더로 구성되고,
    - 각 폴더 내 **파티션 키**는 폴더 이름에 따라 결정된다.
- 아래는 하이브 파티션 파일 계층 구조의 예시다. 파일은 두 개의 키(`연도`와 `월`)로 파티션 되어 있다.
```
orders
├── year=2021
│    ├── month=1
│    │   ├── file1.parquet
│    │   └── file2.parquet
│    └── month=2
│        └── file3.parquet
└── year=2022
   ├── month=11
   │   ├── file4.parquet
   │   └── file5.parquet
   └── month=12
       └── file6.parquet
```


### references


- https://cwiki.apache.org/confluence/display/hive/tutorial
- https://duckdb.org/docs/data/partitioning/hive_partitioning.html