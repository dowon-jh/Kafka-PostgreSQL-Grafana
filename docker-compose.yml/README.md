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
