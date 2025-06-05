---
{"author":"jx2lee","aliases":null,"created":"2023-12-20T00:33:04.000+09:00","last-updated":"2023-11-16 19:11","tags":null,"dg-publish":true,"permalink":"/notes/__/meetup/20231116-dbt-meetup/","dgPassFrontmatter":true,"noteIcon":""}
---


### Survive to Thrive: 다음 단계의 데이터로 향한 여정

**session summary**
- 프로필
- why dbt
- 쏘카 현황
    - 매크로 주로 활용하는 듯 (700개 넘음..)
    - 1.6.5
    - source 들은 하나의 파일로 관리하고 있음
- hard part
    - Learning Curve
    - Cross-project dependency
    - Testing in 
- Semantic Layer with Metricflow (in socar)
    - fastapi (swaggeer)
    - web
    - 서빙한 지 오래되진 않음
- Future
    - Data Contract

**이후 나눈 얘기들..**
- 험프리님과 metricflow 에 대해 이야기 나눔
    - 개인적인 의견으로는 아직 도입하기는 이른감이 있다고 판단
    - 기다릴 수 있을 때까지 기다리는게 좋을 것 같다.

**궁금한 점**
- source yaml 하나로 관리하는지? 하나로 관리하면 코웍중 문제는 없는지 (conflict)
- 사례로 보여주신 서비스 많이들 사용하는지? → 물어보는 이유가 데이터 문화가 잘 잡혀있는 회사에서 적극적으로 보는지 궁금 (우리는 없어서 슬랙으로 해보려고 함)
- cross project dependency - 모노레포로 관리하시는지, 아니면 나누어 관리하시는지
    - dbt 의존성을 줄이고 싶었다.
    - python 모델로 작성 → response 로 컴파일된 SQL 을 뱉어낼 수 있도록 하는 준비를 했지만 결국 실패 (이 부분은 정확히 이해를 못했지만, 여튼 python 으로 관리할 수 있도록 준비했음)
    - 모노레포로 구성해놓고,
        - models 아래 프로젝트별로 나눔
        - 단, 이 방법도 그렇게 좋지 않은 방법
        - 디펜던시를 어떻게 줄일 수 있는지에 대해서는 아직까지 고민중


### How dbt saved my data engineering startup team

**session summary**
- 프로덕트가 되지 않으면 오너쉽이 생기지 않는다.
- 데이터 엔지니어가 모든 데이터의 주인이 되는 문제가 발생한다.
- ownership → data mesh
- snowflake 사용중
- dbt use cases
    - Lineage, Schema, downstream, upstream, test all in one
- conclusion
    - product 관점으로 바라보자
    - 이 얘기가 많네..


### QnA


- 사용되지 않는 테이블 관리 방안은?
    - (험) 그냥 지운다. 