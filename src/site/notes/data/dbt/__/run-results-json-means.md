---
{"dg-publish":true,"permalink":"/data/dbt/__/run-results-json-means/","tags":["dbt"],"noteIcon":"","created":"2024-06-30T00:39:32.600+09:00"}
---


> [!tldr] CLI 이후 생성되는 run-results.json 에 대해 살펴본다.

### overview


- 실행된 각 노드(model, test 등)에 대한 시간 및 상태를 포함한 dbt 호출정보
- 모델 평균 런타임, 테스트 실패율, 스냅샷으로 캡처된 레코드 변경 수 확인
- 실행된 노드만 결과에 표시
- 다른 여러 실행 or 테스트 단계가 있는 경우 각기 다른 실행 결과를 생성한다.

> [We use the terms nodes and resources interchangeably. These encompass all the models, tests, sources, seeds, snapshots, exposures, and analyses in your project. They are the objects that make up dbt's DAG (directed acyclic graph).](https://docs.getdbt.com/reference/node-selection/syntax)
> 

- dbt 에서는 node 와 resource 라는 용어를 같은 의미로 사용
- 모든 모델, 테스트, 소스, 시드(seed), 스냅샷, exposures 및 analyses
- **dbt 의 DAG(directed acyclic graph)을 구성하는 객체라고 이해하자**

### reference


- [https://docs.getdbt.com/reference/artifacts/run-results-json](https://docs.getdbt.com/reference/artifacts/run-results-json)