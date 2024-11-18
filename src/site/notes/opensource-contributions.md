---
{"dg-publish":true,"permalink":"/opensource-contributions/","tags":["opensource"],"dgEnableSearch":true,"dgLinkPreview":true,"noteIcon":"","created":"2024-11-18T23:27:22.872+09:00"}
---


⌛️
- [airflow: Support multiple executors in chart](https://github.com/apache/airflow/pull/43606)
- [dbt-common: Add indent option in JsonLogger](https://github.com/dbt-labs/dbt-common/pull/210)
- [dbt: Modified git tag option in `dbt deps`](https://github.com/dbt-labs/dbt-core/pull/10398)
- [kafka-connect-transform-tosjonstring: Enable to pass empty ArrayList in schemaless](https://github.com/an0r0c/kafka-connect-transform-tojsonstring/pull/20)

✅
- airflow
    - [Bump to mypy-boto3-appflow and pass without # type: ignore[arg-type]](https://github.com/apache/airflow/pull/44115)
    - [Add parent_model param in UploadModelOperator](https://github.com/apache/airflow/pull/42091)
    - [Add CloudRunServiceHook and operator](https://github.com/apache/airflow/pull/40008)
    - [enable AIRFLOW\__CELERY\__BROKER_URL_CMD when passwordSecretName is true](https://github.com/apache/airflow/pull/40270)
- dbt
    - [Fix configuration of turning test warnings into failures](https://github.com/dbt-labs/dbt-core/pull/9347)
    - [Fix package-lock file's bad indentation](https://github.com/dbt-labs/dbt-core/pull/9341)
    - [dbt-bigquery: add test case when raise ServiceUnavailable in is_retryable](https://github.com/dbt-labs/dbt-bigquery/pull/1224)
- kafka-connect-bigquery: [Support n:1 to topic2TableMap (topic:table)](https://github.com/confluentinc/kafka-connect-bigquery/pull/361)
- kafka-connect-transform-tojsonstring: [Support to transtorm record without schema (schemaless json format)](https://github.com/an0r0c/kafka-connect-transform-tojsonstring/pull/18)
- meltano
    - [docs: Fix YAML indent on getting started page and fix link to page source in GitHub](https://github.com/meltano/meltano/pull/7187)
    - [fix(cli): working GCS state backend properly when any files were located in root bucket](https://github.com/meltano/meltano/pull/8648)
- datahub
    - [docs(dbt): fix indentation in dbt meta mapping docs](https://github.com/datahub-project/datahub/pull/7045)
    - [docs: fix image in development](https://github.com/datahub-project/datahub/pull/7637)
    - [docs(google-analytics): Correct grammatical error in README.md](https://github.com/datahub-project/datahub/pull/6870)
    - [docs(architecture): edit documents in architecture section](https://github.com/datahub-project/datahub/pull/6798)
    - [docs(aws): edit markdown link](https://github.com/datahub-project/datahub/pull/6706)
    - [docs(quickstart): enable slack community link](https://github.com/datahub-project/datahub/pull/6209)
- great-expectations: [[DOCS] edit term(data_conext, checkpoints)-link in with airflow](https://github.com/great-expectations/great_expectations/pull/6646)
- mageai
    - [Add postgres client package](https://github.com/mage-ai/mage-ai/pull/5486)
    - [Why bq client create dataset in data loader?](https://github.com/mage-ai/mage-ai/issues/5197) support
- PyAirbyte: [Chore: Remove import only used in TYPE_CHECKING](https://github.com/airbytehq/PyAirbyte/pull/421)