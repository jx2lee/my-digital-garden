---
{"author":"jx2lee","aliases":["오픈소스 기여"],"created":"2024-11-18T23:27:22.000+09:00","last-updated":"2024-06-23 22:17","tags":["opensource"],"comments":true,"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/opensource-contributions/","dgEnableSearch":true,"dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---


> [!info] issues (idle, assignee)[ ](https://github.com/issues?q=is%3Aopen+is%3Aissue+assignee%3Ajx2lee+archived%3Afalse+-org%3Ajx2lee+)

[⌛️In Progress](https://github.com/pulls?q=is%3Aopen+is%3Apr+author%3Ajx2lee+archived%3Afalse+-org%3Ajx2lee+)
[✅Done](https://github.com/pulls?q=is%3Apr+author%3Ajx2lee+archived%3Afalse+-org%3Ajx2lee+is%3Amerged)


내가 오픈소스 기여하는 이유는 다음과 같아요.
**1. 내가 사용한 오픈소스에 기능을 제공하지 않거나 정상적으로 동작하지 않을 때**
- example
    - [Support n:1 to topic2TableMap (topic:table)](https://github.com/confluentinc/kafka-connect-bigquery/pull/361)
    - [Support to transtorm record without schema (schemaless json format)](https://github.com/an0r0c/kafka-connect-transform-tojsonstring/pull/18)
    - [Add postgres client package](https://github.com/mage-ai/mage-ai/pull/5486)
    - [feat: use python-oracledb instead of cx_Oracle](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1515)
- 내가 속한 조직에 필요한 기능이 부족하거나 수정되어야할 경우가 많았습니다. 회사 VCS 에 올려 따로 관리해도 좋지만 불필요한 과정(예를 들어, 변경된 기능을 포함해 cicd 파이프라인을 새로 생성)을 없애고 싶었어요.

**2. 회사에서 많은 코드베이스를 갖는 프로젝트를 접할 기회가 적음**
- Airflow 경우만 보더라도 코드 라인수가 굉장히 많습니다. 이들이 어떻게 동작하는지 코드베이스로 확인할 수 있는 회사 코드는 드물어요. (~~물론 내가 거쳐온 조직이 관리한 코드가 작을 수 있죠~~)
- 자연스럽게 많은 양의 코드를 살펴보면 넓은 시야를 가질 수 있다고 믿는다. 물론 이로인해 불필요한 설계를 할 수 있겠지만 나에게 있어선 많은 코드를 보고 이해하는 것이 내 성장에 굉장히 큰 도움을 준다 생각해요.

**3. 도움 받은 만큼 나도 도움을 주고 싶은 마음**
- 가장 큰 이유. 근데 왜 마지막에 언급했을까요? ~~가식적으로 보일 것 같았다.~~
- 데이터 엔지니어라면 누구나 쓰는 Airflow, 이를 만든 사람들이 있었기에 내 일을 하는데 정말 많은 도움을 주었다. 도움만 받지 말고 나도 누군가에게 도움을 줄 수 있지 않을까? 라는 마음에서 시작했어요.
- 나도 누군가에게 도움을 줄 수 있다고 생각했어요. 프로젝트를 이끄는 컨트리뷰터/커미터/PMC 에게 "편히 사용하게 만들어줘서 고마워요" 라는 말대신 코드로 보답하고 싶었다.


> [!summary]- Outputs
> - airflow
>     - [feat (airflowctl): add dag operations to conform all API endpoints](https://github.com/apache/airflow/pull/50424)
>     - [Fix image url in IDE onboarding doc (about task sdk)](https://github.com/apache/airflow/pull/48549)
>     - [Add a note `airflow users` command is available when FAB auth-manager is enabled](https://github.com/apache/airflow/pull/46862)
>     - [Enable to add inline ssh key in GitHook](https://github.com/apache/airflow/pull/46181)
>     - [Forbid extra fields on execution api](https://github.com/apache/airflow/pull/44986)
>     - [Fix GitDagBundle to support https (include 46073/46179)](https://github.com/apache/airflow/pull/46226)
>     - [Set container name to envSourceContainerName in KEDA ScaledObject](https://github.com/apache/airflow/pull/44963)
>     - [Support connection extra parameters in MsSqlHook](https://github.com/apache/airflow/pull/44310)
>     - [Support multiple executors in chart](https://github.com/apache/airflow/pull/43606)
>     - [Bump to mypy-boto3-appflow and pass without # type: ignore[arg-type]](https://github.com/apache/airflow/pull/44115)
>     - [Add parent_model param in UploadModelOperator](https://github.com/apache/airflow/pull/42091)
>     - [Add CloudRunServiceHook and operator](https://github.com/apache/airflow/pull/40008)
>     - [enable AIRFLOW\__CELERY__BROKER_URL_CMD when passwordSecretName is true](https://github.com/apache/airflow/pull/40270)
> - professional-services-data-validator:
>     - [fix: Remove implicit "connections" subdirectory from GCS connections path (PSO_DV_CONN_HOME)](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1598)
>     - [feat: Enable --grouped-columns for validate custom-query column](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1587)
>     - [feat: add support for Oracle BOOLEAN](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1558)
>     - [feat: use python-oracledb instead of cx_Oracle](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1515)
>     - [fix: deduplicate spanner client in SpannerBackend](https://github.com/GoogleCloudPlatform/professional-services-data-validator/pull/1554)
> - dbt
>     - [Fix configuration of turning test warnings into failures](https://github.com/dbt-labs/dbt-core/pull/9347)
>     - [Fix package-lock file's bad indentation](https://github.com/dbt-labs/dbt-core/pull/9341)
>     - [dbt-bigquery: add test case when raise ServiceUnavailable in is_retryable](https://github.com/dbt-labs/dbt-bigquery/pull/1224)
> - kafka-connect-bigquery: [Support n:1 to topic2TableMap (topic:table)](https://github.com/confluentinc/kafka-connect-bigquery/pull/361)
> - kafka-connect-transform-tojsonstring: [Support to transtorm record without schema (schemaless json format)](https://github.com/an0r0c/kafka-connect-transform-tojsonstring/pull/18)
> - meltano
>     - [docs: Fix YAML indent on getting started page and fix link to page source in GitHub](https://github.com/meltano/meltano/pull/7187)
>     - [fix(cli): working GCS state backend properly when any files were located in root bucket](https://github.com/meltano/meltano/pull/8648)
> - datahub
>     - [docs(dbt): fix indentation in dbt meta mapping docs](https://github.com/datahub-project/datahub/pull/7045)
>     - [docs: fix image in development](https://github.com/datahub-project/datahub/pull/7637)
>     - [docs(google-analytics): Correct grammatical error in README.md](https://github.com/datahub-project/datahub/pull/6870)
>     - [docs(architecture): edit documents in architecture section](https://github.com/datahub-project/datahub/pull/6798)
>     - [docs(aws): edit markdown link](https://github.com/datahub-project/datahub/pull/6706)
>     - [docs(quickstart): enable slack community link](https://github.com/datahub-project/datahub/pull/6209)
> - great-expectations: [[DOCS] edit term(data_conext, checkpoints)-link in with airflow](https://github.com/great-expectations/great_expectations/pull/6646)
> - mageai
>     - [Add postgres client package](https://github.com/mage-ai/mage-ai/pull/5486)
>     - [Why bq client create dataset in data loader?](https://github.com/mage-ai/mage-ai/issues/5197) support
> - PyAirbyte: [Chore: Remove import only used in TYPE_CHECKING](https://github.com/airbytehq/PyAirbyte/pull/421)
> - astronomer-cosmos: [Improve MWAA getting-started docs by removing unused imports](https://github.com/astronomer/astronomer-cosmos/pull/1562)
