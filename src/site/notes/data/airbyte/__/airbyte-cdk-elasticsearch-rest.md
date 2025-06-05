---
{"author":"jx2lee","aliases":["Elasticsearch REST api with Airbyte CDK"],"created":"2024-06-30T00:39:32.000+09:00","last-updated":"2023-08-31 22:59","tags":["airbyte","cdk","elasticsearch"],"dg-publish":true,"dg-home-link":"ture","dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":"ture","dg-show-tags":true,"dg-pass-frontmatter":false,"permalink":"/data/airbyte/__/airbyte-cdk-elasticsearch-rest/","dgHomeLink":"ture","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":"ture","dgShowTags":true,"dgPassFrontmatter":true,"noteIcon":""}
---



> [!tldr;]
> Airbyte python CDK 로 ElasticSearch 데이터를 수집하는 커넥터를 개발했다. 사용자 쿼리(order by timestamp) 로 [incremental append](https://docs.airbyte.com/understanding-airbyte/connections/incremental-append/) 방식의 수집 방식을 제공한다.


### diagrams


![|700](https://i.imgur.com/MX5IJP5.png)

### reference


- https://docs.airbyte.com/connector-development/cdk-python
- https://docs.airbyte.com/understanding-airbyte/connections/incremental-append
