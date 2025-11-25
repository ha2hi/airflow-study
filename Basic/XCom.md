# XCOM
- 태스크간의 작은 데이터를 공유하기 위한 기능
- Meta 데이터베이스 안에 Push를 통해 값을 저장하고, Pull을 통해 읽을 수 있음.
- 데이터프레임과 같이 큰 값을 저장할 때 사용 X
- XCom 백엔드는 DB 뿐만 아니라 S3도 사용 가능
```
def generate_number(ti):
    number = random.randint(0, 100)
    ti.xcom_push(
        key = "random_number",
        value=number
    )

def read_number(ti):
    number = ti.xcom_pull(
        key = "random_number",
        task_ids = "generate_number_task"
    )
    print(number)

generate_number_task = PythonOperator(
    task_id = "generate_number_task",
    python_callable = generate_number,
    provider_context = True
)

read_number_task = PythonOperator(
    task_id = "read_number_task",
    python_callable = read_number,
    provider_context = True
)
```  
  
- 여러 XCom 사용
```
@task(do_xcom_push=True, multiple_outputs=True)
def push_multiple(**context):
    return {"key1" : "value1", "key2" : "value2"}

@task
def xcom_pull_with_multiple_outputs(**context):
    key1 = context[ti].xcom_pull(task_ids = "push_multiple", key= "key1")
    key2 = context[ti].xcom_pull(task_ids = "push_multiple", key= "key2")

    data = context[ti].xcom_pull(task_ids = "push_multiple", key = "return_value")
```