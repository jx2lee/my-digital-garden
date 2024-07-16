---
{"dg-publish":true,"permalink":"/data/airflow/__/airflow-task/","noteIcon":"","created":"2024-06-30T00:39:32.590+09:00"}
---



DAG 개발하던 중 용어가 헷갈려 간단히 정리한다.

**tl;dr**
- task 는 DAG 으로 정의한 하나의 작업 단위이다. task instance 는 작업 단위인 task 를 실행할 때 생성하는 인스턴스이다. task 로 정의해놓은 작업을 실행할 때 task instance 가 생성하면서 작업을 수행한다.

# Task

> *A Task is the basic unit of execution in Airflow. Tasks are arranged into [DAGs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html), and then have upstream and downstream dependencies set between them into order to express the order they should run in.*

`task` 는 DAG(Directed Acyclic Graph)에 정의된 하나의 작업 단위를 나타내며, 작업을 구성하는 코드와 메타데이터를 포함한다.

# Task Instance

> *An instance of a Task is a specific run of that task for a given DAG (and thus for a given data interval). They are also the representation of a Task that has _state_, representing what stage of the lifecycle it is in.*

`task instance` 는 "task"를 실제로 실행할 때 생성되는 인스턴스를 의미한다. 즉, `task instance` 는 **`task` 의 실행 가능한 버전**으로, 실행할 때마다 새로운 인스턴스를 생성한다.

예를 들어, DAG 에 **task_A**와 **task_B** 가 있을 때, 각각의 "task"는 DAG의 노드를 나타내며, 실행되기 전까지는 대기 상태를 갖는다. `task instance` 는 각각의 "task"를 실행할 때 생성되는 실제 실행 단위로, DAG 을 여러번 실행하면 다수의 인스턴스가 생성될 수 있다. 각 `task instance` 는 고유한 인스턴스 ID를 가지며 DAG 실행 시간, 실행 결과 및 상태 등과 같은 메타데이터를 갖는다.

![](https://i.imgur.com/Urfh4r7.png)
*[참고](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html#task-instances): airflow lifecycle*

# Job
Airflow 에서 Job 은 일정에 따라 실행되는 작업의 모음이며, 작업 내 단위를 나타낸다.

Airflow의 각 작업은 스케줄 정보를 가진 작업의 모음인 방향성 비순환 그래프(DAG)로 표시된다. DAG는 작업 종속성 또는 관계와 함께 작업이 실행되어야 하는 순서를 지정한다. 중요한 점은 DAG는 작업이 무엇을 하는지에 관심을 두는 것이 아니라, 어**떤 작업을 수행하든 그 작업이 적시에 올바른 순서로 수행되도록 하는 것이 DAG의 역할**이다.

# references
- [Tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html)
- [Tasks Instance](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html#task-instances)