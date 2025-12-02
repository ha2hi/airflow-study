### Custom Hook
- API 연동 후 작업 로직이 있는 경우 캡슐화하여 재활용 가능한 Hook을 생성하여 사용할 수 있음
- 모든 Hook은 추상 클래스인 BaseHook 클래스의 서브클래스로 생성함
```
from airflow.hooks.base_hook import BaseHook

class MovielensHook(BaseHook):
    def __init__(self, conn_id, retry = 3):
        super().__init__()
```
