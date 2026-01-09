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

# XCom Backend
XCom은 메타 데이터베이스에 값을 저장하고 가져오는 방식이기 때문에 데이터를 저장하는데 한계가 있다.  
이 때 메타데이터베이스로 사용하는 DBMS 종류에 따라 저장 가능한 사이즈가 다르다.
- SQLite : 2GB
- PostgreSQL : 1GB
- MySQL : 64KB
따라서 크기가 큰 데이터를 저장하기 위해서는 DB에 저장할 수 없다.  
이 때 Cloud Storage를 활용해 이러한 문제를 해결할 수 있다.  
- airflow.cfg
```
[core]
# 백엔드 클래스 지정
xcom_backend = airflow.providers.common.io.xcom.backend.XComObjectStorageBackend

[common.io]
#S3 저장 경로 (미리 생성한 S3 커넥션 필요)
xcom_objectstorage_path = s3://my_aws_conn@<my-bucket>/xcom

# 크기 기준 설정 (단위: 바이트)
# 이 예시는 1MB(1048576) 이상일 때만 S3로 보내고, 이하면 DB에 저장합니다.
xcom_objectstorage_threshold = 1048576

# 압축 여부 (선택 사항)
xcom_objectstorage_compression = gzip
```  
  
위 내용을 반영하기 위해 웹서버, 스케줄러 재실행
```
restart webserver
resrart scheduler
```
