---
{"dg-publish":true,"permalink":"/data/airflow/__/airflow-scheduler-process/","tags":["airflow","scheduler"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgShowInlineTitle":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":"","created":"2024-06-30T00:39:32.590+09:00"}
---


> [!info] Airflow 스케쥴러가 동작하는 과정을 코드레벨에서 살펴본다.

### background


- 스케줄러를 만들어보고 싶은 욕심이 있다. 
- 근데 어떤 방식으로 구현되는지, 구현해보지 않았다. 그래서 주변에 사용하는 도구들을 살펴봤을 때, Airflow 가 눈에 띄였다.
- Airflow 는 데이터 엔지니어라면 한 번이라도 사용해보았을 정도로 많이 사용하는 도구이다. Airflow 를 뜯어보는 것으로 시작해보자!
- 뜯어보되,
    - schduler 가 어떻게 동작하는지 파악하기 위해 local docker-compose 로 AIrflow 를 구동해본다.
    - 코드레벨 결국 CLI(`airflow scheduler`) 로 동작하는 스케줄러가 어떻게 실행되고 어떤 방식으로 DAG 을 실행하도록 실행계획을 세우는지 살펴본다.


### Airflow with docker-compose

[링크](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html) 참고해서 준비한다.

- 간단한 스크립트를 준비했다.
- version 명을 인자로 받아 Airflow 해당 버전의 compose 파일을 다운받고 이를 실행(docker compose up) 한다.


### Airflow Scheduler


- 공식문서에는 다음과 같이 설명한다.
    - `스케줄러는 모든 task 와 DAG를 모니터링한 다음, 종속성들이 완료되면 task instance 를 트리거합니다.`
    - ...


### Airflow 스케쥴러가 동작하는 방식


- [scheduler](https://github.com/apache/airflow/blob/main/airflow/cli/commands/scheduler_command.py)
    - 커맨드 관련 모듈은 airflow.cli.commands 안에 존재한다. 이 중 스케줄러는 scheduler_command 안에 포함한다.
    - `airflow scheduler` 커맨드 실행시 airflow.cli.commands.scheduler_command.scheduler 가 실행된다.
    - `scheduler` 함수는 전달받은 argument 와 함께 `run_command_with_daemon_option` 을 실행한다.
- [run_command_with_daemon_option](https://github.com/apache/airflow/blob/14a613fc7dd148b9721e011ec629cb373d0d3c2e/airflow/cli/commands/daemon_utils.py#L31)
    - 데몬 프로세스를 실행할 수 있는 helper 함수이다.
    - `python-daemon` 패키지를 이용해 프로세스를 생성한다. Context 에 사용할 파라미터를 넣어주고, Airflow schduler 프로세스를 실행한다.
    - python-daemon 에 대한 내용은 아래 레퍼런스를 참고해보자.
- 
- [PEP-3143](https://peps.python.org/pep-3143)

references
- [python으로 daemon 만들기](https://oddpoet.net/blog/2013/09/24/python-daemon/)


#### 

