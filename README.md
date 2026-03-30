## 📤 Log Producer (`producer.py`)

이 모듈은 웹 서버에서 발생하는 실시간 로그 데이터를 시뮬레이션하여 Kafka 브로커로 전송하는 역할을 합니다.

### 📜 핵심 코드 미리보기

```python
# Kafka 연결 및 직렬화 설정
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# 실시간 로그 전송 루프
while True:
    log_data = {
        "ip": f"192.168.1.{random.randint(1, 255)}",
        "path": random.choice(paths),
        "status": random.choice(status_codes)
    }
    producer.send('web_logs', log_data)
    time.sleep(1)
