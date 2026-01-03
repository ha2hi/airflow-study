### 1. 개요
데이터 파이프라인의 안정성을 보장하기 위해 두 가지 단계의 테스트가 필수적입니다.
- 단위 테스트 (Unit Test): 하나의 태스크나 오퍼레이터가 의도대로 동작하는지 확인합니다.
- 통합 테스트 (Integration Test): 전체 DAG 및 태스크 간의 흐름이 올바른지 검증합니다.
- 도구: Python의 표준 테스트 프레임워크인 pytest를 사용합니다.

### 2. 무결성 테스트
DAG가 Airflow 스케줄러에서 에러 없이 로드될 수 있는지 검증하는 과정입니다.
- 주요 검증 항목:
    - DAG 내에 **순환(Cycle)**이 포함되어 있지 않은가?
    - 태스크 ID(task_id)가 DAG 내에서 중복되지 않고 고유한가?
    - 필수 파라미터가 누락되지 않았는가?

### 2.1 순환 오류 예시
DAG(Directed Acyclic Graph)는 단방향 비순환 그래프여야 합니다. 아래와 같이 작성할 경우 순환이 발생하여 오류가 납니다.
```
t1 = DummyOperator(task_id = "t1)
t2 = DummyOperator(task_id = "t2)
t3 = DummyOperator(task_id = "t3)

t1 >> t2 >> t3 >> t1
```
### 3. 테스트 환경 구성
### 3.1 pytest 설치
```
pip install pytest
```

### 3.2 디렉토리 구조
`tests/` 디렉터리를 프로젝트 루트에 생성하고, 테스트 파일명은 test_로 시작해야 pytest가 자동으로 인식합니다.
![test directory](../images/airflow_test_tree.png.png)

### 4. 무결성 테스트 코드
- test_dag_integrity.py
아래 코드는 dags 폴더에 .py파일을 모두 탐색하여 순환 참조가 있는지 확인한다.
```
import glob
import importlib.util
import os

import pytest
from airflow.models import DAG

DAG_PATH = os.path.join(os.path.dirname(__file__), "..", "..", "dags/**.py")
DAG_FILES = glob.glob(DAG_PATH)


@pytest.mark.parametrize("dag_file", DAG_FILES)
def test_dag_integrity(dag_file):
    """Import DAG files and check for DAG."""
    module_name, _ = os.path.splitext(dag_file)
    module_path = os.path.join(DAG_PATH, dag_file)
    mod_spec = importlib.util.spec_from_file_location(module_name, module_path)
    module = importlib.util.module_from_spec(mod_spec)
    mod_spec.loader.exec_module(module)

    dag_objects = [var for var in vars(module).values() if isinstance(var, DAG)]

    # DAG 객체가 하나 이상 존재하는지 확인 (선택 사항)
    assert dag_objects

    for dag in dag_objects:
        # 순환 참조 테스트
        dag.test_cycle()

        # 명명 규칙 테스트 예시
        # 예 : 모든 DAG ID는 'import' 혹은 'export'로 시작해야 한다면?
        # assert dag.dag_id.startswith(("import", "export"))
```

### 5. 테스트 실행
```
pytest test/
```
