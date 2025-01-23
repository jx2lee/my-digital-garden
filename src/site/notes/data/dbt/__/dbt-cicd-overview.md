---
{"dg-publish":true,"permalink":"/data/dbt/__/dbt-cicd-overview/","noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---

#dbt #airflow 

---

# tl;dr

- bamboo(build plan & deployment project) 를 이용한 dbt ci/cd 를 airflow 로 이관했다.
- 이미지 빌드를 airflow DAG 으로 수행하는 것은 의도와 맞지 않다 판단하여 기존 bamboo 의 build plan 만 사용했다.
	- stack: Atlasian Bamboo, Docker, Airflow

# overview
1. dev branch push 발생
	- stash repository trigger 로 build plan 을 실행한다.
		- dev 브랜치를 기반으로 container image 를 빌드하고 푸시한다.
			- tag: dbt-run-dev-{build_number}, latest
		- ligthdash sync DAG 을 [트리거](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 한다. ([rest API](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html))
	- build plan 을 성공하면 Atlasian deployment 를 실행한다.
		- build plan 으로 push 한 image 를 이용하여 helm chart 를 업그레이드 한다.
		  - helm upgrade: dbt build 를 수행하는 단계이다.
2. main branch push 발생 시
	- dev branch 코드 변경 시 발생하는 절차는 동일하다. 단, image 를 업로드하고 바라보는 EKS 는 DEV/PRD 로 나누었기 때문에 PRD 에 해당하는 부분들만 변경한다.
3. scheduled: 매일 오전 8시 main branch 기준
	- build plan 을 실행한다.
		- dbt full build 로 전사 지표에 사용하는 모델들을 업데이트 한다.
4. dev/main 작업 시 모든 모델들은 full build 로 빌드된다.
	- **변경한 모델만 빌드하지 않고 모든 모델들을 빌드한다.**

# problem
- Atlasian build/deployment 에 대한 허들이 높다.
	- dbt 모델을 관리하는 분석가가 쉽게 build/deployment 수정하기에 접근이 어렵다. (~~gitlab 도입해줘요 휴먼~~)
- 한 가지 모델을 변경한 경우에도 모든 모델을 다시 빌드한다.
	- 의존하지 않는 모델들도 모두 재생성하기 때문에 비용이 더 발생한다.
	- 모델이 많아질수록 빌드시간이 오래걸린다.

# to-be
1. dev branch push 발생
	- 아래와 같이 build plan 을 실행한다.
		- dev 브랜치 기반 container image 를 빌드하고 ECR repository 로 push 한다.
			- tag: latest
		- ligthdash sync DAG 을 [트리거](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 한다.
		- dbt build 를 실행하는 DAG 을 [트리거](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 한다. 단, 변경된 모델만 build 하는 [slim build](https://docs.getdbt.com/guides/legacy/best-practices#run-only-modified-models-to-test-changes-slim-ci) 로 변경했다.
2. main branch push 발생
	- 아래와 같이 build plan 을 실행한다.
		- main 브랜치 기반 container image 를 빌드하고 ECR repsitory 로 push 한다.
			- tag: latest
		- ligthdash sync DAG 을 [트리거](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 한다.
		- dbt build 를 실행하는 DAG 을 [트리거](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 한다.
			- dev 브랜치와 동일하게 slim build 로 실행한다.
3. scheduled: 매일 오전 8시 main branch 기준
	- build plan 을 실행한다.
		- dbt full build 로 전사 지표에 사용하는 모델들을 업데이트 한다.
4. 변경 사항만 실행하는 slim build 를 이용하고 지표 활용에 필요한 오전 8시 스케쥴 빌드는 full build 를 이용한다.

# improvements
- slim CI 를 이용해 변경한 모델들만 실행하도록 변경하여 빌드 속도를 개선했다.
	- 이전 (full build): 206.439s
	- 변경 후: 77.8919s
- 특정 tag 만 build 하는 등 다양한 옵션으로 build 할 수 있도록 개선했다.
	- dbt build 를 실행하는 entrypoint.sh 스크립트에 원하는 필요한 동작들을 추가할 수 있도록 변경했다.
	- airflow schedule_interval 수정을 통해 원하는 시간대 실행할 수 있다.
- 분석가 지향 CI/CD 환경을 제공한다.
	- Atlasian 보다 익숙한 Airflow 로 이관하여 문제 발생 시 분석가도 빠르게 대응할 수 있다.
		- ~~기왕이면 gitlab 도입하면 좋읍읍~~

# additional improvements
- docker image 태그 변경
	- 변경한 CI/CD 에 사용하는 Docker 이미지의 태그 관리가 필요하다. base & 실제 build 를 수행하는 이미지 모두 한 repository 에 존재하며, dbt build 를 수행하는 이미지 태그가 latest 로 설정했다. 그러다보니 롤백이 필요한 경우 특정 시점 확인이 불가하여 바로 수정해야하는 문제가 있다.
	- **base & build 이미지를 분리하고 tag 명을 지라 티켓으로 명시할 수 있다면 좋을 것 같다.**
- dbt build 고도화
	- 이미지를 실행하는 entrypoint.sh 에서는 `dbt build` 를 수행하고 있다.
	- dbt build 는 run, test, snaptshots 그리고 seed 커맨드를 포함하고 있다.
	- **불필요한 일들을 제어하고 최적화하는 작업이 필요하다.**

# reference
- [데이터에 신뢰성과 재사용성까지, Analytics Engineering with dbt](https://tech.socarcorp.kr/data/2022/07/25/analytics-engineering-with-dbt.html)
- [How to use Slim CI with dbt Core](https://www.vantage-ai.com/blog/how-to-use-slim-ci-with-dbt-core)