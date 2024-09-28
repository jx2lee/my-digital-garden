---
{"dg-publish":true,"permalink":"/data/airflow/__/dag-factory-in-action/","tags":["airflow","dag-factory","getting-started"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"noteIcon":"","created":"2024-06-30T00:39:32.591+09:00"}
---


### intro
[dag-factory](https://github.com/ajbosco/dag-factory) 라는 yaml 기반 동적 DAG 생성 프로젝트를 간단히 살펴본다.

dag-factory는 YAML 로 Apache Airflow DAG를 동적으로 생성하기 위한 라이브러리이다. ([참고](https://github.com/ajbosco/dag-factory#dag-factory))



### prerequisites
- airflow
- curiosity

### follow readme
bashOperator 로 구성한 간단한 dag 를 실행해보도록 한다.

#### simple dag's yaml
```yaml
example_dag1:  
  default_args:  
    owner: 'jj.lee'  
    email: 'jj.lee@coinone.com'  
    email_on_failure: False  
    email_on_retry: False  
    retries: 1  
    retry_delay_min: 10  
    start_date: 2022-09-01  
  catup: False  
  schedule_interval: '0 0 1 * *'  
  concurrency: 1  
  max_active_runs: 1  
  default_view: 'tree'  # or 'graph', 'duration', 'gantt', 'landing_times'  
  orientation: 'LR'  # or 'TB', 'RL', 'BT'  
  description: 'this is an example dag!'  
  on_success_callback_name: print_hello  
  on_success_callback_file: /opt/airflow/plugins/print_hello.py  
  on_failure_callback_name: print_hello  
  on_failure_callback_file: /opt/airflow/plugins/print_hello.py  
  tasks:  
    task_1:  
      operator: airflow.operators.bash_operator.BashOperator  
      bash_command: 'echo 1'  
    task_2:  
      operator: airflow.operators.bash_operator.BashOperator  
      bash_command: 'echo 2'  
      dependencies: [task_1]  
    task_3:  
      operator: airflow.operators.bash_operator.BashOperator  
      bash_command: 'echo 3'  
      dependencies: [task_1]  
  
default:  
  default_args:  
    owner: "default_owner"  
    retries: 1  
    retry_delay_sec: 300  
    start_date: 2022-09-01  
  concurrency: 1  
  max_active_runs: 1  
  dagrun_timeout_sec: 600  
  default_view: "tree"  
  orientation: "LR"  
  schedule_interval: "0 1 * * *"  
  on_success_callback_name: print_hello  
  on_success_callback_file: /opt/airflow/plugins/print_hello.py  
  on_failure_callback_name: print_hello  
  on_failure_callback_file: /opt/airflow/plugins/print_hello.py
```

#### example_dag_factory.py
**이 파일은 dag 으로 인식할 수 있는 폴더내에 위치한다.**

```python
from airflow import DAG  
import dagfactory  
  
dag_factory = dagfactory.DagFactory("/opt/airflow/dags/config/example.yml")  
  
dag_factory.clean_dags(globals())  
dag_factory.generate_dags(globals())
```

### caution
- on_success_callback 으로 사용하는 print_hello.py 의 경로는 해당 스케쥴러의 absolute path 로 작성해야한다.
	- 마찬가지로 custom operator(on_success_callback_file(name)) 의 경우 절대 경로를 작성해야한다.
- 로컬환경 구성 중 계속 config 파일을 못읽어오는 문제가 발생할 수 있다. dag-factory 패키지가 scheduler 및 worker 에 정상적으로 설치되었는지 확인하자.

### result
![](https://i.imgur.com/aXaSucv.png)

code 는 dag-factory 를 불러오는 코드가 보인다.
그래프를 확인해보자.

![](https://i.imgur.com/qLQbPPs.png)

yaml 로 정의한 그래프가 잘 보인다.

![](https://i.imgur.com/hwhUK3z.png)

결과도 깔끔하다.

# Conclusion
- warehouse 용 DAG 은 수정사항이 많지 않아 도입한다면 yaml 기반 DAG 관리로 조금 더 편할 것으로 예상한다.
- 작업하면서 느낀 장점과 단점은 다음과 같다.
	- 장점
		- yaml 기반 관리로 효율 증가
		- DAG 중복코드를 방지하고 수정/삭제가 편리하다.
		- custom operator 및 plugin 을 지원한다.
		- **멋**
	- 단점
		- 현재 환경에 마이그레이션이 가능한 지 검토가 필요하다.
		- 안정화된건지.. 관심이 없는건지.. 최근 커밋이 존재하지 않는다.
		- 배보다 배꼽이 커질 수 있다. (yaml 이 커지면 커질수록 관리가 더 힘들어질 수 있다.)

# reference
- [repo](https://github.com/ajbosco/dag-factory#dag-factory)
- [dag-factory​](https://docs.astronomer.io/learn/dynamically-generating-dags#dag-factory "Direct link to dag-factory")
