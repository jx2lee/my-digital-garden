---
{"author":"jx2lee","aliases":"Self Serve Data","created":"2024-10-02T18:51:46.000+09:00","last-updated":"2024-07-28 02:04","tags":["airbyte","datahub","opensource"],"project":{"include":true,"status":"done","root":true,"company":"coinone","duration":"2022.09 - 2023.03"},"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/data/etc/self-serve-data/","dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---


### background
- 코인원 크루를 대상으로 데이터 조직 도움 없이 데이터를 활용할 수 있도록 하는 self data serve 환경이 필요했습니다. 기능조직인 데이터셀의 업무 부하를 줄이고 사용자들이 쉽게 데이터를 적재하고, 살펴보고, 혼자서도 뜯어볼 수 있는 환경을 만들고 싶었어요.
- 그래서 디스커버리 플랫폼인 Datahub, 외부 데이터 수집을 위한 Airbyte, 사용자 친화적인 고수준 환경을 제공하는 Mage.ai 를 사내 임직원들에게 제공하고자 합니다.


### objective
Self-data-serve 환경을 위한 데이터 도구들을 제공합니다.


### howto
```mermaid
flowchart TB
    subgraph data["Storage Layer"]
        external_data[(외부데이터)]
        bigquery[(BigQuery)]
        subgraph rds["내부 데이터"]
            instance1[("exchange")]
            instance2[("account")]
            instance3[("wallet")]
        end
    end

    subgraph platform["Self Serve Layer"]
        direction TB
        mageai[MAGE]
        airbyte[Airbyte]
        datahub[Datahub]
    end

    %% single node %%
    user

    %% external_data --> |extract| airbyte %%
    %% rds --> |insgest| datahub %%
    user --> |use| platform
    platform --> |load & transform| data
    %% mageai -.- bigquery %%
```
- Datahub
    - 카탈로그 수집을 위해 Airflow 와 수집 스크립트를 이용합니다. RDB 는 information 스키마를, dbt 혹은 bigquery 는 datahub 에서 제공하는 ingestion CLI 를 활용한 컨테이너 이미지를 활용합니다.
        - RDB 의 information 스키마를 읽고 Datahub 포맷에 맞는 파일을 생성, 이를 주입하는 CLI 실행으로 동작합니다. 링크를 참고하여 [스크립트를](https://tech.socarcorp.kr/data/2022/03/16/metdata-platform-02.html) 작성했어요. (source -> [metadata file](https://datahubproject.io/docs/generated/ingestion/sources/metadata-file))
        - 작성된 스크립트는 Airflow 를 통해 일 배치로 수집합니다.
        - dbt, bigquery 의 메타데이터는 [링크](https://tech.socarcorp.kr/data/2022/03/16/metdata-platform-02.html)와 같은 recipe yaml & CLI 를 활용해 주입합니다. 
    - Helm 차트로 구성, ArgoCD 를 이용해 배포합니다.
- AirByte
    - 내부 데이터 수집은 Airflow Custom Operator 로, 외부 데이터(외부 거래소 데이터)는 Airbyte 로 수집했습니다. 데이터 성격(내부vs외부)에 따라 수집도구를 분류하고 싶었어요.
        - Airbyte 에서는 Python CDK 를 제공합니다. 특히, HTTP-API 기반 커넥터를 개발을 쉽게 할 수 있어요. 노코드 커넥터 빌더를 이용해 쉽게 구현할 수 있지만 비즈니스 로직이 더해진 커넥터가 필요할 때 적극 활용했어요.
        - custom connector 들을 담는 github repository 를 만들고 Action Workflow 를 이용해 컨테이너 이미지를 생성 & ECR 로 푸시하도록 구성했어요. 여러 커스텀 커넥터 이미지를 담고 있기 때문에 `metadata.yml` 의 시멘틱버저닝을 이용해 개발/운영 환경에 배포할 수 있도록 했어요.
    - Helm 차트로 구성, ArgoCD 를 이용해 배포했습니다. 
        - oss 버전에는 OAuth 기능을 [제공하지 않습니다](https://github.com/airbytehq/airbyte/issues/13021).
        - 클라우드에서만 제공하기에 oauth-proxy 를 이용해 google 계정으로 로그인 가능한 환경을 구성했어요. 보다 자세한 내용은 아래 more 에 첨부된 문서를 확인해주세요!
- MageAI
    - 외부 기관 요청대응하는 팀(데이터추출셀)을 위해 MageAI 라는 서비스를 제공하였습니다. 3세대 워크플로우 관리도구라 소개하는 메이지는, 블럭 단위로 태스크를 관리하며 airflow 보다 조금 더 직관적인 사용자 경험을 제공해요. 공식 헬름 차트를 이용해 쿠버네티스 환경에 배포하였고, 노드 공유 스토리지로 구성된 차트를 자체적으로 수정했어요.
    - 또한, 메이지 공식 컨테이너 이미지에 postgres 클라이언트 설치가 되어 있지 않아 관련 [PR](https://github.com/mage-ai/mage-ai/commit/bd98aa61b0537d5322b9e64978c8965bc82c3f3e) 을 제출하고 반영되었습니다. 이 덕에 [spotlight](https://www.linkedin.com/posts/magetech_community-spotlight-jaejun-lee-wed-like-activity-7255611108399501312-FSP3?utm_source=share&utm_medium=member_desktop) 도 받았어요!

> [!faq] Airbyte 도입 배경 (Airfow 와 비슷하지 않아요?)
> - `배경`: Airflow / 자체 구현한 수집 엔진으로 ELT 파이프라인을 구성했습니다. 대상은 거래소 서비스에서 발생한 데이터 였어요. 그러다 전사에서 외부 데이터를 보고 싶은 니즈가 생겼습니다. 예를 들어, 국내 5대 거래소의 거래대금이나 혹은 CMC 로 각 가장자산별 볼륨 등이요.
> - `Airbyte?`: Open-Source Data Integration Platform 이라고 설명합니다. 맞아요. 정작 사용해보면 엄청나게 많은 기능을 제공합니다(Source/Target 지정한 커넥션의 스케줄 지정 -> 실행하면 파이프라인 생성). 그 중에 Python Connector Development Kit, CDK 에 대한 인터페이스가 훌륭했어요. 그 중에서도 [HTTP-API-based Connector](https://docs.airbyte.com/platform/connector-development/cdk-python/http-streams) 가 눈에 띄었고, 확인해보니 잘 설계되었고 금방 외부데이터를 수집할 수 있을거라 기대했어요.
> - `그럼 왜?`: 내부 거래소 데이터가 아닌 외부 데이터라 자체 엔진을 개발하면 **신뢰성 높은 파이프라인을 제공하기 어려웠습니다**. 우리 파이프라인은 문제 없지만 외부 데이터 프로바이더에서 문제라도 생기면? **완벽히 제공하는 건 불가능**하다 생각했어요. 빠른 시일내에 딜리버리해야 했습니다. 기간을 맞추기에는 수집기를 직접 개발한다면 **로직에 허점이 많을거라 판단**했어요. (그만큼 HTTP API based Connector 가 견고하게 디자인되었구요!)
> - `결론`: 주어진 기한내에 견고한 외부 데이터 수집을 위해 HTTP API 기반 커넥터를 쉽게 구성할 수 있는 Airbyte 를 이용했습니다.


### result
- 사내 임직원을 위한 셀프 서브 환경을 제공했습니다.
    - DataHub: 사례로 보안 조직에서 개인정보 현황 파악을 위해 Datahub 를 이용했습니다. Tag 기능을 이용해 보안등급을 테이블 혹은 컬럼 단위로 부여하고, 보고자료(현황파악)에 활용했어요. Datahub 덕분에 DBA 조직에 따로 문의하지 않고 내부 데이터의 개인정보를 분류할 수 있었어요.
    - Airbyte: 사례로 국내 거래소 시간봉 api 를 이용해 보다 자세한 점유율을 사내에 제공했습니다. 물론 돈을 지불하고 쉽게 살펴볼 수 있는 서비스가 많았지만, 일 거래 금액을 보다 정확하게 측정하고 거래소 api 로 캔들 정보를 수집했어요. 이때 커스텀 커넥터를 개발하고 활용한 덕에 사내 크루들에게 신뢰할 수 있는 거래소 데이터를 제공했어요.
    - MageAI: 사례로 외부 기간 보고를 위한 정제 작업을 메이지에서 사용하였고, 일/주간 보고를 위한 보고서를 메이지를 통해 생산했습니다.
- 본인이 원하는 데이터를 수집할 수 있도록 환경을 제공하고(Airbyte), 어떤 데이터가 어디 있는지 데이터 조직에 물어보지 않고 찾을 수 있으며(Datahub), 데이터 조직 도움없이 다양한 데이터를 활용할 수 있는 환경(MageAI)을 만들었어요.


### keytakeaway
- 많은 사용자가 선택한 성숙한 제품부터, 얼리 스테이징 단계에 있는 도구까지 여러 오픈소스를 경험했습니다.
    - Datahub 의 경우 문서를 굉장히 많이 참고했었는데, 처음으로 오픈소스에 기여해봤습니다. 대부분이 Docs 영역이긴 하지만 오픈소스 생태계에서 어떻게 소통하고 기여가 이루어지는지 체험할 수 있었어요.
    - 특히 MageAI 는 아직 사용자가 많지 않은 도구라 손이 많이 갔습니다. 배포 후 사용하는 조직에서도 이용에 많은 어려움이 있었지만 코드레벨로 깊게 파보기도 해봤고, 이슈를 발견하고 [리포팅 한 경험](https://github.com/mage-ai/mage-ai/issues/5197)도 있습니다.
    - 도구들이 어떻게 구성되어있는지(어떤 아키텍처인지), 도구들을 디깅하며 어떤 이유로 만들었는지 등 프로젝트 이전에 보지 못했던 넓은 시야를 갖게 되었습니다.
- 클라우드 환경에 오픈소스를 배포, 우리 환경에 맞게 때로는 변경했습니다.
    - 물론 배포한 도구들은 모두 헬름 차트를 제공했어요. Airbyte 의 경우 OAuth 기능을 클라우드에서만 제공 -> oauth2-proxy 등을 이용해 강력한 보안을 제공했어요.
    - 이러한 경험을 바탕으로 내부 데이터 프로덕트 개발 시 헬름 차트로 생성하여 배포하는 것을 목표로 하고 있어요.


### more
[[data/airbyte/__/airbyte-custom-chart\|Airbyte 차트 구성하기 (oauth2 & 커넥터 관리)]]
