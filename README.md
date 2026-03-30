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
```
----------
## 📥 Basic Consumer (`consumer0.py`)

이 스크립트는 Kafka의 `web_logs` 토픽을 구독하여 실시간 데이터를 수신하고, 이를 PostgreSQL 데이터베이스에 저장하는 가장 기본적인 **데이터 수집기**입니다.

### 🛠️ 핵심 기능
- **Kafka Subscription**: `web_logs` 토픽에 발행되는 메시지를 실시간으로 모니터링합니다.
- **Data Deserialization**: JSON 바이트 데이터를 Python 딕셔너리 객체로 변환하여 처리합니다.
- **RDBMS Integration**: `psycopg2` 라이브러리를 사용하여 구조화된 로그 데이터를 PostgreSQL 테이블에 영구적으로 보존합니다.

### 📜 주요 로직 요약
- **Offset 설정**: `auto_offset_reset='earliest'` 옵션을 통해 컨슈머가 처음 구동될 때 과거에 쌓여있던 로그부터 유실 없이 처리하도록 보장합니다.
- **Transaction 관리**: 각 로그 메시지가 삽입될 때마다 `commit()`을 수행하여 데이터의 원자성(Atomicity)을 유지합니다.

### 🚀 실행 방법
```bash
python consumer0.py
