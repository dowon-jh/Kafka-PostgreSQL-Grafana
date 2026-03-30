# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(1)

이 프로젝트는 실시간 데이터 파이프라인을 구축하기 위한 핵심 인프라를 Docker Compose로 정의한 설정 파일입니다.

## 🏗️ 시스템 구조
- **Kafka**: 실시간 메시지 스트림 처리 (KRaft 모드)
- **PostgreSQL**: 정형 로그 데이터 저장소
- **Grafana**: 실시간 데이터 시각화 대시보드



## 📄 docker-compose.yml 상세 설명

| 서비스 | 이미지 | 역할 | 포트 |
| :--- | :--- | :--- | :--- |
| **Kafka** | `apache/kafka` | 실시간 메시지 브로커 | 9092 |
| **PostgreSQL** | `postgres:15` | 로그 데이터 보관 DB | 5432 |
| **Grafana** | `grafana-oss` | 시각화 도구 | 3000 |

### 💡 핵심 설정 포인트
1. **KRaft Mode**: 별도의 Zookeeper 컨테이너 없이 Kafka 단독으로 실행 가능하도록 설정하여 리소스를 절약했습니다.
2. **Depends On**: 서비스 간의 의존성을 정의하여 DB가 완전히 가동된 후 시각화 툴이 연동되도록 안정성을 높였습니다.
3. **Environment Variables**: 각 컨테이너의 핵심 설정(계정 정보, 네트워크 리스너 등)을 환경 변수로 관리하여 유연성을 확보했습니다.

---

## 🚀 시작하기

### 1. 환경 변수 설정


```env
#kafka:
    image: apache/kafka:latest                                        -> 아파치 공식 최신 이미지를 사용
    container_name: kafka-server                                      -> 컨테이너의 고유 이름 설정
    ports:
      - "9092:9092"                                                   -> 외부(내 컴퓨터)와 내부 통신 포트 연결
    environment:
      - KAFKA_NODE_ID=1                                               -> 이 서버의 고유 ID (1번)
      - KAFKA_PROCESS_ROLES=broker,controller                         -> 혼자서 데이터 전송(broker)과 관리(controller) 수행
      - KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093          -> 내부 통신 주소 설정
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092         -> 외부(Python 등)에서 접속할 주소
      - KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER                    -> 관리자용 리스너 이름 지정
      - KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093               -> 관리자 투표 시스템 설정

# PostgreSQL
      - POSTGRES_USER=dowon
      - POSTGRES_PASSWORD=your_password
      - POSTGRES_DB=logdb

# Grafana
      - GRAFANA_USER=dowon
      - GRAFANA_PASSWORD=your_password
```
### 2. 컨테이너 실행
``` 
서비스 실행
docker-compose up -d

서비스 중단
docker-compose down
```

### 3. 접속정보
Grafana: http://localhost:3000

PostgreSQL: localhost:5432

Kafka: localhost:9092



