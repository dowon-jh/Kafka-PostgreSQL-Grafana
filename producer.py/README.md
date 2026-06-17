# Producer

`producer.py`는 웹 서버에서 발생하는 로그를 시뮬레이션해 Kafka `web_logs` topic으로 전송하는 데이터 생성기입니다.

## 역할

- IP, 요청 경로, HTTP 상태 코드를 가진 로그 데이터를 생성합니다.
- Python dict를 JSON byte stream으로 직렬화합니다.
- Kafka broker로 1초마다 메시지를 전송합니다.

## 코드 예시

```python
import json
import os
import random
import time

from kafka import KafkaProducer


def run_producer():
    bootstrap_servers = os.getenv("KAFKA_BOOTSTRAP_SERVERS", "localhost:9092")
    topic = os.getenv("KAFKA_TOPIC", "web_logs")

    producer = KafkaProducer(
        bootstrap_servers=[bootstrap_servers],
        value_serializer=lambda value: json.dumps(value).encode("utf-8"),
    )

    paths = ["/home", "/login", "/product/123", "/cart", "/order", "/admin"]
    status_codes = [200, 201, 404, 500]

    try:
        while True:
            log_data = {
                "ip": f"192.168.1.{random.randint(1, 255)}",
                "path": random.choice(paths),
                "status": random.choice(status_codes),
            }

            producer.send(topic, log_data)
            print(f"sent={log_data}")
            time.sleep(1)
    finally:
        producer.close()


if __name__ == "__main__":
    run_producer()
```

## 실행

```bash
python producer.py
```

## 학습 포인트

- Kafka producer는 메시지를 byte 단위로 전송하므로 JSON 직렬화가 필요합니다.
- topic 이름과 broker 주소는 환경 변수로 분리해 실행 환경별 변경을 쉽게 합니다.
- 운영 환경에서는 `flush()`, 전송 실패 처리, 재시도 정책, key 기반 partitioning 전략을 추가로 고려해야 합니다.
