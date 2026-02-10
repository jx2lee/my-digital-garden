---
{"dg-publish":true,"permalink":"/data/etc/getting-started/sqlmesh/","dgPassFrontmatter":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


### sqlmesh

SQLMesh 는 데브옵스 모범 사례를 데이터 팀에 제공하는 오픈 소스 데이터옵스 프레임워크이다. 데이터 사이언티스트, 애널리스트, 엔지니어가 SQL 또는 Python으로 작성된 데이터 transformation 을 효율적으로 실행하고 배포할 수 있게 한다.

> 이 프레임워크는 Airbnb, Apple, Netflix의 데이터 리더들이 설립한 Tobiko Data에서 만들고 유지 관리한다고 한다. 엔지니어 출신들을 확인해보니 맞다.

### why

데이터 크기가 커지거나 데이터 사용자 수가 늘어날 때 데이터 팀이 직면하는 몇 가지 챌린지를 다음과 같이 설명한다.

- **Data pipelines are fragmented and fragile (파편화되고 취약한 데이터 파이프라인)**
	- 데이터 파이프라인은 일반적으로 테이블을 통해 암시적으로 서로 의존하는 Python 또는 SQL 스크립트로 구성된다. downstream 종속성을 깨뜨리는 업스트림 스크립트의 변경 사항은 일반적으로 런타임에만 감지된다.
**- Data quality checks are not sufficient (데이터 품질 검사만으로는 충분하지 않음)**
	- 데이터 커뮤니티는 데이터 파이프라인을 테스트하기 위한 '솔루션'으로 데이터 품질 검사를 이용했다. 데이터 품질 검사는 예기치 않은 대규모 데이터 변경을 감지하는 데는 훌륭하지만, 실행 비용이 많이 들고 정확한 로직의 유효성을 검사하는 데는 어려움이 있다.
- **It's too hard and too costly to build staging environments for data (데이터를 위한 준비 환경을 구축하는 것은 어렵고 많은 비용 발생)**
	- 프로덕션에 배포하기 전 데이터 파이프라인의 변경 사항을 검증하는 것은 불확실하고 때로는 비용이 많이 든다. 브랜치를 환경에 배포할 수는 있지만, 프로덕션에 병합하면 코드를 다시 실행해야 한다. 이는 낭비이며 데이터가 다시 생성되기 때문에 불확실성(uncertainty)이 발생합니다.
- **Silos transform data lakes to data swamps (데이터 호수를 데이터 늪으로 만드는 사일로)**
	- 핵심 파이프라인을 변경하는 데 드는 어려움과 비용으로 인해 파이프라인이 중복될 수 있다. 쉽게 변경하고 검증할 수 없기 때문에 사용자들은 `저항이 가장 적은 경로(path of least resistence)`를 따르게 된다. 유사한 테이블 확산은 추가 비용, 불일치 및 유지 관리 부담으로 이어진다.

### what

SQLMesh는 데이터 파이프라인을 쉽고 효율적이며 안전하게 개발 및 배포할 수 있도록 CLI, Python API, 웹 UI로 구성된다.

**핵심 원칙** 3가지는 다음과 같다.
- 정확성은 타협할 수 없다 (Correctness is non-negotiable)
	- 잘못된 데이터는 데이터가 없는 것보다 더 나쁘다. SQLMesh 는 협업이 많은 환경에서도 데이터의 일관성을 보장한다.
- 자신감 있는 변화 (Change with confidence)
	- SQLMesh 는 변경으로 인한 영향을 요약하고 자동화된 가드레일을 제공하여, 사용자들이 안전하고 신속하게 기여할 수 있도록 지원한다.
- 복잡성 없는 효율성 (Efficiency without complexity)
	- SQLMesh 는 테이블을 재사용하고 계산을 최소화하여, 워크로드를 자동으로 최적화하므로 시간과 비용을 절약할 수 있다.

### key features
- 효율적인 개발/스테이징 환경
	- SQLMesh는 뷰를 사용하여 가상 데이터 마트를 구축하므로 변경 사항을 원활하게 롤백하거나 롤포워드할 수 있다. 저렴한 포인터 스왑을 통해 '스테이징' 데이터를 프로덕션 환경에서 재사용하므로 유효성 검사 목적으로 실행하는 모든 데이터 계산을 낭비되지 않도록 한다. 즉, 데이터 탐색과 변경 사항 미리 보기를 재미있고 안전하게 수행할 수 있는 무제한 복사-온-쓰기(copy-on-write) 환경을 제공한다.
- SQL 또는 Python 스크립트를 분석하고 이해하여 자동 DAG 생성
	- 종속성에 수동으로 태그를 지정할 필요가 없다. SQLMesh는 전체 데이터 웨어하우스의 종속성 그래프를 이해할 수 있는 기능으로 구축되었다.
- 변경 사항 요약
	- SQLMesh가 변경된 내용을 파악하여 영향을 받는 작업의 전체 그래프를 표시한다.
- CI 실행 가능 단위 및 통합 테스트
	- YAML에서 쉽게 정의하고 CI에서 실행할 수 있다. SQLMesh는 선택적으로 쿼리를 DuckDB 로 트랜스파일하여 테스트를 독립적으로 수행할 수 있다.
- 스마트한 변경 범주화 (smart change categorization)
	- 열 수준 계보가 변경 사항이 "중단" 또는 "비중단"인지 여부를 자동으로 결정하여 변경 사항을 올바르게 분류하고 값비싼 백필을 스킵할 수 있다.
- 간편한 증분 로드
	- 테이블을 증분식으로 로드하는 것은 전체 새로 고침만큼 쉽다. SQLMesh는 로드가 필요한 간격을 추적하는 복잡한 작업을 투명하게 처리하므로 날짜 필터를 지정하기만 하면 된다.
- Airflow 통합
	- 내장된 간단한 스케줄러로 작업을 예약하거나 기존 Airflow 클러스터를 사용할 수 있다. SQLMesh는 동적으로 Airflow DAG를 생성하고 푸시할 수 있다. 향후에는 Dagster 및 Prefect와 같은 다른 스케줄러도 지원할 예정이다.
- 노트북 / CLI
	- 익숙한 도구로 SQLMesh 를 이용할 수 있다.
- 개발중
	- 웹 기반 IDE
		- 브라우저에서 쿼리를 편집, 실행 및 시각화할 수 있다.
	- Github CI/CD 봇
		- 코드를 데이터에 직접 연결하는 봇입니다.
	- 테이블/열 수준 lineage 시각화
		- 열 전체 계보와 변환 순서를 빠르게 이해할 수 있다.

### quickstart
https://sqlmesh.readthedocs.io/en/latest/quick_start/ 을 참고하여 진행했다.

- sqlmesh 프로젝트 초기화
```bash
> sqlmesh plan
======================================================================
Successfully Ran 1 tests against duckdb
----------------------------------------------------------------------
New environment `prod` will be created from `prod`
Summary of differences against `prod`:
└── Added Models:
    ├── sqlmesh_example.example_incremental_model
    └── sqlmesh_example.example_full_model
Models needing backfill (missing dates):
├── sqlmesh_example.example_incremental_model: (2020-01-01, 2023-04-03)
└── sqlmesh_example.example_full_model: (2023-04-03, 2023-04-03)
Apply - Backfill Tables [y/n]: y
sqlmesh_example.example_incremental_model ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 1/1 • 0:00:00
       sqlmesh_example.example_full_model ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 1/1 • 0:00:00

All model batches have been executed successfully

Virtually Updating 'prod' ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 0:00:00

The target environment has been updated successfully

```
- dev 환경을 위한 plan 생성 (default: prod)
```zsh
> sqlmesh plan dev
======================================================================
Successfully Ran 1 tests against duckdb
----------------------------------------------------------------------
New environment `dev` will be created from `prod`
Apply - Virtual Update [y/n]: y
Virtually Updating 'dev' ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 0:00:00

The target environment has been updated successfully


Virtual Update executed successfully
```
- models/example_incremental_model.sql 에 명시한 테이블의 칼럼을 추가한다. (`new_column`)
- dev 플랜을 재실행하면 전 스냅샷과 비교하여 변경된 사항을 알려주고 backfill 여부를 확인한다.
```zsh
> sqlmesh plan dev
======================================================================
Successfully Ran 1 tests against duckdb
----------------------------------------------------------------------
Summary of differences against `dev`:
├── Directly Modified:
│   └── sqlmesh_example.example_incremental_model
└── Indirectly Modified:
    └── sqlmesh_example.example_full_model
---                                                                                                                       
                                                                                                                          
+++                                                                                                                       
                                                                                                                          
@@ -1,6 +1,7 @@                                                                                                           
                                                                                                                          
 SELECT                                                                                                                   
   id,                                                                                                                    
   item_id,                                                                                                               
+  'z' AS new_column,                                                                                                     
   ds                                                                                                                     
 FROM (VALUES                                                                                                             
   (1, 1, '2020-01-01'),                                                                                                  
Directly Modified: sqlmesh_example.example_incremental_model (Non-breaking)
└── Indirectly Modified Children:
    └── sqlmesh_example.example_full_model
Models needing backfill (missing dates):
└── sqlmesh_example.example_incremental_model: (2020-01-01, 2023-04-03)
Enter the backfill start date (eg. '1 year', '2020-01-01') or blank for the beginning of history: 
Enter the backfill end date (eg. '1 month ago', '2020-01-01') or blank to backfill up until now: 
Apply - Backfill Tables [y/n]: y
sqlmesh_example.example_incremental_model ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 1/1 • 0:00:00

All model batches have been executed successfully

Virtually Updating 'dev' ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 0:00:00

The target environment has been updated successfully
```

#### integration

현 버전에서는 airflow, dbt, github acitons, execution engine([BigQuery](https://sqlmesh.readthedocs.io/en/latest/integrations/engines/#bigquery---airflow-scheduler), [Databricks](https://sqlmesh.readthedocs.io/en/latest/integrations/engines/#databricks---airflow-scheduler),[Redshift](https://sqlmesh.readthedocs.io/en/latest/integrations/engines/#redshift---airflow-scheduler),[Snowflake](https://sqlmesh.readthedocs.io/en/latest/integrations/engines/#snowflake---airflow-scheduler),[Spark](https://sqlmesh.readthedocs.io/en/latest/integrations/engines/#spark---airflow-scheduler)) 와의 integration 을 지원한다. 

### reviews

sqlmesh 의 가장 큰 장점은 **가상(virtual) 데이터 마트**다.
- 각 환경별 plan 을 실행하고 duckdb 로 조회해보니, physical 테이블 이외 각 환경별 schema 가 생성되고 작업자가 정의한 모델은 view 로 생성되었다.
- 스키마 정보를 보면 \_\_{environment} 로 plan 별 view 테이블을 확인할 수 있다.
- prod 환경에서는 **table materialization** 이며 가상화를 view 를 이용했다.
- 활용방안
	- prd 데이터를 이용해서 테스트 해볼 수 있다.
		- dev 데이터를 따로 수집하지 않고 prd 데이터로 테스트 환경 구축 가능
	- prod plan 인 경우 materialziation 이 가능하기 때문에 비용 절감이 가능하다. 

```sql
D select table_schema, table_name, table_type from information_schema.tables;
┌──────────────────────┬───────────────────────────────────────────────────────┬────────────┐
│     table_schema     │                      table_name                       │ table_type │
│       varchar        │                        varchar                        │  varchar   │
├──────────────────────┼───────────────────────────────────────────────────────┼────────────┤
│ sqlmesh              │ sqlmesh_example__example_full_model__669545005__temp  │ BASE TABLE │
│ sqlmesh              │ sqlmesh_example__example_incremental_model__798825632 │ BASE TABLE │
│ sqlmesh              │ sqlmesh_example__example_full_model__3634844499       │ BASE TABLE │
│ sqlmesh              │ sqlmesh_example__example_incremental_model__990679052 │ BASE TABLE │
│ sqlmesh              │ _environments                                         │ BASE TABLE │
│ sqlmesh              │ _snapshots                                            │ BASE TABLE │
│ sqlmesh              │ sqlmesh_example__example_incremental_model__312832206 │ BASE TABLE │
│ sqlmesh              │ sqlmesh_example__example_full_model__1819925695       │ BASE TABLE │
│ sqlmesh_example      │ example_full_model                                    │ VIEW       │
│ sqlmesh_example      │ example_incremental_model                             │ VIEW       │
│ sqlmesh_example__dev │ example_incremental_model                             │ VIEW       │
│ sqlmesh_example__dev │ example_full_model                                    │ VIEW       │
│ sqlmesh_example__stg │ example_incremental_model                             │ VIEW       │
│ sqlmesh_example__stg │ example_full_model                                    │ VIEW       │
├──────────────────────┴───────────────────────────────────────────────────────┴────────────┤
│ 14 rows                                                                         3 columns │
└───────────────────────────────────────────────────────────────────────────────────────────┘

```

(레딧 질문 참고) dbt 의 경쟁으로 생각해야하냐는 [의견](https://www.reddit.com/r/dataengineering/comments/124tspm/comment/je15h4t/?utm_source=share&utm_medium=web2x&context=3)에는 두 제품 방향은 다르다고 이야기한다. dbt 는 transformation 도구로, sqlmesh 는 dataops 도구로 설명을 한다. sqlmesh 에서 내세우는 dbt 와의 차이점의 자세한 내용은 [공식 도큐먼트](https://sqlmesh.readthedocs.io/en/stable/comparisons/)에서 확인할 수 있다.
- **우선 sqlmesh 의 목적은 dbt 와의 호환**이다. 이를 계기로 dbt 사용자들에게 어필할 수 있도록 기능을 제공할 것으로 기대된다.
- 개인적으로 프로젝트를 살펴볼 예정이다. 단, 현재 상황에서 대체할 수 있는 도구로는 보이지 않는다.
	- 🌈 다만 현재 dev 데이터로는 모델을 확인할 수 없어 prd 데이터를 덤프하여 dev 데이터셋에 밀어넣고 있긴 하다. sqlmesh 로 prd 데이터셋을 이용해 테스트해볼 수 있는 환경을 마련할 수 있을 것으로 예상한다.
아래 레딧 링크에서 재밌는 의견들이 많다. 참고 바랍니다.

### referecens
- [sqlmesh overview](https://sqlmesh.readthedocs.io/en/latest/#why-sqlmesh)
- [https://www.reddit.com/r/dataengineering/comments/124tspm/sqlmesh_the_future_of_dataops/](https://www.reddit.com/r/dataengineering/comments/124tspm/sqlmesh_the_future_of_dataops/)
	- reddit 에서 소개하는 글
	- 코멘트를 살펴보면 커뮤니티 의견을 파악할 수 있다.
