---
{"author":"jx2lee","aliases":null,"created":"2023-12-20T00:33:04.000+09:00","last-updated":"2023-09-18 14:28","tags":null,"dg-publish":true,"permalink":"/etc/__/lakehouse-tool-localtest/","dgPassFrontmatter":true,"noteIcon":""}
---

> Data Lakehouse 테스트한 내용들을 간략히 정리하고 공유해보자

## iceberg

### quickstart - hive

[Hive and Iceberg Quickstart](https://iceberg.apache.org/hive-quickstart/) 링크를 참고해 튜토리얼을 진행했다. 공식 문서의 예시는 embedded metastore 를 derby 를 이용해 hive 를 실행하고 iceberg 테이블을 생성한다. 컨테이너를 재시작하면 hiveserver2 서비스가 계속 죽기 때문에 hive 저장소에서 제공하는 compose 파일을 조금 수정했다.

- hive 에 대한 설명은 하위 링크를 참고하자
    - https://cwiki.apache.org/confluence/display/hive/design
    - https://github.com/apache/hive
    - https://cwiki.apache.org/confluence/display/Hive/GettingStarted
- `Hive Metastore functions as the Iceberg catalog to locate Iceberg files, which can be anywhere.`
    - TODO: **s3 연동 테스트**

<details>
<summary>docker-compose-hive.yml</summary>
```yaml
version: '3.9'
name: iceberg-hive-quickstart
services:
  postgres:
    image: postgres
    restart: unless-stopped
    container_name: postgres
    hostname: postgres
    environment:
      POSTGRES_DB: 'metastore_db'
      POSTGRES_USER: 'hive'
      POSTGRES_PASSWORD: 'password'
    ports:
      - '5432:5432'
    volumes:
      - hive-db:/var/lib/postgresql

  metastore:
    image: apache/hive:4.0.0-alpha-1
    depends_on:
      - postgres
    restart: unless-stopped
    container_name: metastore
    hostname: metastore
    environment:
      DB_DRIVER: postgres
      SERVICE_NAME: 'metastore'
      SERVICE_OPTS: '-Xmx1G -Djavax.jdo.option.ConnectionDriverName=org.postgresql.Driver
                     -Djavax.jdo.option.ConnectionURL=jdbc:postgresql://postgres:5432/metastore_db
                     -Djavax.jdo.option.ConnectionUserName=hive
                     -Djavax.jdo.option.ConnectionPassword=password'
    ports:
        - '9083:9083'
    volumes:
        - warehouse:/opt/hive/data/warehouse

  hiveserver2:
    image: apache/hive:4.0.0-alpha-1
    depends_on:
      - metastore
    restart: unless-stopped
    container_name: hiveserver2
    environment:
      HIVE_SERVER2_THRIFT_PORT: 10000
      SERVICE_OPTS: '-Xmx1G -Dhive.metastore.uris=thrift://metastore:9083'
      IS_RESUME: 'true'
      SERVICE_NAME: 'hiveserver2'
    ports:
      - '10000:10000'
      - '10002:10002'
    volumes:
      - warehouse:/opt/hive/data/warehouse

volumes:
  hive-db:
  warehouse:
```

</details>

iceberg 테이블 포맷으로 테이블을 생성하기 전 data directory 는 다음과 같다.

```bash
hive@9cb7504b7cc7:/opt/hive/data/warehouse$ pwd
/opt/hive/data/warehouse
hive@9cb7504b7cc7:/opt/hive/data/warehouse$ ls -alF
total 0
drwxr-xr-x 1 hive root  0 Aug 25 12:35 ./
drwxr-xr-x 1 root root 18 Aug 25 12:35 ../
```

nyc 데이터베이스를 생성하고 taxis 테이블을 생성하면 data directory 에 다음과 같은 파일들이 쌓인다.

```sql
CREATE DATABASE nyc;
CREATE TABLE nyc.taxis
(
  trip_id bigint,
  trip_distance float,
  fare_amount double,
  store_and_fwd_flag string
)
PARTITIONED BY (vendor_id bigint) STORED BY ICEBERG;
```

nyc 데이터베이스를 생성하면 warehouse 밑 nyc.db 폴더가 생성된다.

```bash
hive@9cb7504b7cc7:/opt/hive/data/warehouse/nyc.db$ pwd
/opt/hive/data/warehouse/nyc.db
hive@9cb7504b7cc7:/opt/hive/data/warehouse/nyc.db$ ls -alF
total 0
drwxr-xr-x 1 hive hive  0 Sep 18 07:23 ./
drwxr-xr-x 1 hive root 12 Sep 18 07:23 ../
```

테이블을 생성하면 iceberg table format 으로 메타데이터가 생성된다.

```bash
hive@9cb7504b7cc7:/opt/hive/data$ tree .
.
└── warehouse
    └── nyc.db
        └── taxis
            └── metadata
                └── 00000-1732b0ca-d725-47d6-a06c-a1bfb2efa983.metadata.json
hive@9cb7504b7cc7:/opt/hive/data/warehouse/nyc.db/taxis/metadata$ cat 00000-1732b0ca-d725-47d6-a06c-a1bfb2efa983.metadata.json
{
  "format-version" : 1,
  "table-uuid" : "e1c15319-c94a-44b2-814c-a09f8355fe54",
  "location" : "file:/opt/hive/data/warehouse/nyc.db/taxis",
  "last-updated-ms" : 1695027274973,
  "last-column-id" : 5,
  "schema" : {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "trip_id",
      "required" : false,
      "type" : "long"
    }, {
      "id" : 2,
      "name" : "trip_distance",
      "required" : false,
      "type" : "float"
    }, {
      "id" : 3,
      "name" : "fare_amount",
      "required" : false,
      "type" : "double"
    }, {
      "id" : 4,
      "name" : "store_and_fwd_flag",
      "required" : false,
      "type" : "string"
    }, {
      "id" : 5,
      "name" : "vendor_id",
      "required" : false,
      "type" : "long"
    } ]
  },
  "current-schema-id" : 0,
  "schemas" : [ {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "trip_id",
      "required" : false,
      "type" : "long"
    }, {
      "id" : 2,
      "name" : "trip_distance",
      "required" : false,
      "type" : "float"
    }, {
      "id" : 3,
      "name" : "fare_amount",
      "required" : false,
      "type" : "double"
    }, {
      "id" : 4,
      "name" : "store_and_fwd_flag",
      "required" : false,
      "type" : "string"
    }, {
      "id" : 5,
      "name" : "vendor_id",
      "required" : false,
      "type" : "long"
    } ]
  } ],
  "partition-spec" : [ {
    "name" : "vendor_id",
    "transform" : "identity",
    "source-id" : 5,
    "field-id" : 1000
  } ],
  "default-spec-id" : 0,
  "partition-specs" : [ {
    "spec-id" : 0,
    "fields" : [ {
      "name" : "vendor_id",
      "transform" : "identity",
      "source-id" : 5,
      "field-id" : 1000
    } ]
  } ],
  "last-partition-id" : 1000,
  "default-sort-order-id" : 0,
  "sort-orders" : [ {
    "order-id" : 0,
    "fields" : [ ]
  } ],
  "properties" : {
    "engine.hive.enabled" : "true",
    "bucketing_version" : "2",
    "storage_handler" : "org.apache.iceberg.mr.hive.HiveIcebergStorageHandler",
    "serialization.format" : "1"
  },
  "current-snapshot-id" : -1,
  "refs" : { },
  "snapshots" : [ ],
  "statistics" : [ ],
  "snapshot-log" : [ ],
  "metadata-log" : [ ]
}
```

- 위 메타데이터 json 말고 dot crc 파일(`.00000-1732b0ca-d725-47d6-a06c-a1bfb2efa983.metadata.json.crc`)도 함께 생성된다. 
    - 일반적으로 CRC는 파일 포맷 자체가 아니라 데이터 무결성을 검사하기 위한 알고리즘 또는 기술로 사용된다고 한다.

테이블에 값을 삽입하면 아래와 같이 여러 메타정보들이 생성되는 것을 확인할 수 있다.

```bash
hive_warehsoue
└── nyc.db
    ├── _tmp.taxis
    └── taxis
        ├── data
        │   ├── vendor_id=1
        │   │   └── 00000-0-data-hive_20230918092244_a25b33bb-b233-41cd-a330-61140063ee35-job_16950289664771_0001-1-00001.parquet
        │   └── vendor_id=2
        │       └── 00000-0-data-hive_20230918092244_a25b33bb-b233-41cd-a330-61140063ee35-job_16950289664771_0001-1-00002.parquet
        ├── metadata
        │   ├── 00000-1732b0ca-d725-47d6-a06c-a1bfb2efa983.metadata.json
        │   ├── 00001-e4db34cd-ed3a-44f8-b3fa-1c4c47d19a02.metadata.json
        │   ├── bc13d33a-7117-414f-b1d1-5a6a5b522604-m0.avro
        │   └── snap-3573650824477192202-1-bc13d33a-7117-414f-b1d1-5a6a5b522604.avro
        ├── stats
        │   └── default_iceberg.nyc.taxis3573650824477192202
        └── temp

9 directories, 7 files
```

### quickstart - spark

[Spark and Iceberg Quickstart](https://iceberg.apache.org/spark-quickstart/) 링크를 참고하여 튜토리얼을 진행했다.
- 

### references
- [iceberg](https://iceberg.apache.org/)
    - [github](https://github.com/apache/iceberg)
    - quickstart
        - [hive](https://iceberg.apache.org/hive-quickstart/)
        - [spark](https://iceberg.apache.org/spark-quickstart/)
- [dremio](https://docs.dremio.com/)
    - [github](https://github.com/dremio/dremio-oss)
    - [quickstart](https://docs.dremio.com/current/)