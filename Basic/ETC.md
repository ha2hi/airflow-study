# LISTING TASK IN THE DAG
```
task1 = DummyOperator(task_id="task1")
task2 = DummyOperator(task_id="task2")
task3 = DummyOperator(task_id="task3")
task4 = DummyOperator(task_id="task4")

# normal way
task1 >> task2
task1 >> task3
task2 >> task3
task3 >> taskt

# better way
task1 >> [task2, task3] > task4
```  

### PARAMS ARGUMENT
```
params = {
    "p1" : "v1",
    "p2" : "v2"
}
with DAG("params_args", params = params):
    task1 = BashOperator(
        task_id = "task1",
        bash_command = 'echo {{ pharams.p1}}' 
    )

    task2 = BashOperator(
        task_id = "task2",
        bash_command = 'echo {{ pharams.p3}}',
        params = {
            "p3" : "v3"
        } 
    )
```  

### Connection INFO
```
from airflow.hook.base_hook import BaseHook

def _access_connection(ti):
    print(BaseHook.get_connection("my_postgres_conn").host)
    print(BaseHook.get_connection("my_postgres_conn").password)
```  

### Access Variable with Dict
```
from airflow.models import Variable

Variable.get("var1")
Variable.get("var_dict", deserialize_json=True)
```