# Branch Operator
- 다운 스트림 2개 이상의 task 중 어떤 task를 실행할지 전략을 취할지정함.
- TASK 분기를 위한 Operator
```
from airflow.operators.dummy_operator import DummyOperator
from airflow.operators.python_operator import BranchPythonOperator

def decide_which_path():
    if int(time.time()) % 2 == 0:
        return "even_path_task"
    else:
        return "ood_path_task"

branch_task = BranchPythonOperator(
    task_id = "branch_task",
    python_callable = decide_which_path
)

    even_path_task = DummpyOperator(task_id = "even_path_task")
    odd_path_task = DummyOperator(task_id = "odd_path_task)

branch_task >> even_path_task
branch_task >> odd_path_task
```  