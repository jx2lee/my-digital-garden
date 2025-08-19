---
{"author":"jx2lee","aliases":"Apache Spark","created":"2024-11-21T21:58:58.000+09:00","last-updated":"2024-11-21 21:58","tags":["spark"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":true,"dg-pass-frontmatter":false,"permalink":"/data/etc/spark/","dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"dgPassFrontmatter":true,"noteIcon":""}
---


### Definition
> Apache Spark™ is a multi-language engine for executing data engineering, data science, and machine learning on single-node machines or clusters.

[공식 홈페이지에 따르면,](https://spark.apache.org/)
- 단일 노드 혹은 클러스터에서 데이터 엔지니어링, 데이터 사이언스, 머신러닝을 위한 다중 언어(프로그래밍) 엔진
- Keyword
    - Simple
    - Fast
    - Scalable
    - Unified(통합)
- SQL 엔진 지원. ANSI SQL, 정형/비정형 처리
- IMO: 분산처리에 대한 내용이 공식 홈페이지에 언급되어 있지 않음. "분산"이 키워드로 있을 줄 알았는데.. 의아했다. 이 키워드는 기본으로 제공하기 때문에 언급되지 않은 건가?
    - distributed 는 Spark SQL 부분에 언급한다.
    - 분산 쿼리 엔진을 지원한다. Spark 특징보단 SparkSQL 설명에 쓰인걸 보면 스파크의 SQL 컴포넌트의 장점으로 활용가능하다 로 이해하면 될 듯
- [공식 Documentation 에 따르면,](https://spark.apache.org/docs/latest/) [Apache Spark is a unified analytics engine for large-scale data processing](https://spark.apache.org/docs/latest/#:~:text=Apache%20Spark%20is%20a%20unified%20analytics%20engine%20for%20large%2Dscale%20data%20processing):  대용량 데이터 처리를 위한 통합 분석 엔진. 공식 홈페이지와 크게 차이가 없다.


### 그냥저냥


### References
- [Spark Shuffle Partition과 최적화](https://tech.kakao.com/posts/461)