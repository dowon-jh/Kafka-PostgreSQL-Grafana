# Consumer 0: Basic Ingest

`consumer0.py`는 Kafka `web_logs` topic의 모든 로그를 PostgreSQL `web_logs` 테이블에 저장하는 기본 적재 consumer입니다.

## 역할

```text
Kafka message 수신
-> JSON 역직렬화
-> PostgreSQL INSERT
-> transaction commit
```

## 코드 예시

```python
import json
import os

import psycopg2
from kafka import KafkaConsumer


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
        auto_offset_reset="earliest",
        value_deserializer=lambda message: json.loads(message.decode("utf-8")),
    )

    try:
        for message in consumer:
            log = message.value
            cur.execute(
                "INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)",
                (log["ip"], log["path"], log["status"]),
            )
            conn.commit()
            print(f"saved={log}")
    finally:
        cur.close()
        conn.close()
        consumer.close()


if __name__ == "__main__":
    run_consumer()
```

## 실행

```bash
python consumer0.py
```

## 학습 포인트

- DB 접속 정보는 `.env`에서 읽고, 비밀번호를 코드에 직접 쓰지 않습니다.
- INSERT에는 문자열 결합이 아니라 파라미터 바인딩을 사용합니다.
- 메시지 단위 commit은 이해하기 쉽지만 대량 처리에서는 batch commit을 고려해야 합니다.
- 운영 환경에서는 DB commit과 Kafka offset commit의 순서를 명확히 설계해야 합니다.
