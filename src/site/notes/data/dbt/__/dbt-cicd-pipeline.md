---
{"dg-publish":true,"permalink":"/data/dbt/__/dbt-cicd-pipeline/","tags":["dbt","cicd"],"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":"","created":"2024-06-30T00:39:32.597+09:00"}
---



> [!tldr]
> dbt CI/CD 개선 프로젝트


### 배경


- github enterprise(ghe) 를 사용하기 시작했다.
- Atlassian bamboo & Airflow 를 조합한 ci/cd 파이프라인에 불필요한 단계들이 많다. 이를 개선하고 Github Actions 로 마이그레이션이 필요하다.


### 목적


- dbt cicd 파이프라인을 개선한다.


### 개선 지표


- (전환 이전) 성공 빌드 / 전체 빌드
- (전환 이후) 성공 빌드 / 전체 빌드
	- 단, 전환 전/후 빌드의 사용자 오류는 제외한다.


### 목표


- 기존 dbt CICD 파이프라인(bamboo)을 Github Actions 로 이관한다.
    - 이관한 파이프라인은 정상동작 해야한다.
        - dbt slim ci/full build
    	- lightdash deploy
- Github Actions 을 활용한 다양한 Workflow 를 추가한다.
    - pr-checker (생성한 PR 을 체크하는 workflow)
    - pr-labeler (PR 레이블을 자동으로 태깅하는 worflow)
    - sqlfluff (PR 생성 시 SQL linter)


### 더보기


- [[data/dbt/__/github-actions-in-data\|Improvement dbt CI/CD with Github Actions]]
