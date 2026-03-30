# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(4)

## 🚫 Anti-Spam Consumer (`consumer1.py`)

이 컨슈머는 동일한 IP에서 발생하는 단기 반복 요청(스팸성 로그)을 필터링하여 데이터베이스의 품질을 유지하는 **보안관** 역할을 합니다.

### 🔍 핵심 로직 분석

```diff
# 5초 이내 동일 IP 재접속 차단 로직
+ if ip in last_access and current_time - last_access[ip] < 5:
+     print(f"⏩ SKIP: Duplicate request from {ip}")
+     continue
```

```
import json
import time
import psycopg2
from kafka import KafkaConsumer

def run_consumer():
    # DB 및 Kafka 연결 설정
    conn = psycopg2.connect(host="127.0.0.1", database="logdb", user="dowon", password="1234", port="5432")
    cur = conn.cursor()
    consumer = KafkaConsumer('web_logs', bootstrap_servers=['localhost:9092'], value_deserializer=lambda m: json.loads(m.decode('utf-8')))

    # 인메모리 캐시 (IP별 마지막 접속 시각 저장)
    last_access = {}

    print("🚫 Consumer 1: Anti-Spam Mode (5s cool down)...")

    for message in consumer:
        log = message.value
        ip = log['ip']
        current_time = time.time()

        # [핵심] 5초 쿨타임 검증
        if ip in last_access and current_time - last_access[ip] < 5:
            print(f"⏩ SKIP: Duplicate request from {ip} within 5 seconds.")
            continue
        
        # 접속 정보 갱신 및 DB 적재
        last_access[ip] = current_time
        cur.execute("INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)", (ip, log['path'], log['status']))
        conn.commit()
        print(f"✅ SAVED: Unique access from {ip}")

if __name__ == "__main__":
    run_consumer()

```
### 📈 작동 프로세스
- **메시지 수신**: Kafka로부터 실시간 로그 데이터를 수신합니다.

- **메모리 대조**: last_access 딕셔너리에 해당 IP가 있는지, 있다면 시간이 5초 경과했는지 확인합니다.

- **조건부 실행**:

- **5초 미만**: 저장 로직을 SKIP 합니다.

- **5초 이상/첫 접속**: 접속 시각을 갱신하고 PostgreSQL에 INSERT 합니다.

#### 💡 학습 노트: 이 방식은 Stateful(상태 유지) 컨슈머의 기초입니다. 단순히 데이터를 받는 것을 넘어, 과거의 정보를 기억하고 현재의 동작을 결정하는 로직을 배울 수 있습니다.
