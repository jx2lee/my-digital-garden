---
{"dg-publish":true,"alias":"merge 전략 incremental 모델의 임시 테이블 관리","dg-pass-frontmatter":false,"dg-show-backlinks":false,"tags":["dbt","troubleshooting"],"permalink":"/data/dbt/__/dbt-tmp-table-of-incremental-merge/","dgPassFrontmatter":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


# background
- dbt model 중 `incremental_strategy=merge` 인 경우 `__dbt_tmp` 임시 테이블을 생성한다.
	- 만료시간이 12시간으로 설정한다.
- bigquery 혹은 카탈로그 서비스를 이용하는 사용자들의 위 테이블 문의가 증가하고 있다.
- 따라서 incremental 모델 중 merge 전략은 취한 경우 temp 테이블 관리가 필요해졌다.

# incremental strategy

> 일부 adapter 에 incremental_strategy 옵션은 dbt가 증분 모델(incremental model)을 빌드하는 데 코드를 제어한다. 데이터 양, unique_key의 신뢰성 또는 가용성에 따라 접근 방식이 다를 수 있다. 본문에서는 target DW 를 bigquery 인 케이스를 다룬다.

dbt 에서 제공하는 incremental 모델을 이용할 때 사용하는 옵션이다. dbt 는 기본적으로 full-refresh 전략을 취한다. 이는 모델 빌드 시 destination 테이블을 삭제하고 새롭게 생성하는데, 자세한 내용과 advantage & disadvantage 는 아래 그림의 링크를 참고한다. 아래는 full refresh 전략으로 생성한 모델의 흐름을 나타낸다.

![](https://i.imgur.com/N8N5mUZ.png)

하지만 위 full refresh 전략은 비효율적이다. 매번 destination 테이블을 삭제하고 재생성하기 때문이다. 비용을 절감하거나, 속도를 높여야 하거나, 테이블이 새 데이터를 자주 확인(e.g *일마다 새로운 데이터가 적재된 경우*) 해야하는 경우 incremental 모델을 고려해야한다. 기본적으로 제공하는 incremental 전략들이 존재하고 adpater 마다 약간의 동작이 다를 수 있으니 adapter 상세 config 를 살펴보는 것이 좋다.

제공하는 strategy 에 대해 설명이 잘되어 있는 링크를 아래 남겨두니 자세한 내용은 [링크](https://medium.com/indiciumtech/understanding-dbt-incremental-strategies-part-1-2-22bd97c7eeb5)를 확인해보자. merge 전략으로 incremental 모델 생성 흐름은 아래와 같다.

![](https://i.imgur.com/YEWL6ET.png)


# what's wrong

merge 옵션으로 incremental 모델을 생성할 때 기존 destination 모델을 이용해 temp 테이블을 생성한다 (with `__dbt_tmp` suffix). strategy 가 insert_overwrite 인 경우에는 생성한 temp 테이블을 삭제하는 쿼리가 컴파일로 확인가능하지만, merge 인 경우 생성한 temp 테이블을 Drop 하는 쿼리를 확인할 수 없었다.

혹시나 merge strategy 시 temp table 을 삭제하지 않는 원인이 궁금하여 커뮤니티에 [질문](https://getdbt.slack.com/archives/CBSQTAPLG/p1674915934074389)해보았다. 하루가 걸려 두 개 답변을 확인할 수 있었다. 첫번째는 specific 한 부분으로 해당 git repo 에 issue 를 생성해보는 것이 좋겠다고 하였다. 두번째, adapter 코드를 직접 확인하면 로직 파악이 쉬울것이며 현재 진행중인 이슈번호([#184](https://github.com/dbt-labs/dbt-bigquery/issues/184))를 남겨주었다. 해당 이슈를 간단히 정리하면 다음과 같다.

## #184
![](https://i.imgur.com/nENgYHM.png)

글보다 캡쳐로 설명하는게 좋을 것 같다. 위 오프너와 동일하게 on_schema_change 를 fail 로 변경해도 동일하게 temp 테이블이 삭제되지 않았다. 이에 답변으로 [#154](https://github.com/dbt-labs/dbt-bigquery/issues/154) 이슈와 비슷한 사례로 보이며 on_schema_change 측면에서는 [#163](https://github.com/dbt-labs/dbt-bigquery/issues/163) 이슈와 동일할 것 같으니 확인해보라고 했다. 하지만, 이슈 opener 는 공유해준 이슈와는 다른 케이스라며 **삭제되지 않은 temp table 로 인해 자신의 target database 가 오염**되고 있다고 주장했다.

이후 동일한 현상이 발생한 유저와 airbyte 이슈에 멘션되면서 추가 답변은 달리지 않았다. 결국 소스코드 수정을 하고 회사에서만 사용가능한 dbt 를 빌드하고 유지보수해야한다. 이는 불필요한 리소스가 많이 소모될 것 같아 우회할 수 있는 방안을 찾기 위해 incremental 모델의 merge 전략이 어떠한 방식으로 구현됐는지 트래킹 해보려 한다.

# materialized **incremental** & **merge** strategy & **append_new_columns**

prep_app_log 모델 생성 시 일어나는 과정을 디버깅했다. dbt 커맨드의 `--debug` 플래그를 추가하면 실행한 SQL 문을 상세히 확인할 수 있다.
- 파티션 키로 설정한 event_created_at 의 최댓값을 변수로 선언한다.
```sql
declare _dbt_max_partition timestamp default (
	select max(event_created_at) from `******`.`******`.`prep_app_log`
	where event_created_at is not null
	);
```
- prep_app_log__dbt_tmp 테이블을 생성한다. (`crete or replace`)
	- incremental 모델 생성을 위한 SQL 기반(*예제에서는 prep_app_log.sql*)으로 생성한다.
	- expired timestamp: 12 hour
```sql
	create or replace table `******`.`******`.`prep_app_log__dbt_tmp`
	partition by timestamp_trunc(event_created_at, day)
	
	OPTIONS(
	description="""******""",
	
	expiration_timestamp=TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 12 hour)
	)
	as (
-- 이하 prep_app_log.sql 에 작성한 SQL
with stg_analytics as (
	...
	...
),
prep_app_log as (
	select * from renamed 
),
final as (
	select * from prep_app_log
)
select * from final
	);
```
- tmp 테이블과 prep_app_log 테이블을 merge 한다.
```sql
merge into `********`.`********`.`prep_app_log` as DBT_INTERNAL_DEST
	using (
	select * from `********`.`********`.`prep_app_log__dbt_tmp`
	) as DBT_INTERNAL_SOURCE
	on 
			DBT_INTERNAL_SOURCE.log_id = DBT_INTERNAL_DEST.log_id
		


when matched then update set
	`log_id` = DBT_INTERNAL_SOURCE.`log_id`,..., `********` = DBT_INTERNAL_SOURCE.`********`


when not matched then insert
	(`log_id`, ..., `********`)
values
	(`log_id`, ..., `********`)
```

위 내용을 토대로 이해한 incremental model & merge 전략의 실행과정은 다음과 같다.
- partition key 의 max 값을 구하고 이를 기준으로 dbt__tmp 임시 테이블을 생성한다.
- 생성한 임시테이블을 이용해 기존 source 테이블과 merge 한다.
- 단, merge 전략에서는 사용한 임시 테이블을 삭제하지 않았다.

# solution
구글링 & chatgpt & 커뮤니티(*비슷한 질문이 [slack 채널](https://getdbt.slack.com/archives/CBSQTAPLG/p1671160243056179)에 있어 공유한다*)를 검색하고 고민한 결과 **테이블 정리 작업을 위해 post-hook 을 사용**하기로 결정했다. 모든 merge 모델들을 검색하고 config 에 post-hook 하는 방법 대신, root 폴더의 dbt_project.yml 에 post-hook 을 추가하여 모델 생성 이후 `DROP {type} IF EXISTS` 쿼리가 실행될 수 있도록 macro 를 만들었다. 설정 과정은 다음과 같다.
- macro: `delete_tmp_table.sql`
	- dbt-utils: get_relations_by_pattern-source 를 참고하여 만들었다.
```sql
{% macro delete_tmp_table() %}
{% for relation in relations_to_drop %}
	{% set drop_command -%}
		DROP {{ relation.type }} IF EXISTS {{ relation }};
	{%- endset %}
	{% do log(drop_command, info=True) %}
	{% do run_query(drop_command) %}
{% endfor %}
{% endmacro %}
```

- 작성한 매크로를 프로젝트 root 폴더 dbt_project.yml 내 post-hook 으로 등록한다.
```
models:
...
	...
+post-hook: `"{{ delete_tmp_table() }}"`
```

- dbt run 커맨드를 실행하며 tmp 테이블들을 삭제하는 쿼리를 확인하고 DW 에 오염된 테이블이 있는지 확인한다.

# referenece
- [Incremental models](https://docs.getdbt.com/docs/build/incremental-models)
- [Understanding dbt Incremental Strategies part 1/2](https://medium.com/indiciumtech/understanding-dbt-incremental-strategies-part-1-2-22bd97c7eeb5)
- [Understanding dbt Incremental Strategies part 2/2](https://medium.com/indiciumtech/understanding-dbt-incremental-strategies-part-2-2-add59889ea17)
- [dbt-utils: get_relations_by_pattern-source](https://github.com/dbt-labs/dbt-utils#get_relations_by_pattern-source)