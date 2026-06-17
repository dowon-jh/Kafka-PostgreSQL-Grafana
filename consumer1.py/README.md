# Consumer 1: Anti-Spam Filter

`consumer1.py`는 동일 IP에서 짧은 시간 안에 반복되는 요청을 필터링해 저장 데이터의 품질을 높이는 consumer입니다.

## 역할

- Kafka `web_logs` topic을 구독합니다.
- IP별 마지막 저장 시각을 메모리에 보관합니다.
- 같은 IP가 5초 이내 다시 들어오면 PostgreSQL 저장을 건너뜁니다.

## 코드 예시

```python
import json
import os
import time

import psycopg2
from kafka import KafkaConsumer


COOLDOWN_SECONDS = 5


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
    last_access_by_ip = {}

    try:
        for message in consumer:
            log = message.value
            ip = log["ip"]
            now = time.time()

            if ip in last_access_by_ip and now - last_access_by_ip[ip] < COOLDOWN_SECONDS:
                print(f"skipped_duplicate_ip={ip}")
                continue

            last_access_by_ip[ip] = now
            cur.execute(
                "INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)",
                (ip, log["path"], log["status"]),
            )
            conn.commit()
            print(f"saved_unique_ip={ip}")
    finally:
        cur.close()
        conn.close()
        consumer.close()


if __name__ == "__main__":
    run_consumer()
```

## 학습 포인트

- 메모리 딕셔너리를 사용해 상태를 유지하는 Stateful Consumer의 기본 구조를 실습했습니다.
- 짧은 시간 내 중복 요청을 줄여 저장 공간과 분석 품질을 개선합니다.
- 여러 프로세스로 확장하면 메모리 상태가 분산되므로 Redis 같은 외부 상태 저장소를 고려해야 합니다.
