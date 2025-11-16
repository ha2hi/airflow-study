# Data Aware Scheduling (Dataset)
- Dataset : DAG란 데이터 의존성을 표현한 객체
- Data aware scheduling : Producer DAG에서 Dataset으로 지정한 파일을 업데이트되면 이를 구독하는 Consumer DAG가 자동으로 트리거 되는 기능
- Producer DAG
    - 특정 Dataset을 업데이트하면 신호(signal) 발생
    - Dataset은 업데이트가 발생했다는 사실만 전달하며, 데이터가 정확히 생성되었는지는 알 수 없음
- Consumer DAG
    - 구독한 Dataset이 업데이트되면 실행
- Producer DAG
```
from airflow import DAG
from airflow.datasets import Dataset
from airflow.operators.bash import BashOperator
from datetime import datetime

local_file = Dataset("/tmp/sample.txt")
with DAG(
    dag_id = "my_producer",
    start_date = datetime(2025,11,16),
    schedule = "0 0 * * *",
    catchup = False) as dag:

    task_producer = BashOperator(
        task_id = "task_producer",
        bash_command = 'echo "hello world" > /tmp/sample.txt',
        outlets = [local_file]
    )
```  
- Consumer DAG
```
from airflow import DAG
from airflow.datasets import Dataset
from airflow.operators.bash import BashOperator
from datetime import datetime

local_file = Dataset("/tmp/sample.txt",extra={"team":"test"})
local_file2 = Dataset("/tmp/sample2.txt")
with DAG(
    dag_id = "my_consumer",
    schedule = (local_file | local_file2),
    start_date = datetime(2025,11,16),
    catchup=False) as dag:

    task_consumer = BashOperator(
        task_id="consumer1",
        bash_command='cat /tmp/sample.txt')
```
- schedule에서 `|`로 지정하는 경우 OR, `&`로 지정하는 경우 AND다.  
- extra는 추가적인 정보일뿐 실행에 영향을 미치지 않는다.

# 참고
- https://airflow.apache.org/docs/apache-airflow/2.11.0/authoring-and-scheduling/datasets.html