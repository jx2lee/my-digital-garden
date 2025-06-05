---
{"author":"jx2lee","aliases":"MageAI","created":"2024-06-30T00:39:32.000+09:00","last-updated":"2024-10-18 22:21","tags":["mageai"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/data/getting-started/mage/","dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---



airflow 를 대체할 수 있다고 자부(🧙 A modern replacement for Airflow) 하는 [mage](https://docs.mage.ai/introduction/overview) 를 살펴본다.

### background
---
데이터셀에서 사용중인 dbt 에 불편사항들이 존재한다.
1. 환경 설정이 번거롭다
	- python 설치도 안되어 있는 사용자도 있다.
	- poetry 설치가 잘 안된다. (poetry path 설정)
	- dbt dependency 설치가 필요하다.
2. backfill 작업이 번거롭다.
	- incremantal model 의 컬럼 추가 작업 시 기존 테이블 삭제 및 재실행이 필요하다.
	- **수동 테이블 삭제(콘솔 이용)와 dbt run 이 필요하다.**
3. (분석가가 사용하기에) dbt CLI 가 까다롭다.
4. main 브랜치에 대한 run을 할 수 없다.
	- 응급 상황에 대한 대처가 어렵다.

개선된 환경을 위해 고민하던 중 mage 도구를 알게되었고, 도입이 가능한 지(위 불편사항들을 제거할 수 있는지) 검토하고자 한다.

### 💁‍♂️ mage?
---
![|400](https://i.imgur.com/QTiMkEi.png)

- 🧙 A modern replacement for Airflow.
- [core design](https://docs.mage.ai/design/core-design-principles)
	- 💻 Easy developer experience
	- 🚢 Engineering best practices built-in
	- 💳 Data is a first class citizen
	- 🪐 Scaling is made simple
	- component
		- [Block](https://docs.mage.ai/design/blocks)
			- 파이프라인의 각 블록은 프로젝트 내 개별 파일과 매핑된다.
			- 블록은 동일 프로젝트 내 여러 파이프라인에서 재사용 및 공유할 수 있다.
		- [Pipeline](https://docs.mage.ai/design/core-abstractions#pipeline)
			- 실행하려는 모든 코드 블록에 대한 참조, 시각화를 위한 차트를 포함한다.
			- 각 블록 간의 종속성을 구성한다.
			- ![|500](https://i.imgur.com/YkJnzRd.png)
		- [Trigger](https://docs.mage.ai/guides/triggering-pipelines#trigger)
			- 파이프라인을 실행해야 하는 방법을 결정한다.
			- 하나의 파이프라인에는 1개 이상 trigger 를 사용할 수 있다.
			- Trigger type: `Schedule`, `Event`, `API`
- 로컬 환경 세팅
	- https://docs.mage.ai/getting-started/setup
- [demo video](https://www.youtube.com/watch?v=hrsErfPDits)

### 기능확인
---
- 테스트 버전: v0.9.3
- 환경: docker
- 확인 내용
	- background 언급내용

#### 🙆‍♂️ 환경 설정이 번거롭다
- mage 안에 dbt 패키지가 모두 설치되어 있다.
	- v0.9.4 기준 dbt v1.4.0 설치
	- 테스트를 위해 현재 우리가 사용중인 dbt 버전으로 변경했다.
		- ![|400](https://i.imgur.com/pWb3Ep4.png)
- mage root 디렉토리/dbt 폴더 안에 기존 dbt 프로젝트를 옮겨야 사용할 수 있다.
	- sidecar pattern 을 이용해 dbt-metric repo 를 sync 할 수 있다.
	- mage 에서 [git sync](https://docs.mage.ai/production/data-sync/git#git-sync) 기능을 제공한다.
- dbt 패키지나 python 패키지를 일일이 설치할 필요가 없다. 따라서 통합 환경을 제공할 수 있다.
	- **단, 코드 수정 시 라이브로 상대방 코드가 변경되지 않는다.** 
	- 새로고침을 해야 수정사항이 보인다.

#### 🙆‍♂️ backfill 작업이 번거롭다
- incremental 모델을 rebuild 할 수 있는 [document](https://docs.mage.ai/dbt/incremental-models#how-to-rebuild-incremental-models) 도 제공하고 있다.
	- [trigger](https://docs.mage.ai/guides/triggering-pipelines) 기능을 활용한다.
	- 자세한 내용은 document 를 살펴보자.
	- 테스트로 incremental 모델을 full-refresh 하는 파이프라인을 생성하였고 정상적으로 동작했다.
- incremental 모델의 스키마 변경이 있을 때 위 기능으로 대체할 수 있다.
	- incremental 모델의 rebuild 테스트는 링크를 [참고](http://localhost:6789/pipelines/incremental_full_refresh_test/runs/7)해주세요.

#### 🤷‍♂️ (분석가가 사용하기에) dbt CLI 가 까다롭다
- 새로운 모델 생성 시 테스트하기 편리하지만
	- 기존 모델만 Import 해서 사용하는 경우
	- 결국 CLI 플래그들을 숙지하고 있어야 사용하기 편리하다.
- **커스텀한 dbt run 을 사용할 경우 까다로운 CLI 는 해결할 수 없을 것으로 예상한다.**

#### 🙆‍♂️ main 브랜치에 대한 run을 할 수 없다
- DBT model 에 대한 다양한 block 을 제공한다.
	- ![](https://i.imgur.com/cYwW209.png)
		- New model
		- Single model or snapshot (from file, dbt-metric 프로젝트에 정의한 모델을 사용할 수 있음)
		- All models
		- Generic dbt command
- generic dbt command 는 command 를 직접 설정하여 실행할 수 있다.
	- ![](https://i.imgur.com/I7DbQ7r.png)
	- 매일 오전 8시에 실행하는 full-build 에 실패하는 경우, **실패한 특정 모델을 실행할 수 있다.**
- 특정 모델을 재실행할 수 있는 Pipeline 을 생성하여 빠른 대응이 가능하다.

### IMO
---
- 처음 도구를 접했을 때 **활용할 수 있을까?** 라는 생각이 들었다.
- 기능들을 하나하나 뜯어보고 나니 dbt integration 을 많이 지원하고 있다.
- **dbt 를 활용하는 환경에서 편리하게 사용할 수 있지 않을까** 생각이 바뀌었다.
- 도입 시 추가로 살펴봐야 하는 부분은
	- credential 관리
	- 접근 제한
		- oauth2-proxy 로 해결가능해 보임
	- 등을 관리해야한다.
- 만족도: ⭐️⭐️⭐️⭐️ (5개 중)
