# 🚀 Real-time Log Pipeline: Kafka, Postgres & Grafana
데이터 생성부터 시각화까지, 실시간 데이터 엔지니어링의 전 과정을 다루는 End-to-End 프로젝트입니다.

본 프로젝트는 가상의 웹 로그를 실시간으로 생성하고, 메시지 브로커(Kafka)를 통해 유통하며, 목적에 맞는 4가지 컨슈머 로직을 거쳐 데이터베이스(PostgreSQL)에 적재한 뒤 대시보드(Grafana)로 모니터링하는 파이프라인을 구축합니다.

### 🏗️ 시스템 아키텍처 (System Architecture)
전체 데이터 흐름은 다음과 같은 구조로 설계되었습니다.

**Producer**: 실시간 웹 로그(IP, 요청 경로, 상태 코드)를 생성하여 Kafka로 전송합니다.

**Kafka (KRaft Mode)**: Zookeeper 없이 동작하는 최신 모드로 메시지를 효율적으로 중계합니다.

**Multi-Consumers (0~3)**: 동일한 토픽을 구독하여 각기 다른 비즈니스 로직(저장, 필터링, 보안, 마스킹)을 처리합니다.

**PostgreSQL**: 구조화된 로그 데이터를 영구 저장소에 적재합니다.

**Grafana**: SQL 쿼리를 기반으로 에러율 및 접속 현황을 실시간 시각화합니다.

### 🛠️ 기술 스택 (Tech Stack)
**Infrastructure**: Docker, Docker Compose

**Message Broker**: Apache Kafka (Latest KRaft Mode)

**Database**: PostgreSQL 15

**Visualization**: Grafana OSS

**Language**: Python 3.x (kafka-python, psycopg2-binary)

### 📂 프로젝트 구성 (Project Structure)
**producer.py**: 실시간 로그 생성기

**consumer0.py**: 기본 데이터 적재 (All Ingest)

**consumer1.py**: 스팸 방지 필터 (5s Cooldown)

**consumer2.py**: 보안 필터 (Admin/Login 접근 차단)

**consumer3.py**: 개인정보 보호 (IP Masking)

**docker-compose.yml**: 인프라 정의 파일

-----------------------------------------
## ✅ 파이프라인 검증 (Integrated Test)

4개의 컨슈머를 동시에 가동하여 동일한 로그 스트림이 각 비즈니스 로직에 따라 처리되는지 확인

### 🖥️ DB 적재 데이터 모니터링 (psql)
<img width="981" height="420" alt="종합결과확인2" src="https://github.com/user-attachments/assets/7bcf49af-353b-4681-826a-cce39ea88b69" />


- **병렬 처리**: 동일 토픽을 구독하는 `consumer0`(전체저장)과 `consumer3`(마스킹)의 데이터가 동시에 적재됨
- **데이터 변환**: 원본 IP와 비식별화된 IP가 실시간으로 변환되어 DB 테이블(`web_logs`)에 기록됨
- **시간 일관성**: 밀리초(ms) 단위의 타임스탬프(`ts`)를 통해 데이터 흐름의 순서와 지연 없음을 확인
-------------------------
-------------------------
## 📤 Log Producer (`producer.py`)

"이 모듈은 웹 서버에서 발생하는 실시간 로그 데이터를 시뮬레이션하여 Kafka 브로커로 전송하는 역할"

#### 📜 핵심 코드

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
"Kafka의 `web_logs` 토픽을 구독하여 실시간 데이터를 수신하고, 이를 PostgreSQL 데이터베이스에 저장하는 가장 기본적인 `데이터 수집기`"

#### 🛠️ 핵심 기능
- **Kafka Subscription**: `web_logs` 토픽에 발행되는 메시지를 실시간으로 모니터링합니다.
- **Data Deserialization**: JSON 바이트 데이터를 Python 딕셔너리 객체로 변환하여 처리합니다.
- **RDBMS Integration**: `psycopg2` 라이브러리를 사용하여 구조화된 로그 데이터를 PostgreSQL 테이블에 영구적으로 보존합니다.

#### 📜 주요 로직
- **Offset 설정**: `auto_offset_reset='earliest'` 옵션을 통해 컨슈머가 처음 구동될 때 과거에 쌓여있던 로그부터 유실 없이 처리하도록 보장합니다.
- **Transaction 관리**: 각 로그 메시지가 삽입될 때마다 `commit()`을 수행하여 데이터의 원자성(Atomicity)을 유지합니다.

#### 🚀 실행 방법
```bash
python consumer0.py
```
-------------------------
# 🚫 Anti-Spam Consumer (`consumer1.py`)
"동일한 사용자가 짧은 시간 내에 반복적으로 보내는 데이터를 필터링"

#### 🛠️ 핵심 기능: IP 기반 5초 쿨타임 시스템 (Anti-Spam).
#### ⚙️ 작동 원리: Python Dictionary를 활용한 실시간 메모리 캐싱 모니터링.
#### ✏️사용 목적: 
  - 무의미한 중복 로그 제거를 통한 저장 공간 절약.
  - 비정상적인 반복 요청(도배 등)에 대한 1차 방어막 구축.
#### ‼️ 특이사항: `consumer0.py`와 같은 토픽을 구독하지만, 조건에 맞는 데이터만 선택적으로 PostgreSQL에 적재합니다.

--------------------------
## 🛡️ Consumer 2: Security Filter (consumer2.py)
"보안 민감 경로에 대한 실시간 접근 차단 및 필터링"

#### 📋 개요
모든 로그 중 /admin, /login과 같이 보안상 중요한 경로로의 접근을 감지하고, 해당 데이터를 메인 DB 저장소에서 제외하는 보안 특화 컨슈머입니다.

#### ⚙️ 핵심 로직
```
### 블랙리스트 경로 설정
excluded_paths = ['/admin', '/login']

### 감시 및 차단 로직
if path in excluded_paths:
    print(f"🛡️ BLOCKED: {path}")
    continue # DB 저장 단계를 건너뜀
```
#### ✨ 주요 효과
- **데이터 정제**: 불필요하거나 위험한 시도 로그가 통계 데이터에 섞이는 것을 방지합니다.

- **실시간 경보**: 터미널 로그를 통해 비정상적인 접근 시도를 즉각 확인 가능합니다.

------------------------------
## 🎭 Consumer 3: Privacy Masking (consumer3.py)
"개인정보 보호를 위한 IP 주소 비식별화 처리"

#### 📋 개요
수집된 로그의 IP 주소 뒷자리를 ***로 변환하여 저장함으로써, 사용자의 개인정보를 보호하고 데이터 보안 규정을 준수하는 데이터 가공 컨슈머입니다.

#### ⚙️ 핵심 로직
```
def mask_ip(ip):
    # '192.168.1.123' -> '192.168.1.***'
    parts = ip.split('.')
    parts[-1] = '***'
    return '.'.join(parts)
```
#### 📊가공된 데이터 저장
```
masked_ip = mask_ip(original_ip)
cur.execute("INSERT INTO web_logs (ip, ...) VALUES (%s, ...)", (masked_ip, ...))
```
#### ✨ 주요 효과
- **개인정보 보호**: 데이터 유출 시에도 실제 사용자 IP가 노출되지 않도록 방어합니다.

- **통계 가치 유지**: 전체 IP는 가리되 앞부분(대역대)은 남겨두어 지역별/대역별 통계 분석은 가능하게 합니다.
