---
{"dg-publish":true,"permalink":"/data/elasticsearch/__/es-index-api/","tags":["elasticsearch"],"noteIcon":"","created":"2024-06-30T00:39:32.602+09:00"}
---


> [!tldr]
> 엘라스틱서치 인덱스와 관련된 API 를 간략하게 정리한다.


### aliases API
---


- Index 에 별칭을 부여할 수 있는 API
- `{es_address:port}/_aliases`
- 하나 또는 그 이상의 index 에 별칭 설정 가능
- 단, 두 개 이상 index 에 별칭을 사용한 경우 제약사항
    - 검색만 가능, 색인은 불가능
    - index 상태가 closed 인 경우
- [more..](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html)


### rollover API
---


- Index 에 특정 조건을 설정하여 해당 조건을 만족하면 새롭게 Index 생성 및 요청
- `{es_address:port}/{alias}/_rollover`
- aliases API 가 필수적으로 선행
- `conditions` field 로 조건 설정 가능
- \_rollover URL 에 특정 Index 명을 작성하면 새롭게 생성된 Index 이름으로 생성
[[함\|함]]
- dry_run 가능 (query parameter)
- [more..](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-rollover-index.html)


### Refresh API
---

- refresh_interval 과 상관없이 메모리 버퍼 캐시에 있는 문서들을 바로 세그먼트로 저장하여 검색가능한 상태로 변경
- `{es_address:port}/{index}/_refresh`
- [more..](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-refresh.html)


### forcemerge API
---


- segment 를 강제로 병합하는 API
- `{es_address:port}/{index}/_forcemerge`
- max_num_segments 로 병합하고자 하는 segment 수를 설정 가능함 (query parameter)
- 방대한 segment 수를 설정하면 디스크 IO 작업으로 성능 저하 발생할 수 있고 색인이 일어나는 Index 의 경우 이 API 사용은 지양하는 것이 좋다.
    - 더 이상 세그먼트에 대한 변경 작업이 일어나지 않은 과거 데이터의 Index 에 활용하면 좋다.
- [more..](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-forcemerge.html)


### reindex API
---


- Index 를 복제하는 API
- `{es_address:port}/_reindex}`
- `source & dest` field 를 사용하여 소스와 타겟을 설정
- Index 의 Analyzer 변경 시 주로 사용
    - 이미 색인이 완료된 Index 의 Analyzer 는 변경해도 아무런 변화가 없음
    - Analyzer 를 변경한 Index 를 생성, 이후 reindex API 로 migration 진행
- 클러스터 간 migration 작업은 elasticsearch.yml 의 `whitelist` 설정으로 reindex API 사용 가능
    - A -> B 클러스터 간 migration 을 진행하는 경우
    - B 클러스터의 elasticsearch.yml: reindex.remote.whitelist 설정
    - 마찬가지로 API 호출은 B 에서 진행, `source` field 에 remote.host 작성 필요
- [more..](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)


### Reference
[기초부터 다지는 Elasticsearch 노하우](https://product.kyobobook.co.kr/detail/S000001033104)

