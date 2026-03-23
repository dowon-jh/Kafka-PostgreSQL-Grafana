services:
  kafka:
    image: apache/kafka:latest
    container_name: kafka-server
    ports:
      - "9092:9092"
    environment:
      - KAFKA_NODE_ID=1
      - KAFKA_PROCESS_ROLES=broker,controller
      - KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
      - KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093


db:
    image: postgres:15
    container_name: postgres-db
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}

grafana:
    image: grafana/grafana-oss:latest
    container_name: my_grafana
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}

  💡 설정의 의미와 사용 이유
분리된 서비스 구조: 각 요소를 독립된 컨테이너로 실행하여 환경 충돌을 방지하고 확장성을 높였습니다.

depends_on: 서비스 간의 의존성(예: DB가 뜬 후 Grafana 실행)을 정의하여 오류를 방지했습니다.

Port Forwarding: 로컬 환경에서 컨테이너 내부 서비스(5432, 9092, 3000 등)에 직접 접근할 수 있도록 설정했습니다.
