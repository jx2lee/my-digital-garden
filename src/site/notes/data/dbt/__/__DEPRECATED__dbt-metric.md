---
{"author":"jx2lee","alias":"DEPRECATED: dbt metric","created":"2024-06-30T00:39:32.000+09:00","last-updated":"2024-03-03 20:38","tags":["dbt","metric","deprecated"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":true,"dg-pass-frontmatter":true,"permalink":"/data/dbt/__/__DEPRECATED__dbt-metric/","dgPassFrontmatter":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":""}
---



### overview


metric은 0개 이상의 차원(dimension)을 지원하는 테이블에 대한 집계다.
- 활성 사용자 수
- 월별 반복 수익(MRR)

`exposures` 과 마찬가지로 metric 은 노드로 표시하며 YAML 파일로 표현할 수 있다. dbt 프로젝트에서 메트릭을 정의하면 중요한 비즈니스 로직을 테스트하거나, version-controlled 코드로 인식할 수 있다. 또한 이러한 메트릭 정의를 downstream tooling 에 추가하여 metric report 의 일관성 및 정확성을 높일 수 있다.

![](https://i.imgur.com/kykrg65.png)


### metric profit


**다운스트림 도구(downstream tool)에서 메트릭 사양 사용**

dbt 컴파일 컨텍스트는 graph.metrics 변수를 통해 메트릭에 접근할 수 있다. manifest 에는 다운스트림 메타데이터 사용에 대한 메트릭을 포함한다.

**종속성(dependency) 확인 및 선택**

`exposure` 와 마찬가지로 메트릭으로 롤업되는 모든 것을 확인하고(`dbt ls -s +metric:*`) dbt docs 에서 시각화할 수 있다.
- [자세한 내용](https://docs.getdbt.com/reference/node-selection/methods#the-metric-method)


### metric 이용


#### `metrics:` in `.yml`


metrics: 아래로 파일에 정의할 수 있다. metric 네이밍 제약사항은
- 문자, 숫자 및 밑줄만 포함해야 한다(*공백이나 특수 문자 제외*).
- 문자로 시작해야 한다.
- under 250 자
- 제목 대/소문자, 공백 및 특수 문자를 사용하여 사람이 읽기 쉽게 생성한다. 짧은 이름을 만들려면 레이블 속성을 사용한다.
	- 메트릭을 정의하고 구조화하는 방법에 대한 더 많은 예와 팁은 [링크](https://docs.getdbt.com/blog/how-to-design-and-structure-metrics)에서 확인하자

**example**
```yaml
# models/marts/product/schema.yml

version: 2

models:
 - name: dim_customers
   ...

metrics:
  - name: rolling_new_customers
    label: New Customers
    model: ref('dim_customers')
    [description](description): "The 14 day rolling count of paying customers using the product"

    calculation_method: count_distinct
    expression: user_id 

    timestamp: signup_date
    time_grains: [day, week, month, quarter, year]

    dimensions:
      - plan
      - country
    
    window:
      count: 14
      period: day

    filters:
      - field: is_paying
        operator: 'is'
        value: 'true'
      - field: lifetime_value
        operator: '>='
        value: '100'
      - field: company_name
        operator: '!='
        value: "'Acme, Inc'"
      - field: signup_date
        operator: '>='
        value: "'2020-01-01'"
        
    # general properties
    config:
      enabled: true | false
      treat_null_values_as_zero: true | false

    meta: {team: Finance}
```


### dynamic query


### dbt_metrics package


- dbt 패키지 중 하나
- dbt 모델의 실행 결과에 대한 메트릭을 생성하고 모니터링할 수 있도록 도와주는 패키지
- dbt 실행 결과에서 중요한 측정 지표를 추출하고, 해당 측정 지표를 이용하여 알림을 설정하거나 대시보드를 구성할 수 있음
- `metric.yml` 파일을 사용하여 모델 실행 결과에서 측정할 지표를 정의할 수 있음
- 지표추출 방법을 지정
	- 예를 들어 특정 열의 합계, 평균 또는 행 수
- `dbt run` 명령을 실행할 때마다 계산
- `dbt docs generate` 명령을 실행하여 생성된 메트릭에 대한 문서를 자동으로 생성


### reference


[Document: Metrics](https://docs.getdbt.com/docs/build/metrics#defining-a-metric)
[package: dbt_metrics](https://github.com/dbt-labs/dbt_metrics)