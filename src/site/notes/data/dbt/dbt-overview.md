---
{"author":"jx2lee","alias":"dbt","created":"2024-06-30T00:39:32.000+09:00","last-updated":"2024-09-14 00:59","tags":["dbt","overview"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":"ture","dg-show-tags":false,"dg-pass-frontmatter":false,"priority":2,"permalink":"/data/dbt/dbt-overview/","dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":"ture","dgPassFrontmatter":true,"noteIcon":""}
---



> [!tldr;]
> **ELT 의 Transform 을 유연(sql)하게 관리해주는 워크플로우 도구


### basic
- [[data/dbt/__/__DEPRECATED__dbt-metric\|DEPRECATED: metric in dbt]]
- [[data/dbt/__/run-results-json-means\|run_results.json in dbt]]
- [[data/dbt/__/dbt-thread\|thread in dbt]]


### job knowledge
- [[data/dbt/__/dbt-cicd-pipeline\|dbt-cicd-pipeline]]
- [[data/dbt/__/dbt-custom-schemas\|dbt_projects.yml 에 설정한 schema 이름 변경하기]]
- [[data/dbt/__/dbt-snapshot-query\|dbt snapshot 모델 과정 뜯어보기]]
- [[data/dbt/__/declare_dbt_max_partition marco\|dbt 사용량 줄여보기 - override declare_dbt_max_partition 매크로]]
- [[data/dbt/__/dbt-incremental-full-scan\|dbt 증분모델에서 merge 시 풀스캔 현상(with datetime partition column)]]
- [[data/dbt/__/dbt-braze-cdi-integration\|dbt 와 cdi custom attribute 통합하기]]


### project
- [[data/dbt/__/dbt-metricstore\|SAD - Metric Store]]

### troubleshooting
- [[data/dbt/__/dbt-tmp-table-of-incremental-merge\|merge 전략 incremental 모델의 임시 테이블 관리]]


### archive
- [[data/dbt/__/dbt-cicd-overview\|Atlassian bamboo & Airflow 를 이용한 dbt CI/CD]]


### reference
- [dbt로 ELT 파이프라인 효율적으로 관리하기](https://www.humphreyahn.dev/blog/efficient-elt-pipelines-with-dbt)
- slim ci
    - [데이터에 신뢰성과 재사용성까지, Analytics Engineering with dbt](https://tech.socarcorp.kr/data/2022/07/25/analytics-engineering-with-dbt.html)
    - [Setup a Slim CI for dbt with BigQuery and Docker](https://medium.com/teads-engineering/setup-a-slim-ci-for-dbt-with-bigquery-and-docker-ce8e0a1a38f)
    - [How to use Slim CI with dbt Core](https://www.vantage-ai.com/blog/how-to-use-slim-ci-with-dbt-core)
- [Developer Blog](https://docs.getdbt.com/blog)
    - 흥미롭게 읽은 글감
        - [Building a Kimball dimensional model with dbt](https://docs.getdbt.com/blog/kimball-dimensional-model)
        - [Surrogate keys in dbt: Integers or hashes?](https://docs.getdbt.com/blog/managing-surrogate-keys)
        - [Strategies for change data capture in dbt](https://docs.getdbt.com/blog/change-data-capture)
- dbt cloud 계정
    - https://cloud.getdbt.com
    - ID: dev.jaejun.lee.1991@gmail.com (github 연동 완료)