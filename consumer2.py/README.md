# Consumer 2: Security Path Filter

`consumer2.py`는 `/admin`, `/login` 같은 보안 민감 경로를 저장 대상에서 제외하는 consumer입니다.

## 역할

- Kafka 로그에서 `path` 값을 확인합니다.
- 민감 경로는 차단 로그만 남기고 DB 저장을 건너뜁니다.
- 일반 경로만 PostgreSQL에 저장합니다.

## 코드 예시

```python
import json
import os

import psycopg2
from kafka import KafkaConsumer


EXCLUDED_PATHS = {"/admin", "/login"}


def get_connection():
    return psycopg2.connect(
        host=os.getenv("POSTGRES_HOST", "127.0.0.1"),
        database=os.getenv("POSTGRES_DB", "logdb"),
        user=os.getenv("POSTGRES_USER", "dowon"),
        password=os.getenv("POSTGRES_PASSWORD"),
        port=os.getenv("POSTGRES_PORT", "5432"),
    )


def run_consumer():
    topic = os.getenv("KAFKA_TOPIC", "web_logs")
    bootstrap_servers = os.getenv("KAFKA_BOOTSTRAP_SERVERS", "localhost:9092")

    conn = get_connection()
    cur = conn.cursor()
    consumer = KafkaConsumer(
        topic,
        bootstrap_servers=[bootstrap_servers],
        value_deserializer=lambda message: json.loads(message.decode("utf-8")),
    )

    try:
        for message in consumer:
            log = message.value
            path = log["path"]

            if path in EXCLUDED_PATHS:
                print(f"blocked_sensitive_path={path}")
                continue

            cur.execute(
                "INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)",
                (log["ip"], path, log["status"]),
            )
            conn.commit()
            print(f"saved_public_path={path}")
    finally:
        cur.close()
        conn.close()
        consumer.close()


if __name__ == "__main__":
    run_consumer()
```

## 학습 포인트

- 저장 전에 정책을 적용하는 Pre-storage Filtering 구조입니다.
- 모든 로그를 무조건 저장하지 않고 목적에 맞는 데이터만 적재합니다.
- 운영 환경에서는 차단 로그를 별도 security topic 또는 audit table에 저장하는 방식도 고려할 수 있습니다.
