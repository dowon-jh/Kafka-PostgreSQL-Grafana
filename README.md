# Kafka-PostgreSQL-Grafana

Kafka, PostgreSQL, Grafana를 사용해 실시간 웹 로그 파이프라인을 구성한 데이터 엔지니어링 실습 프로젝트입니다. Python producer가 가상 웹 로그를 생성하고, Kafka topic으로 전송한 뒤, 목적별 consumer가 PostgreSQL에 적재하거나 필터링/마스킹을 수행합니다. 최종 데이터는 Grafana에서 SQL 기반 대시보드로 확인합니다.

## 목표

```text
Python Producer
-> Kafka web_logs topic
-> Consumer 0: 전체 로그 적재
-> Consumer 1: IP 기준 중복 요청 필터링
-> Consumer 2: 민감 경로 필터링
-> Consumer 3: IP 마스킹
-> PostgreSQL web_logs table
-> Grafana dashboard
```

이 프로젝트의 핵심 목표는 단순 로그 저장이 아니라, 동일한 Kafka topic을 여러 consumer가 구독하면서 서로 다른 비즈니스 로직을 처리하는 구조를 이해하는 것입니다.

## 기술 스택

- Python 3
- kafka-python
- psycopg2-binary
- Apache Kafka KRaft Mode
- PostgreSQL 15
- Grafana OSS
- Docker Compose

## 프로젝트 구성

```text
Kafka-PostgreSQL-Grafana/
  README.md
  .env.example
  docker-compose.yml/
    README.md
  producer.py/
    README.md
  consumer0.py/
    README.md
  consumer1.py/
    README.md
  consumer2.py/
    README.md
  consumer3.py/
    README.md
```

## 실행 흐름

1. `.env.example`을 참고해 `.env` 파일을 생성합니다.
2. Docker Compose로 Kafka, PostgreSQL, Grafana를 실행합니다.
3. PostgreSQL에 `web_logs` 테이블을 생성합니다.
4. `producer.py`를 실행해 `web_logs` topic으로 로그를 전송합니다.
5. 필요한 consumer를 실행해 적재, 필터링, 마스킹 결과를 확인합니다.
6. Grafana에서 PostgreSQL datasource를 연결하고 로그 지표를 시각화합니다.

## 환경 변수 예시

```env
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_DB=logdb
POSTGRES_USER=dowon
POSTGRES_PASSWORD=change_me
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_TOPIC=web_logs
```

Python 코드에서는 DB 비밀번호를 직접 쓰지 않고 `os.getenv()`로 읽는 방식을 사용합니다.

```python
import os

conn = psycopg2.connect(
    host=os.getenv("POSTGRES_HOST", "127.0.0.1"),
    database=os.getenv("POSTGRES_DB", "logdb"),
    user=os.getenv("POSTGRES_USER", "dowon"),
    password=os.getenv("POSTGRES_PASSWORD"),
    port=os.getenv("POSTGRES_PORT", "5432"),
)
```

## 포트폴리오 핵심 포인트

- Kafka topic 하나를 여러 consumer가 목적별로 구독하는 Pub/Sub 구조를 실습했습니다.
- 전체 저장, 중복 필터링, 보안 경로 제외, 개인정보 마스킹을 독립 consumer로 분리했습니다.
- PostgreSQL INSERT는 파라미터 바인딩을 사용해 안전하게 처리합니다.
- DB 접속 정보는 `.env`로 분리해 민감정보가 코드나 문서에 직접 노출되지 않도록 개선했습니다.
- Grafana를 통해 적재된 로그를 운영 지표로 전환하는 흐름까지 연결했습니다.

## 운영 관점 개선 아이디어

- consumer별 `group_id`를 명시해 offset 관리 의도를 분명히 합니다.
- DB 저장 성공과 Kafka offset commit 시점을 함께 설계합니다.
- 실패 메시지는 DLQ topic으로 보내 재처리 가능하게 만듭니다.
- 중복 적재를 막기 위해 idempotent key 또는 unique constraint를 검토합니다.
