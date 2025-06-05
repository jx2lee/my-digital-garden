---
{"author":"jx2lee","aliases":["dbt_projects.yml 에 설정한 schema 이름 변경하기"],"created":"2024-06-30T00:39:32.000+09:00","last-updated":"2023-09-12 14:12","tags":["dbt"],"dg-publish":true,"permalink":"/data/dbt/__/dbt-custom-schemas/","dgPassFrontmatter":true,"noteIcon":""}
---

### background
- [dbt](https://www.getdbt.com/) 프로젝트를 새로 생성했다.
- [dbt_projects.yml](https://docs.getdbt.com/reference/dbt_project.yml) 을 설정하던 도중
    - {project_name}.{folder} 로 models 프로퍼티를 설정하고 dbt run 을 실행하면 `project_name` 을 prefix 로 데이터셋이 생성되었다.
    - 예를 들어, project 이름이 jaejun 이고 models 하위 test 폴더(`a.sql 이 있다고 가정하자`) 가 있다고 하자. dbt run 을 실행하면 **jaejun_test** 데이터셋이 생성된다.
- 목표는 기존에 생성되어 있는 다른 데이터셋 내 테이블로 생성하고 싶었다.
- 이를 해결하는 방법을 간단히 살펴본다.

goal! **models.<resource_path> 로 생성된 데이터셋의 prefix 를 지우고 싶다.**

### why
- 공식 document 에서 [custom schema configuration](https://docs.getdbt.com/docs/build/custom-schemas#advanced-custom-schema-configuration) 방법을 소개하고 있다.
    - you are using dbt's default implementation of the macro, as defined [here](https://github.com/dbt-labs/dbt-core/blob/main/core/dbt/include/global_project/macros/get_custom_name/get_custom_schema.sql#L47C1-L60)
    - default 로 설정되는 매크로가 있다.
- 더 자세히는 [default macro](https://github.com/dbt-labs/dbt-core/blob/8aaed0e29f9560bc53d9d3e88325a9597318e375/core/dbt/include/global_project/macros/get_custom_name/get_custom_schema.sql#L21) 의 if/else 를 살펴보면
    - custom_schema_name 이 없다면 default_schema 를 (profiles - schema 설정값)
    - custom\_schema\_name 이 지정되었다면 `{{ default_schema }}_{{ custom_schema_name | trim }}` prefix 를 붙이도록 구현되었다.
 
```template
{% macro generate_schema_name(custom_schema_name, node) -%}

    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}

        {{ default_schema }}

    {%- else -%}

        {{ default_schema }}_{{ custom_schema_name | trim }}

    {%- endif -%}

{%- endmacro %}
```

### how to solve
- macro 폴더안에 deault macro 를 override 하는 매크로 파일을 생성한다.
- else 구분에 설정한 schema 패턴을 설정한다.
    - ex) `{{ custom_schema_name | trim }}`
- **prefix 로 default_schema 가 설정되는 것을 삭제할 수 있다.**
- 문제를 파악하는데 시간이 다소 소요되었지, 해결과정은 매우 간단하다.
    - dbt 도 결국 사용자가 지정한 매크로의 우선순위가 높다. (마치 스프링에서 사용하고자 빈을 설정하듯)

### references
- https://github.com/dbt-labs/dbt-core/discussions/6341
- https://docs.getdbt.com/docs/build/custom-schemas#understanding-custom-schemas