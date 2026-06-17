# Consumer 3: Privacy Masking

`consumer3.py`는 로그의 IP 주소 마지막 구간을 마스킹한 뒤 PostgreSQL에 저장하는 개인정보 보호 consumer입니다.

## 역할

```text
192.168.1.123
-> 192.168.1.***
```

원본 IP를 그대로 저장하지 않고, 분석에 필요한 대역 정보만 남기는 방식입니다.

## 코드 예시

```python
import json
import os

import psycopg2
from kafka import KafkaConsumer


def mask_ip(ip):
    parts = ip.split(".")
    if len(parts) != 4:
        return ip
    parts[-1] = "***"
    return ".".join(parts)


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
            masked_ip = mask_ip(log["ip"])

            cur.execute(
                "INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)",
                (masked_ip, log["path"], log["status"]),
            )
            conn.commit()
            print(f"masked_ip={log['ip']} saved_ip={masked_ip}")
    finally:
        cur.close()
        conn.close()
        consumer.close()


if __name__ == "__main__":
    run_consumer()
```

## 학습 포인트

- Kafka에서 받은 데이터를 DB 저장 전에 변환하는 ETL의 Transform 단계를 분리했습니다.
- IP 전체를 저장하지 않아 개인정보 노출 위험을 줄입니다.
- 통계 분석에 필요한 IP 대역 정보는 유지할 수 있습니다.
- 운영 환경에서는 IP 형식 검증, IPv6 처리, 마스킹 정책 테스트를 추가해야 합니다.
