---
{"dg-publish":true,"permalink":"/data/airbyte/__/airbyte-cdk-elasticsearch-rest/","tags":["airbyte","cdk","elasticsearch"],"dgHomeLink":"ture","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":"ture","dgShowTags":true,"noteIcon":"","created":"2024-06-30T00:39:32.589+09:00"}
---



> [!tldr;]
> Airbyte python CDK 로 ElasticSearch 데이터를 수집하는 커넥터를 개발했다. 사용자 쿼리(order by timestamp) 로 [incremental append](https://docs.airbyte.com/understanding-airbyte/connections/incremental-append/) 방식의 수집 방식을 제공한다.


### diagrams


![|700](https://i.imgur.com/MX5IJP5.png)

### reference


- https://docs.airbyte.com/connector-development/cdk-python
- https://docs.airbyte.com/understanding-airbyte/connections/incremental-append
