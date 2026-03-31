# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(2)

## 📤 Log Producer (`producer.py`)
## : 실행 시 콘솔에 실시간으로 전송되는 로그 데이터가 출력됩니다.

이 모듈은 웹 서버에서 발생하는 실시간 로그 데이터를 시뮬레이션하여 Kafka 브로커로 전송하는 **데이터 생성기**입니다.
```
import time
import json
import random
from kafka import KafkaProducer

def run_producer():
    try:
        # 1. Kafka 연결 설정
        # bootstrap_servers: 도커로 실행 중인 Kafka 서버의 주소와 포트
        # value_serializer: 딕셔너리 데이터를 JSON 문자열로 바꾼 뒤 바이트로 인코딩하여 전송 준비
        producer = KafkaProducer(
            bootstrap_servers=['localhost:9092'],
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )

        # 2. 가상 데이터 생성을 위한 샘플 리스트
        paths = ['/home', '/login', '/product/123', '/cart', '/order']
        status_codes = [200, 201, 404, 500]

        print("🚀 Producer started! Sending logs to Kafka...")

        # 3. 데이터 생성 및 전송 루프 (무한 반복)
        while True:
            # 실시간 로그 데이터 생성 (딕셔너리 형태)
            log_data = {
                "ip": f"192.168.1.{random.randint(1, 255)}", # 랜덤 IP 주소
                "path": random.choice(paths),                # 리스트 중 랜덤 경로
                "status": random.choice(status_codes)       # 리스트 중 랜덤 상태 코드
            }
            
            # 'web_logs'라는 이름의 토픽으로 데이터 전송
            producer.send('web_logs', log_data)
            
            # 전송 성공 확인을 위해 콘솔에 출력
            print(f"Sent: {log_data}")
            
            # 1초 대기 (너무 빠르게 보내면 시스템 부하가 생길 수 있으므로 속도 조절)
            time.sleep(1)
            
    except Exception as e:
        print(f"Error: {e}")
    finally:
        # 4. 종료 시 연결 해제
        # 실행 중 에러가 나거나 강제 종료 시 메모리 누수를 방지하기 위해 닫아줌
        if 'producer' in locals():
            producer.close()

if __name__ == "__main__":
    run_producer()
```
### 🛠️ 주요 기능
- **실시간 스트리밍**: 무한 루프를 통해 중단 없는 데이터 흐름을 생성합니다.
- **데이터 다양성**: 랜덤 함수를 사용하여 다양한 IP 주소, 요청 경로, HTTP 상태 코드를 생성하여 실제 운영 환경과 유사한 데이터를 제공합니다.
- **자동 직렬화**: `value_serializer`를 사용하여 Python 딕셔너리 데이터를 Kafka 전송에 적합한 JSON 바이트 스트림으로 자동 변환합니다.

### 🔍 코드 상세 설명
- **KafkaProducer**: `localhost:9092`에 떠 있는 Kafka 서버와 통신 채널을 엽니다.

- **Value Serializer**: Python의 dict 객체를 Kafka가 이해할 수 있는 `byte stream`으로 자동 변환합니다.

- **Infinite Loop**: while True를 사용하여 실제 운영 환경에서 로그가 `끊임없이` 쌓이는 상황을 재현합니다.

- **Random Logic**: random.choice와 random.randint를 활용하여 매 초마다 서로 다른 사용자의 요청 시나리오를 생성합니다.

### 🚀 실행 방법
```bash
python producer.py
```
<img width="1484" height="426" alt="프로듀서" src="https://github.com/user-attachments/assets/c9cd2588-2c9a-4749-9ee9-ebe54b1e22db" />

