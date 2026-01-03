### Airflow 메트릭 시각화 및 모니터링
Airflow는 태스크를 대기열에 저장한 후 실제 실행까지 걸리는 시간,
태스크 실행 상태 및 시스템 부하 등을 메트릭으로 수집할 수 있다.
이를 시각화하고 추적하기 위한 모니터링 환경 구성이 필요하다.

Airflow는 StatsD 호환 클라이언트를 통해 메트릭을 Push 방식으로 전송한다.
Prometheus를 사용하는 경우, StatsD 메트릭을 Prometheus 포맷으로 변환해주는
statsd-exporter와 같은 중계 서비스가 필요하다.
이후 Grafana를 통해 실시간 모니터링 환경을 구성할 수 있다.

  
### 모니터링 대상
1. Latency
서비스 요청에 따른 소요 시간 확인이 필요
예를 들어 웹서버 응답 시간, 태스크 대기 상태 -> 실행 상태 시간  

2. Traffic
시스템에 얼마나 많은 요청이 오는지 확인 필요
예를 들어 처리해야되는 태스크 수, 사용 가능한 오픈 풀 슬롯  

3. Errors
어떤 오류가 발생했는지 확인 필요
예를 들어 좀비 프로세스 수(기본 프로세스가 사라진 작업 실행), 웹서버 상태코드가 200이 아닌 응답 수, 시간 초과된 태스크 수  

4. Stturation
자원 사용량 확인 필요
- Airflow의 CPU/Memory/Disk/Network IO 사용량
- 주요 측정 항목
    - DAG 정상동작 여부
        - dag_processing.import_errors : DAG 처리 중 발생한 오류 수
        - dag_processing.total_parse_time : 총 Parse 시간, DAG 추가 및 변경 후 크게 상승하면 확인 필요
        - ti_failures : 실패한 태스크 인스턴스의 수
    - Airflow 성능 상태 확인
        - dag_processing.last_duration.[filename] : DAG 파일을 처리하는데 걸리는 시간
        - executor.open_slots : 사용 가능한 익스큐터 슬롯의 수
        - executor.queued_tasks : 대기 상태의 태스크 수
        - executor.running_tasks : 실행 상태의 태스크 수  
  
### 참고
- https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/metrics.html