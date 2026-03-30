# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(5)

## 🛡️ Security Filter Consumer (consumer2.py)
#### : 이 컨슈머는 전체 로그 스트림 중에서 관리자 페이지(/admin)나 로그인 페이지(/login)와 같이 보안상 민감한 경로에 대한 접근을 실시간으로 필터링하는 데이터 보안 처리기입니다.

```
import json
import psycopg2
from kafka import KafkaConsumer

def run_consumer():
    # [1] 데이터베이스 및 Kafka 연결 설정 (PostgreSQL 15 & Kafka KRaft)
    conn = psycopg2.connect(host="127.0.0.1", database="logdb", user="dowon", password="1234", port="5432")
    cur = conn.cursor()
    consumer = KafkaConsumer('web_logs', bootstrap_servers=['localhost:9092'], value_deserializer=lambda m: json.loads(m.decode('utf-8')))

    # [2] 필터링 대상 리스트 정의 (보안 민감 경로)
    # ! 역할: 접근을 제한하거나 별도로 관리해야 할 경로들을 리스트로 관리합니다.
    excluded_paths = ['/admin', '/login']

    print(f"🔒 Consumer 2: Security Filter (Excluding {excluded_paths})...")

    # [3] 메시지 수신 루프
    for message in consumer:
        log = message.value
        path = log['path'] # 로그 데이터에서 접근 경로(path) 추출

        # [4] 조건문: 민감 경로 접근 확인
        # ? 동작: 현재 로그의 path가 'excluded_paths' 목록에 포함되어 있는지 검사합니다.
        if path in excluded_paths:
            # ! 결과: 민감 경로일 경우 DB 저장을 건너뛰고(continue) 차단 로그를 출력합니다.
            print(f"🛡️ BLOCKED: Security sensitive path accessed: {path}")
            continue
        
        # [5] 일반 경로 데이터 저장
        # ? 역할: 필터를 통과한 일반적인 접근 데이터만 PostgreSQL 테이블에 적재합니다.
        cur.execute("INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)", (log['ip'], path, log['status']))
        conn.commit()
        print(f"✅ SAVED: Public path access - {path}")

if __name__ == "__main__":
    run_consumer()

```

### ❓ 왜 이 코드를 사용했는가? (Motivation)
- **보안 계층의 분리**: 방화벽이나 애플리케이션 레벨 외에도, 데이터 파이프라인 단계에서 비정상적인 접근 시도를 감지하는 추가적인 방어선을 구축하기 위함입니다.

- **데이터 정제(Cleaning)**: 분석 가치가 낮거나 보안상 노출되면 안 되는 민감한 접근 로그가 메인 통계 DB에 섞이는 것을 방지합니다.

- **실시간 대응**: 로그가 쌓인 후 분석하는 것이 아니라, 유입되는 즉시 위험을 감지하는 메커니즘을 구현하기 위해 사용했습니다.
------------------------
### 🛠️ 이 코드를 어떻게 사용했는가? (Implementation)
- **Blacklist 방식 채택**: excluded_paths 리스트를 통해 관리 포인트(민감 경로)를 명시적으로 지정했습니다.

- **Selective Ingestion**: 모든 데이터를 수집하는 consumer0과 달리, 특정 비즈니스 로직(보안)에 부합하는 데이터만 선택적으로 수집(Ingest)하도록 구현했습니다.

- **실시간 모니터링**: 차단된 요청에 대해 🛡️ BLOCKED 접두어를 붙여 터미널에서 즉각적인 시각적 피드백을 받을 수 있게 설계했습니다.
-------------------------
### ✨ 이 코드를 사용한 결과 (Outcome)
- **깨끗한 데이터 확보**: PostgreSQL의 web_logs 테이블에는 일반 사용자의 서비스 이용 패턴만 남게 되어 데이터 품질이 향상되었습니다.

- **위협 감지 가시성**: 관리자 페이지 등에 대한 비정상적인 접근 시도를 실시간 로그를 통해 파악할 수 있게 되었습니다.

- **효율적인 리소스 사용**: 불필요한 데이터를 저장하지 않음으로써 데이터베이스의 저장 공간 및 I/O 성능을 최적화했습니다.
-----------------------------
### 💡 이 코드를 사용함으로써 얻은 인사이트 (Insight)
- **Pub/Sub 모델의 유연성**: 동일한 Kafka 토픽(web_logs)을 여러 컨슈머가 구독하면서, 한쪽은 전체 저장(consumer0)을 하고 다른 쪽은 보안 필터링(consumer2)을 하는 등 목적에 따른 다중 처리가 가능함을 깨달았습니다.

- **프로액티브(Proactive) 데이터 처리**: 데이터가 저장되기 전(Pre-storage) 단계에서 로직을 처리하는 것이 사후 분석보다 훨씬 강력한 제어 수단이 됨을 이해했습니다.

- **확장 가능한 아키텍처**: 차단 경로뿐만 아니라 특정 IP 대역 차단, 특정 상태 코드 감지 등 다양한 보안 정책을 리스트 추가만으로 쉽게 확장할 수 있다는 점을 학습했습니다.
