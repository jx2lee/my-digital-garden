---
{"dg-publish":true,"permalink":"/data/datahub/__/datahub-catalog/","noteIcon":"","created":"2024-06-30T00:39:32.593+09:00"}
---

#datahub #catalog 

---

> 아래 내용들은 Monte Carlo CEO 인 Barr Moses 가 작성한 글을 토대로 작성하였다. 나름 문맥에 맞게 수정했지만 글을 완전히 이해하는데 어려울 수 있다. [socar 기술블로그: 데이터 디스커버리 플랫폼 도입기](https://tech.socarcorp.kr/data/2022/02/25/data-discovery-platform-01.html) 가 잘 읽히기 때문에 먼저 기술블로그 내용을 보시고 본 글을 읽어보는 것을 권장한다.

# tl;dr

- data catalog 는 데이터의 데이터인 메타데이터를 저장하는 inventory 를 나타내며, 데이터 상태/위치/데이터 의미에 대한 정보를 제공한다.
- data discovery 는 전통적인 data catalog 의 단점을 보완한 새로운 접근방식으로, 데이터 계보(lineage)/분산 도메인 지향/확장성/실시간 가시성을 제공한다.
- data discovery platform 은 data discovry 를 구현한 플랫폼으로 오픈소스로는 datahub 가 있다.

# data catalog

- data catalog 를 검색하면, 위키에서는 아래와 같이 정의한다.
	- 데이터베이스의 [개체](https://ko.wikipedia.org/wiki/%EA%B0%9C%EC%B2%B4_(%EC%BB%B4%ED%93%A8%ED%8C%85) "개체 (컴퓨팅)")들에 대한 정의를 담고 있는 [메타데이터](https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0 "메타데이터")들로 구성된 [데이터베이스](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4 "데이터베이스") 인스턴스
	- 기본 테이블, 뷰 테이블, [동의어](https://ko.wikipedia.org/wiki/%EB%8F%99%EC%9D%98%EC%96%B4 "동의어")(synonym)들, 값 범위들, [인덱스](https://ko.wikipedia.org/wiki/%EC%9D%B8%EB%8D%B1%EC%8A%A4 "인덱스")들, [사용자](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9A%A9%EC%9E%90_(%EC%BB%B4%ED%93%A8%ED%8C%85) "사용자 (컴퓨팅)")들, 사용자 그룹 등등과 같은 데이터베이스의 [개체](https://ko.wikipedia.org/wiki/%EA%B0%9C%EC%B2%B4_(%EC%BB%B4%ED%93%A8%ED%8C%85) "개체 (컴퓨팅)")들을 갖고 있.
	- DBMS는 특정 데이터베이스의 구조를 파악하기 위해여 데이터의 타입이나 포맷과 같은 정보를 데이터베이스 카탈로그를 통해 얻는다.
- 데이터 카탈로그는 메타데이터의 저장소(inventory) 역할을 하며 데이터 상태, 접근성 및 위치에 대한 정보를 제공한다.
- 원하는 데이터의 위치 & 데이터가 의미하는 항목 및 사용 방법에 대한 질문을 답하는 데 도움을 준다.

# data discovery

- discovery 가 나오게 된 배경은 다음과 같다.
	- 전통적인 데이터 카탈로그는 웨어하우스의 구조화된 데이터 요구 사항을 충족할 수 있었다.
	- 많은 데이터 카탈로그에는 UI 중심 workflow 가 있지만, 데이터 엔지니어는 프로그래밍 방식으로 카탈로그와 커뮤니케이션 할 수 있는 유연성이 필요했다. (광범위한 데이터 관리 작업을 수행할 수 있도록 API 기반 접근 )
	- 데이터는 여러 통로로 레이크에 입수할 수 있어 엔지니어는 각각 설명할 수 있는 카탈로그가 필요했다.
	- 레이크 내 데이터를 저장하는 것은 저렴하고 유연할 수 있지만, 보유하고 있는 데이터 현황과 어떻게 사용되는지 추적하는 것은 어렵다.
	- 데이터는 다양한 방식(JSON 또는 Parquet 와 같은)으로 저장할 수 있기 때문에 데이터 엔지니어는 때에 따라 데이터 처리 방법을 달리해야했다.
		- summary 작업에 Spark를 사용하거나 Presto를 사용할 수 있다. 즉, 깨지거나 잘못된 데이터로 인해 오류가 발생할 가능성이 많다.
		- 계보(lineage)가 없으면 데이터 레이크 내의 이러한 오류는 자주 발생하고 해결하기 어렵다.
- **data discovery** 은 [데이터 메시 모델](https://martinfowler.com/articles/data-monolith-to-mesh.html)(Zhamak Deghani와 Thoughtworks)을 이용한 분산 도메인 지향 아키텍처 기반의 새로운 접근 방식을 의미한다.
- 도메인별 데이터 소유자는 데이터를 제품으로 만들고 분산된 데이터 커뮤니케이션을 용이하게 하는 책임을 갖는다.
- 다음 네 가지 주요 방법으로 전통적인 데이터 카탈로그를 대체한다.
	1. 확장 가능한 자동화
		- 테이블 및 필드 수준 계보 추적을 자동화하여 업스트림 및 다운스트림 종속성을 매핑한다.
		- 데이터가 발전함에 따라 data discovery 를 통해 데이터와 데이터 사용 방식을 이해할 수 있다.
	2. 데이터 상태(health)에 대한 실시간 가시성(real-time visibility)
		- 전통적인 data catalog 와 달리 data discovery 는 데이터의 **카탈로그** 또는 데이터의 현재 상태의 실시간 가시성을 제공한다.
		- 데이터를 수집, 저장, 집계 및 사용하는 방법을 포함하여 어떤 데이터 세트가 오래되어 삭제할 수 있는지, 주어진 데이터 세트가 좋은 품질인지, 언제 사용되는지와 같은 insight 를 얻을 수 있다.
	3. 비즈니스에 영향을 주는 데이터 계보(lineage)
		- 리니지를 사용하면 스키마 변경과 같이 자주 눈에 띄지 않는 문제를 감지할 수 있다.
		- 데이터 파이프라인이 중단될 때 문제를 더 빠르게 해결할 수 있다,.
	4. 도메인 간 셀프 서비스 검색
		- 셀프 서비스를 가능하게 하여 전담팀 없이 데이터를 쉽게 활용하고 이해할 수 있다.
	5. 거버넌스 및 최적화
		- data discovery 를 통해 기업은 수명 주기 동안 **어떤 데이터가 사용/소비/저장/폐기되는지** 이해할 수 있을 뿐만 아니라, 데이터 거버넌스에 매우 중요하고 레이크 전체에서 최적화에 사용할 수 있는 insight 을 얻을 수 있다.
- data discovry 는 보편적 거버넌스(universal governence) 및 접근성 표준(accessibility standard)을 준수하면서 데이터 스택 여러 부분에 걸친 데이터의 분산된 실시간 통찰력(insight)을 제공한다. 이는 data catalog 를 대체하거나 보완할 수 있다.

	![](https://miro.medium.com/max/640/0*COcB9K9ihOLUrSzr)


# data discovery platform (DDP)
- data discovery 를 구현한 플랫폼 (==metadata platform==) 을 의미한다
- 웹 UI 를 제공한다.
- 데이터 구조와 관계 등 메타데이터를 쉽게 확인하고 검색할 수 있다.
- features
	- ElasticSearch 또는 Solr 기반 free-text 검색을 지원한다.
	- 테이블 정보/미리보기/컴럼통계를 제공한다.
	- 데이터 계보(Lineage) 를 제공한다.
	- 대표적인 오픈소스는 다음과 같다.
		- [Datahub](https://github.com/datahub-project/datahub)
		- [Apache Atlas](https://github.com/apache/atlas)
		- [Amundsen](https://github.com/amundsen-io/amundsen)
		- [Metacat](https://github.com/Netflix/metacat)
- Datahub/Atlas 의 경우 최근 커밋 이력이 존재하지만, Amundsen/Metacat 는 없는 월 단위로 존재했다.
	- **그만큼 웰메이드 제품일 수 있지만 비주류에 속할 수 있다.**
- 두 개 프레임워크를 비교하며 결정하는 방법이 좋지만, 이미 개발 환경에 구축한 datahub 를 이용하고자 한다.
- Atlas 는 apache 오픈소스 프로젝트이다 보니 apache 프로젝트들로 구성한 데이터 환경이면 강력히 도입할 것 같다. 그렇다고 datahub 가 apache 프로젝트를 지원하지 않는 건 아니다.

# datahub
## overview

> DataHub is a modern data catalog built to enable end-to-end data discovery, data observability, and data governance. This extensible metadata platform is built for developers to tame the complexity of their rapidly evolving data ecosystems and for data practitioners to leverage the total value of data within their organization.

- DataHub 는 data discovery/observability/governence 를 지원하도록 구축된 모던 데이터 카탈로그다. **(확장 가능한 메타데이터 플랫폼)**
- 개발자는 복잡한 데이터 생태계에 적응하고 데이터 실무자가 데이터 가치를 창출 할 수 있다.
- features
	1. **Search and Discovery**
		- 모든 데이터 stack 에 대한 검색을 지원한다.
		- Lineage 추적 기능을 제공한다.
		- Impact Analysis 을 이용한 메타데이터 변경 영향을 확인할 수 있다.
		- 메타데이터를 결합하여 데이터 엔터티 관찰이 용이하다.
	2. **Modern Data Governance**
		- [Action Framework](https://datahubproject.io/docs/actions/) 를 이용한 실시간 유스케이스는 다음과 같다. 상당히 유용하다.
			- Notifcations
			- Workflow Integration
			- Synchronization
			- Auditing
		- Entity Ownership 을 관리한다. (*데이터 주인은 누구?*)
		- Tag, Glossary Terms & Domain 으로 메타 데이터를 관리한다.
			- Tag: 검색 및 발견을 위한 도구 역할을 하는 비공식적이고 느슨하게 제어하는 레이블이다. 공식적인 중앙 관리가 없다. 즉 태그 관리 권한만 있다면 생성/삭제 가 가능하다.
			- Glossary Term: 핵심 비즈니스 개념 및 측정을 설명하는 데 일반적으로 사용되는 선택적 계층 구조(hierarchy) 가 있는 어휘입니다.
			- Domain: 부서(예: 재무, 마케팅) 또는 데이터 제품별로 엔터티를 구성하기 위해 사용되는 최상위 폴더 또는 범주를 의미한다.
	3. **DataHub Administration**
		- 사용자, 그룹 및 액세스 정책을 관리한다.
		- UI 에서 메타데이터를 수집할 수 있다. (간혹 수집 데이터를 rollback 할 때 유용하다.)

# reference

- [https://towardsdatascience.com/data-discovery-the-future-of-data-catalogs-for-data-lakes-7b50e2e8cb28](https://towardsdatascience.com/data-discovery-the-future-of-data-catalogs-for-data-lakes-7b50e2e8cb28)
- [https://towardsdatascience.com/the-missing-piece-of-data-discovery-and-observability-platforms-open-standard-for-metadata-37dac2d0503](https://towardsdatascience.com/the-missing-piece-of-data-discovery-and-observability-platforms-open-standard-for-metadata-37dac2d0503)
