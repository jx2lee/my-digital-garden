---
{"dg-publish":true,"permalink":"/data/airflow/__/airflow-landing-time/","noteIcon":"","created":"2024-06-30T00:39:32.590+09:00"}
---


# definition
- [airflow document](https://airflow.apache.org/docs/apache-airflow/stable/ui.html#landing-times)
	- landing time 은 작업의 예정된 시간부터 성공 또는 다른 상태로 완료되는 시간을 의미한다.
	- ![](https://airflow.apache.org/docs/apache-airflow/stable/_images/task_lifecycle_diagram.png)
	- task instance 의 state 를 나타내는 그래프에서, success 시간과 scheduler 시간의 차이를 landing time 으로 볼 수 있다.
	- 코드를 뜯어보면 다음과 같다.
		- [airflow/www/views.py](https://github.com/apache/airflow/blob/e89a7eeea6a7a5a5a30a3f3cf86dfabf7c343412/airflow/www/views.py#L3250)
		- `get_task_instances_before` 값으로 스케쥴된 시간과 `get_run_data_interval` 로 성공/실패로 끝난 두 시점의 total_seconds 를 **y 값으로 설정**하고 있다.
			```python
			tis = dag.get_task_instances_before(base_date, num_runs, session=session)
			...
			
			for task in dag.tasks:
				task_id = task.task_id
				for ti in tis:
					if ti.task_id != task.task_id:
						continue
					ts = dag.get_run_data_interval(ti.dag_run).end
					if ti.end_date:
						dttm = wwwutils.epoch(ti.execution_date)
						secs = (ti.end_date - ts).total_seconds()
						x_points[task_id].append(dttm)
						y_points[task_id].append(secs)
			```
- [astronomer document](https://docs.astronomer.io/learn/airflow-ui)
	- 시간이 지남에 따라 각 작업이 시작된 시간을 선 그래프로 표시한다.

# possibility for metric
