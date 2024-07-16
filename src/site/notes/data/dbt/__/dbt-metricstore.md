---
{"author":"jx2lee","aliases":null,"created":"2024-06-30T00:39:32.598+09:00","last-updated":"2024-02-24 22:39","tags":["dbt","metricstore","sad"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":true,"dg-pass-frontmatter":true,"permalink":"/data/dbt/__/dbt-metricstore/","dgPassFrontmatter":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":""}
---



> [!tldr] 진입 장벽 낮은 슬랙을 이용해 데이터 프로덕트를 개발한다.


### 배경
- [metric 패키지](https://docs.getdbt.com/docs/build/metrics-overview) 를 통해 코인원 지표 저장소를 오랜 시간 운영했다. dbt v1.6 이후에는 지표 저장소에 사용한 dbt_metrics 패키지 지원을 중단한다고 한다.
    - [https://docs.getdbt.com/blog/deprecating-dbt-metrics](https://docs.getdbt.com/blog/deprecating-dbt-metrics)
- 지표저장소의 현상황은 다음과 같다.
    - Analytics Engineer 를 제외하고 지표들을 생성하고 관리하는 자유도가 낮다. 모든 지표 생성요청은 Analytic Engineer 로 향하고 있다. 심지어 Git 작업은 Analytics Engineer 에게도 부담이다. (꽤나 많은 conflict..)
    - 이미 만들어진 지표는 잘 관리되고 있지 않다. 누가 Owner 이며, 퇴사자를 대체할 다른 Owner 가 누군지 알 방법이 없다.
- `지표 저장소를 활용할 수 있는 프로덕트를 만들어볼까?`


### 목적
지표 저장소를 개선한다


### 개선 지표
TBD


### 목표
- 개선 이전 지표 저장소에서 제공한 기능을 동일하게 제공한다. 지표를 목록을 조회하고, 지표 목록 중 하나를 선택해 직접 결과를 확인할 수 있어야 한다.
- 현 지표저장소의 문제를 파악하고 개선할 수 있도록 한다.
- 남들에게 자랑할 수 있는 프로덕트를 만든다.


### 더보기

- Software Architecture Documentation
    - [[data/dbt/__/dbt-metricstore-sad-current-status\|SAD - Current status]]
    - [[data/dbt/__/dbt-metricstore-sad-toplevel-module-uses-view\|SAD - Top Level Module Veiw]]
    - [[data/dbt/__/dbt-metricstore-sad-cnc-view\|SAD - C&C View]]
    - [[data/dbt/__/dbt-metricstore-deployment-view\|SAD - Deployment View]]