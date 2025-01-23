---
{"dg-publish":true,"permalink":"/data/dbt/__/metricflow/","noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


dbt & metricflow 를 이용한 메트릭 지정은 다음 순서로 진행해야한다.
1. semantic_models 정의
2. metrics 정의
3. mf 커맨드로 metric 조회
    - ex. `mf query —metric {metric_name}`
    - …

metricflow.cli.cli_context.CLIContext 를 이용하면 dbt 관련 메타 정보들을 확인할 수 있다.
- 확인 가능한 정보
    - dbt_artifacts: profile, adapter, project, semantic_manifest
        - `각 필요한 정보만 가져올 수 있는 메서드도 존재하는가?`
        - profile, project, semantic_manifest 정도만 가져와도 좋을 듯 하다.
    - dbt_project_metadata
        - dbt_paths, profile, project, project_path

`dbt compile` 명령어도 target warehouse 에 쿼리를 실행한다.
- dbt-metricflow 패키지를 설치한 경우 metric_time_spine 테이블을 생성한다.

CLIContext 혹은 MetricFlowClient 를 이용할 수 있다.
- CLIContext 는 스크립트 위치가 dbt 프로젝트의 루트 디렉토리여야 한다.
- MetricFlowClient 를 이용하고 싶은데 SemanticManifest 설정부분이 조금 까다롭다.
    - 이 부분은 더 파봐야 알 것 같음