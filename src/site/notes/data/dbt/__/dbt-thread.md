---
{"dg-publish":true,"permalink":"/data/dbt/__/dbt-thread/","tags":["dbt","thread"],"noteIcon":"","created":"2024-06-30T00:39:32.599+09:00"}
---

### definition

dbt가 실행되면 모델 간 링크의 DAG 이 생성된다.
`thread` 는 dbt가 한 번에 작업할 수 있는 그래프의 최대 경로 수를 나타내며, 수를 늘리면 프로젝트의 런타임을 최소화할 수 있다. 기본값은 4이다.

예를 들어 `thread` 를 1로 지정하면 dbt는 하나의 모델만 빌드를 시작하고 완료한 후 다음 모델로 이동한다. `thread` 값이 8이라는 것은 종속성(dependency)을 위반하지 않고 한 번에 최대 8개 모델을 작업할 수 있음을 의미한다.
- 실제 작업할 수 있는 모델 수는 종속성 그래프에서 사용 가능한 경로에 의해 제한될 수 있다.

### Considerations

설정할 수 있는 최대 `thread` 는 제한이 없다. `thread` 수를 늘리면 실행 시간이 단축되지만 고려해야 할 사항이 많다.
- `thread` 를 늘리면 데이터 웨어하우스의 부하가 증가하여 다른 도구에 영향을 미칠 수 있다. 예를 들어, BI 도구가 dbt와 동일한 컴퓨팅 리소스를 사용하는 경우 **dbt 실행 중에 해당 쿼리가 큐에 대기**될 수 있다.
- 많은 thread 를 사용하면 구축할 수 있는 모델 수를 제한하는 문제를 일으킬 수 있다. 동시 실행 limit 에 걸려 일부 모델은 사용 가능한 쿼리 슬롯을 기다리는 동안 대기할 수 있다.
- **일반적으로 최적의 스레드 수는 데이터 웨어하우스와 그 구성에 따라 다르다**. 다양한 값을 테스트하여 프로젝트에 가장 적합한 스레드 수를 찾는 것이 가장 좋다. 처음에는 4로 설정해보자.

dbt 명령을 실행할 때 `--threads` 옵션을 사용하여 대상에 정의된 값과 다른 스레드 수를 사용할 수 있습니다.

### references
[Understanding threads​](https://docs.getdbt.com/docs/core/connection-profiles#understanding-threads "Direct link to heading")