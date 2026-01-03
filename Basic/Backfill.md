### Backfill
- 과거의 날짜에 대해 실행하는 작업
- 재처리 제어
    - none : 해당 날짜(Logical Date)에 실행 기록이 있으면, 상태와 상관없이 새로 생성 x
    - failed : 실행 기록이 있더라도 상태가 failed인 경우 재실행
    - completed : 무조건 재실행
- 동시성 제어
    - 동시에 많은 재실행하는 경우 시스템 과부하가 발생할 수 있음
    - 따라서 `max_active_runs`을 통해 동시에 실행되는 DAG Run의 개수 제한 가능
- 실행 순서
    - 과거 -> 현재(default)로 실행됨.
    - `--run-backwards` 옵션을 통해 현재 -> 과거로 역순 실행 가능
- CLI
```
airflow backill create --dag-id <dag_id> \
    -s <start_date> \
    -e <end_date> \ 
    --reprocessing-behavior failed \
    --max-active-runs 3 \
    --run-backwards
```
- --mark-success : 실제로 실행x, mark만 표시
- --rerun-failed-tasks : 실패한 작업만 실행
- --reset-dagruns : 모두 재실행
```  

### REF
- https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/backfill.html
