
📄 Consumer 상세 분석: consumer0.py
이 프로세스는 Kafka로부터 유입되는 원시 데이터(Raw Data)를 PostgreSQL 데이터베이스로 안전하게 이적시키는 데이터 파이프라인의 종착점 역할을 수행합니다.
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
```

if __name__ == "__main__":
    run_consumer()
