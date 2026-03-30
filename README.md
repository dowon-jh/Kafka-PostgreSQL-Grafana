Kafka 서비스 (메시지 브로커)
Kafka는 실시간으로 흐르는 데이터를 전달해주는 통로입니다.

image: apache/kafka:latest: 아파치 공식 최신 이미지를 가져옵니다.

container_name: kafka-server: 도커 내부에서 이 컨테이너를 부를 별명입니다.

ports: - "9092:9092": 로컬 PC(9092)와 컨테이너 내부(9092)를 연결합니다.

KAFKA_NODE_ID=1: 클러스터 내에서 이 브로커를 식별하는 고유 번호입니다.

KAFKA_PROCESS_ROLES=broker,controller: KRaft 모드를 사용한다는 뜻입니다. (예전처럼 별도의 Zookeeper 없이 혼자서 관리자 역할까지 다 한다는 의미죠.)

KAFKA_ADVERTISED_LISTENERS: 클라이언트(Python 코드 등)가 Kafka에 접속할 때 사용하는 주소입니다. 여기서는 localhost:9092를 사용합니다.

DB 서비스 (데이터베이스)
데이터를 영구적으로 저장하는 장소입니다.

image: postgres:15: 안정적인 PostgreSQL 15 버전을 사용합니다.

POSTGRES_USER/PASSWORD: DB에 접속할 마스터 계정 정보입니다.

POSTGRES_DB=logdb: 실행 시 자동으로 생성될 데이터베이스 이름입니다.

Grafana 서비스 (시각화)
DB에 쌓인 데이터를 예쁜 그래프로 보여주는 대시보드입니다.

depends_on: - db: DB가 먼저 켜진 후에 Grafana를 실행하라는 순서 지정입니다.

GF_SECURITY_ADMIN_USER/PASSWORD: Grafana 웹 사이트 로그인 계정입니다. (어제 말씀하신 생일 비밀번호가 여기 들어갔네요!)
