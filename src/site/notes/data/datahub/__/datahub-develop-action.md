---
{"dg-publish":true,"permalink":"/data/datahub/__/datahub-develop-action/","noteIcon":"","created":"2024-06-30T00:39:32.593+09:00"}
---

#datahub #action #develop
# overview
- column description 에 특정 키워드를 포함하는 경우 이벤트를 캐치하고 태그를 붙이는 action 을 개발한다.
- action 은 다음 상황일 때 동작한다.
	- column description 이 변경된 경우

# environment
- DataHub 로컬
	- [링크](https://datahubproject.io/docs/docker/development)를 참고하면 빠른 로컬 환경을 구성할 수 있다.
	- 이전 [multipass](https://multipass.run) 를 이용했지만.. Docker Desktop 으로 도커를 설치했다.
- sample metadata
	- 사내 mssql 소스를 file 로 받아오는 recipe 로 몇몇 테이블만 수집했다. 이후 특정 키워드로만 이벤트를 캐치하기 위해 description 을 수정하였다. (샘플 양 -> 1개 스키마: 8개 테이블)

# Develop
사실 개발은 [도큐먼트](https://datahubproject.io/docs/actions/guides/developing-an-action)를 참고하면 쉽게 가능하다. 따라서 링크만 남겨두고 구현한 내용만 간략히 공유하고자 한다.

## goal 
## action workflow
## implementation details
```python
```

# conclusion
