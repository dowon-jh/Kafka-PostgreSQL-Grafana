# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(3)

## 📄 Consumer 상세 분석: consumer0.py
## : 이 프로세스는 Kafka로부터 유입되는 원시 데이터(Raw Data)를 PostgreSQL 데이터베이스로 안전하게 이적시키는 데이터 파이프라인의 종착점 역할을 수행합니다.
```
import json
import psycopg2
from kafka import KafkaConsumer

def run_consumer():
    # [1] 데이터베이스 연결 설정
    conn = psycopg2.connect(
        host="127.0.0.1",
        database="logdb",
        user="dowon",
        password="1234",
        port="5432"
    )
    cur = conn.cursor()

    # [2] Kafka Consumer 환경 설정
    consumer = KafkaConsumer(
        'web_logs',
        bootstrap_servers=['localhost:9092'],
        auto_offset_reset='earliest', # 핵심: 과거 데이터부터 유실 없이 수집
        value_deserializer=lambda m: json.loads(m.decode('utf-8')) # 역직렬화
    )

    print("📥 Consumer started! Waiting for logs...")

    try:
        for message in consumer:
            log = message.value
            print(f"Received from Kafka: {log}")

            # [3] 데이터 적재 (INSERT)
            cur.execute(
                "INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)",
                (log['ip'], log['path'], log['status'])
            )
            # [4] 트랜잭션 확정
            conn.commit() 
            print("✅ Successfully saved to DB!")
            
    except KeyboardInterrupt:
        print("Consumer stopped.")
    finally:
        # [5] 자원 반납
        cur.close()
        conn.close()


if __name__ == "__main__":
    run_consumer()
```

### 🛠️ 데이터 흐름도 (Data Flow)
- **Subscribe**: web_logs 토픽에 새로운 이벤트가 발생하는지 감시합니다.

- **Poll**: 새로운 로그 데이터(바이트)를 가져옵니다.

- **Transform**: 역직렬화(json.loads)를 통해 데이터 구조를 복원합니다.

- **Load**: INSERT 쿼리를 통해 관계형 데이터베이스(RDBMS)에 행(Row) 단위로 적재합니다.

### 📜 핵심 학습 포인트
- **안정성**: try-finally 구조를 사용하여 비정상 종료 시에도 DB 자원을 안전하게 보호합니다.

- **보안**: 쿼리 작성 시 변수를 직접 결합하지 않고 psycopg2의 파라미터 바인딩(%s)을 사용하여 보안성을 확보했습니다.

- **정확성**: commit()의 위치를 루프 내부로 설정하여 각 메시지 단위로 저장을 확정 짓습니다.
