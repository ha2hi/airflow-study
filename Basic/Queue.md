Airflow Queue는 Celery와 같은 분산 Executor를 사용할 때 스케줄러가 실행할 작업을 임시로 모아두는 가상 대기열이다.  
여러 워커들은 각자의 큐에 있는 작업을 가져가서 심행함으로써 작업 부하를 분산 시킬 수 있다.  
큐는 TASK를 관리한다. 즉, 우선 순위가 높은 task가 먼저 실행되도록 관리한다.  
  
테스크별로 큐를 지정할 수 있음.
- worker
```
airflw-worker2:
  <<: *airflow_common
  command: celery worker -q cpu_intensive
```
- DAG
```
task1 = BashOperator(
    task_id = "task1",
    queue = "cpu_intensive",
    bash_command = "sleep 20"
)