---
{"dg-publish":true,"permalink":"/data/getting-started/elasticsearch-scroll-api/","tags":["elasticsearch","scroll"],"noteIcon":"","created":"2024-06-30T00:39:32.588+09:00"}
---


**tl;dr** elasticsearch 에서 제공하는 paginataion 기능 중 scroll API 를 간단히 살펴봐요

### overview

- es 데이터를 bigquery 에서 보고싶다는 요청이 발생했어요.
- 검색엔진 -> dw 파이프라인은 매력적이지 않지만 필요하다기에 적재하려던 중,
	- 반환하는 최대 도큐먼트 수는 10,000 이었고
	- pagination 기능이 있나 살펴보던중 `search after` 가 있었다.
	- 하지만 사용중인 es 버전이 낮아 사용이 불가했어요.
- `scroll api` 를 사용해야했고 이를 기록용으로 남겨요.

### scroll API
[documnet](https://www.elastic.co/guide/en/elasticsearch/reference/current/scroll-api.html)
- scroll API를 사용하여 대량의 결과를 검색할 수 있어요.
- scroll API 를 이용하려면 scroll ID 가 필요한데,
    - ID를 얻으려면 처음 쿼리 매개변수에 대한 인수가 포함된 API 이용해야해요.
    - `scroll` query param 은 Elasticsearch 가 컨텍스트를 얼마나 오래 유지해야 하는지를 의미하는데, 예시를 살펴볼게요.
    - {endpoint}?scroll=1m
        - 해당 scroll ID 를 이용하여 검색할 수 있는 시간은 1분이에요. 1분 이후 해당 scroll ID 로 검색할 수 없어요.
- 응답은 \_scroll_id 매개변수에 스크롤 ID를 반환해요.
    - 그런 다음 스크롤 API와 함께 스크롤 ID를 사용하여 다음 결과를 검색할 수 있어요.
    - Elasticsearch 보안 기능이 활성화된 경우, 특정 스크롤 ID의 결과에 대한 액세스는 검색을 제출한 사용자 또는 API 키로 제한할 수 있어요.
- 스크롤 API는 검색 컨텍스트의 보존 기간을 연장하거나 단축하는 새로운 스크롤 매개변수를 지정할 수도 있어요.

> 요청에서 반환되는 결과는 **스냅샷**처럼 초기 검색 요청이 이루어진 시점의 색인 상태를 반영해요. 만약 문서에 대한 변경(색인, 업데이트 또는 삭제)이 발생한 경우 스크롤 API 로 조회 시 반영되지 않아요.

### 사용해보자

### tips
`GET /_nodes/stats/indices/search`
- [nodes stats API](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/cluster-nodes-stats.html) 로 현재 생성된 search context 를 조회할 수 있어요.
- 스크롤 API 로 생성된 context 들을 확인하며 불필요하게 생성된 컨텍스트가 있는지 확인할 수 있어요.
### reference
- https://www.elastic.co/guide/en/elasticsearch/reference/6.5/search-request-scroll.html#search-request-scroll
- https://opster.com/guides/elasticsearch/search-apis/elasticsearch-score
- https://stackoverflow.com/a/41309914
- https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#common-options-response-filtering