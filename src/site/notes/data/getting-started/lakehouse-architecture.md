---
{"author":"jx2lee","aliases":null,"created":"2024-06-30T00:39:32.000+09:00","last-updated":"2023-08-25 19:33","tags":null,"dg-publish":true,"permalink":"/data/getting-started/lakehouse-architecture/","dgPassFrontmatter":true,"noteIcon":""}
---


### Functional Requirements
- 카탈로그
	- 기록의 보유 여부 확인이 가능해야 한다.
- 데이터 소스
	- 데이터베이스와 백업 파일 스토리지의 데이터를 사용할 수 있어야 한다.
	- 권한 관리
		- 허가 받은 사용자에 한하여 접근이 가능해야 한다.
	- 조회 도구
		- 사용자가 직접 SQL을 사용해 데이터를 조회할 수 있도록 한다.   
		- 사용자를 위한 웹 페이지와 쿼리 작성을 위한 에디터가 제공되어야 한다.  
    - 기록
		- 조회 시 사용된 SQL과 조회 사용자, 시간 등을 기록하고 감사 기록 형태로 보관한다.
		- 사용자를 위한 이전 조회 기록 목록 조회용 웹 페이지와 재실행 기능이 제공되어야 한다.

### checklist
- ingestion
	- 원본 source 들을 analytics engine 이 읽을 수 있는 포맷으로 상시 저장하고 있어야 하는가
		- [ ] 조회 대상 테이블이 정해져있는가
			- 정해져 있다면 어떤건지
			- 정해져 있지 않다면 요청마다 변경되는것인지
		- [ ] 요청 주기는 어느정도 인가
	- IMO
		- 어차피 오로라 3 버전으로 올라가면 redshift 와 함께 storage layer 를 공유할 수 있다. 이전까지 효율적으로 ingestion 할 수 있는 방법을 생각해보자.


### 로컬테스트
- [x] duckdb with s3 (httpfs)
- [ ] local spark cluster
    - [[spark-standalone-mode-on-docker\|로컬에서 spark cluster 세팅해보기]]
- [ ] prepare sample dataset

### references
