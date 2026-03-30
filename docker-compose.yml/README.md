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
보안을 위해 `.env` 파일을 생성하고 아래 변수들을 본인의 환경에 맞게 수정하여 사용하세요.

```env
# PostgreSQL
POSTGRES_USER=dowon
POSTGRES_PASSWORD=your_password
POSTGRES_DB=logdb

# Grafana
GRAFANA_USER=dowon
GRAFANA_PASSWORD=your_password
```
### 2. 컨테이너 실행
``` 
서비스 실행
docker-compose up -d

서비스 중단
docker-compose down
```

