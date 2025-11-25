# Trigger_rule
- 업스트림 태스크들의 상태에 따라 현재 태스크가 실행될지 말지를 결정하는 조건
- all_success(default) : 업스트림의 모든 태스크가 성공해야 실행
- all_failed : 업스트림의 모든 태스크가 실패해야 실행
- all_done : 업스트림의 모든 태스크가 종료만 되면 실행(상태 상관x)
- one_failed : 업스트림 중 하나 이상이라도 failed 상태가 있을 경우 실행
- one_success:업스트림 중 하나 이상이라도 success 상태가 있을 경우 실행
- non_failed : 업스트림 중 failed 상태가 하나도 없어야 실행(skip 허용)
```
task = PythonOperator(
    task_id = "task",
    trriger_rule = "one_success"
)
```