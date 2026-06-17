# Docker Compose Infrastructure

이 문서는 Kafka, PostgreSQL, Grafana 로컬 실습 환경을 구성하는 `docker-compose.yml`의 역할을 정리한 문서입니다.

## 구성 서비스

| 서비스 | 역할 | 기본 포트 |
| --- | --- | --- |
| Kafka | 실시간 로그 메시지를 중계하는 broker | 9092 |
| PostgreSQL | 구조화된 로그 데이터를 저장하는 DB | 5432 |
| Grafana | PostgreSQL 데이터를 시각화하는 대시보드 | 3000 |

## 핵심 설계

- Kafka는 Zookeeper 없이 KRaft Mode로 구성합니다.
- PostgreSQL은 `web_logs` 테이블을 통해 로그 데이터를 저장합니다.
- Grafana는 PostgreSQL datasource를 연결해 SQL 기반 패널을 구성합니다.
- 계정, 비밀번호, DB명은 `.env`에서 읽도록 관리합니다.

## `.env` 예시

```env
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_DB=logdb
POSTGRES_USER=dowon
POSTGRES_PASSWORD=change_me
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_TOPIC=web_logs
```

## 실행 명령

```bash
docker-compose up -d
```

중단:

```bash
docker-compose down
```

## PostgreSQL 테이블 예시

```sql
CREATE TABLE web_logs (
    id BIGSERIAL PRIMARY KEY,
    ip VARCHAR(50) NOT NULL,
    path VARCHAR(200) NOT NULL,
    status INTEGER NOT NULL,
    ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 점검 포인트

- Kafka가 `localhost:9092`로 접근 가능한지 확인합니다.
- PostgreSQL 접속 정보가 `.env`와 일치하는지 확인합니다.
- Grafana datasource에서 PostgreSQL 연결 테스트가 성공하는지 확인합니다.
- DB 비밀번호는 README나 코드에 직접 기록하지 않습니다.
