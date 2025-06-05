---
{"author":"jx2lee","aliases":"dbt 사용량 줄여보기 - override declare_dbt_max_partition 매크로","created":"2024-06-30T00:39:32.000+09:00","last-updated":"2023-08-08 21:01","tags":["dbt","macro"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":true,"dg-pass-frontmatter":"ture","permalink":"/data/dbt/__/declare_dbt_max_partition marco/","dgPassFrontmatter":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":""}
---



### purpose


`dataset.INFORMATION_SCHEMA` 로 max partition 값을 구하는 쿼리를 변경한다.

- dbt incremental 모델 실행 시 max 파티션 값을 계산한다.
- 이때, 실행 전 모델 테이블에서 select max(partition_id) 쿼리를 수행한다.
- 테이블 로우가 많아질수록 위 쿼리의 Billed Bytes 값이 증가할 수 있다.
- 조금이라도 용량을 아껴볼겸 단순 select max 쿼리를 실행하지 않고 information_schema 를 읽어 파티션 최댓값을 가져와보자.


### deliverable


- incremental 모델 생성에 사용하는 dbt_max_partition 을 계산하는 매크로를 확인한다.
	- `declare_dbt_max_parittion`
- {project_root}/macro 폴더 안에 동일한 이름의 macro 파일을 생성하면 override 한다.
- 쿼리를 수정해보자.

	```jinja2
	{% macro declare_dbt_max_partition(relation, partition_by, compiled_code, language=‘sql’) %}
	
	  {#-- TODO: revisit partitioning with python models --#}
	  {%- if ‘_dbt_max_partition’ in compiled_code and language == ‘sql’ -%}
	
	    declare _dbt_max_partition {{ partition_by.data_type_for_partition() }} default (
	      select parse_{{ partition_by.data_type_for_partition() }}(‘%Y%m%d’, max(partition_id)) from {{ relation.dataset }}.INFORMATION_SCHEMA.PARTITIONS
	      where table_name = ‘{{ relation.table }}’ and partition_id != ‘__NULL__’
	    );
	
	  {%- endif -%}
	
	{% endmacro %}
	```
- 

### result

dev 쿼리 사용량 비교
- before: 135.8GiB
- after: 135.5GiB
- **300MB 감소**
main 쿼리 사용량 비교
- before: 
- after: 


### discussion


- https://github.com/dbt-labs/dbt-core/issues/3666#issuecomment-891428493
- https://github.com/dbt-labs/dbt-bigquery/pull/285
  - [issue](https://github.com/dbt-labs/dbt-bigquery/issues/286)
  - 같은 생각을 한 개발자 PR 이 존재한다.
  - get_max_partition jinja 로 INFORMATION_SCHMEA 를 이용할 수 있게 변경한 요청이다.
  - 아직까지 병합되지 않았다.
    - 병합된 후 사용해도 좋을 듯 하다.
  - 매크로 사용이 제한적이다.
      - partition 칼럼이 integer range 인 경우 실행에 실패할 것이다.
      - 범용적으로 사용할 수 있도록 개선이 필요하다.
