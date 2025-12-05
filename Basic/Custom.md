### Custom Hook
- API 연동 후 작업 로직이 있는 경우 캡슐화하여 재활용 가능한 Hook을 생성하여 사용할 수 있음
- 모든 Hook은 추상 클래스인 BaseHook 클래스의 서브클래스로 생성함
```
from airflow.hooks.base import BaseHook

class MovielensHook(BaseHook):
    def __init__(self, conn_id, retry = 3):
        super().__init__()
        self.conn_id = conn_id

    def get_conn(self):
        # Connection의 정보를 갖고옴
        config = self.get_connection(self.conn_id)
        schema = config.schema or self.DEFAULT_SCHEMA
        port = config.port or self.DEFAULT_PORT

        self._base_url = f"{schema}://{config.host}:{port}"
```

### Custom Operator
- Custom Operator를 사용하여 반복적인 태스크 수행 시 코드 중복을 최소화할 수 있음
- template_fields : {{ ds }}와 같은 Jinja 템플릿 변수를 실제 날짜로 바꾸려면 반드시 필요합니다.
- dags/custom/operators.py
```
import json
import os

from airflow.models import BaseOperator
from custom.hooks import MovielensHook

class MovielensFetchRatingsOperator(BaseOperator):
    template_fields = ("start_date", "_end_date", "output_date")

    def __init__(
            self,
            conn_id,
            output_path,
            start_date="{{ ds }}",
            end_date="{{ next_ds }}",
            batch_size = 1000,
            **kwargs
    ):
        super().__init__(**kwargs)

        self._conn_id = conn_id
        self.output_path = output_path
        self.start_date = start_date
        self.end_date = end_date
        self.batch_size = batch_size

    # 실제 실행할 함수 입력
    def execute(self, context):
        hook = MovielensHook(self._conn_id)

        try:
            self.log.info(
                f"Fetching ratings for {self._start_date} to {self._end_date}"
            )
            ratings = list(
                hook.get_ratings(
                    start_date=self._start_date,
                    end_date=self._end_date,
                    batch_size=self._batch_size,
                )
            )
            self.log.info(f"Fetched {len(ratings)} ratings")
        finally:
            # Make sure we always close our hook's session.
            hook.close()

        self.log.info(f"Writing ratings to {self._output_path}")

        # Make sure output directory exists.
        output_dir = os.path.dirname(self._output_path)
        os.makedirs(output_dir, exist_ok=True)

        # Write output as JSON.
        with open(self._output_path, "w") as file_:
            json.dump(ratings, fp=file_)
```
- operator.py
```
MovielensFetchRatingsOperator(
        task_id="fetch_ratings",
        conn_id="movielens",
        start_date="{{ds}}",
        end_date="{{next_ds}}",
        output_path="/data/custom_operator/{{ds}}.json",
    )
```

### Custom Sensor Operator
- 다운 스트림 태스크가 실행하기 전 조건을 확인하기 위해 대기하는 작업
- 상태가 False인 경우 대기, True인 경우 다운 스트림 태스크 실행
- custom/sensors.py
```
from airflow.sensors.base import BaseSensorOperator
from custom.hooks import MovielensHook

class MovielensRatingsSensor(BaseSensorOperator):
    template_fields = ("start_date", "end_date")

    def __init__(
        self, 
        conn_id, 
        start_date="{{ds}}", 
        end_date="{{next_ds}}", 
        **kwargs):
        super().__init__(**kwargs)
        
        self.conn_id = conn_id
        self.start_date = start_date
        self.end_date = end_date

    def poke(self, context):
        hook = MovielensHook(self.conn_id)

        try:
            next(
                hook.get_ratings(
                    start_date=self.start_date, end_date=self.end_date, batch_size=1
                )
            )
            self.log.info(
                f"Found ratings for {self.start_date} to {self.end_date}, continuing!"
            )
            return True
        except StopIteration:
            self.log.info(
                f"Didn't find any ratings for {self.start_date} "
                f"to {self.end_date}, waiting..."
            )
            return False
        finally:
            hook.close()
```

