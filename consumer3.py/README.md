# 🐳 Kafka-PostgreSQL-Grafana Infrastructure(6)


## 🎭 Privacy Masking Consumer (consumer3.py)
#### :이 컨슈머는 수집된 로그 데이터에서 개인정보에 해당하는 IP 주소의 뒷자리를 마스킹(Masking) 처리하여 저장하는 데이터 변환 및 개인정보 보호 처리기입니다.

```
import json
import psycopg2
from kafka import KafkaConsumer

# [1] IP 마스킹 함수 정의
def mask_ip(ip):
    # 역할: '192.168.1.123' 형태의 문자열을 '.' 기준으로 분리
    parts = ip.split('.')
    # 마지막 자리(123)를 '***'로 교체하여 식별 불가능하게 만듦
    parts[-1] = '***'
    # 분리된 조각들을 다시 '.'으로 합쳐서 반환
    return '.'.join(parts)

def run_consumer():
    # [2] 인프라 연결: DB 및 Kafka 접속 설정
    conn = psycopg2.connect(host="127.0.0.1", database="logdb", user="dowon", password="1234", port="5432")
    cur = conn.cursor()
    consumer = KafkaConsumer('web_logs', bootstrap_servers=['localhost:9092'], value_deserializer=lambda m: json.loads(m.decode('utf-8')))

    print("🎭 Consumer 3: Privacy Mode (IP Masking)...")

    # [3] 데이터 수신 및 가공 루프
    for message in consumer:
        log = message.value
        original_ip = log['ip']
        
        # [4] 데이터 변환 실행: 원본 IP를 마스킹된 IP로 변환
        masked_ip = mask_ip(original_ip)

        # [5] 가공된 데이터 저장: 원본 대신 마스킹된 정보를 DB에 적재
        cur.execute("INSERT INTO web_logs (ip, path, status) VALUES (%s, %s, %s)", (masked_ip, log['path'], log['status']))
        conn.commit()
        
        # 원본과 변환 결과를 대조하여 콘솔에 출력
        print(f"🎭 MASKED: {original_ip} -> {masked_ip} (Saved)")

if __name__ == "__main__":
    run_consumer()

```
<img width="747" height="331" alt="컨수머3" src="https://github.com/user-attachments/assets/b81e021c-ea38-4b33-9ad5-29c803e5d5f4" />

### ❓ 왜 이 코드를 사용했는가? (Motivation)
- **법적 규제 준수**: GDPR(유럽 개인정보보호법)이나 국내 개인정보보호법에 따라 사용자의 IP 주소는 개인 식별 정보로 간주될 수 있으므로, 이를 비식별 처리하여 저장할 필요가 있습니다.

- **데이터 보안**: 데이터베이스가 해킹당하더라도 사용자의 실제 접속 IP 주소가 유출되지 않도록 **보안의 깊이(Defense in Depth)**를 더하기 위함입니다.

- **통계적 가치 유지**: 전체 IP는 가리더라도 앞부분(192.168.1.***)을 남겨둠으로써, 대략적인 접속 대역대나 지역 통계는 여전히 분석 가능하도록 타협점을 찾기 위해 사용했습니다.

---------------------------------
### 🛠️ 이 코드를 어떻게 사용했는가? (Implementation)
- **ETL(Extract-Transform-Load) 적용**: Kafka에서 데이터를 추출(Extract)하고, 마스킹 함수로 변환(Transform)한 뒤, DB에 적재(Load)하는 정석적인 파이프라인을 구현했습니다.

- **비식별화 알고리즘**: 단순하지만 강력한 문자열 마스킹 방식을 채택하여 시스템 리소스를 최소화하면서 개인정보를 보호했습니다.

- **실시간 피드백**: 마스킹 전/후 데이터를 콘솔에 병렬로 출력하여 변환 로직이 정확히 작동하는지 실시간으로 검증했습니다.

----------------------------------
### ✨ 이 코드를 사용한 결과 (Outcome)
- **개인정보 보호 강화**: 데이터베이스 내의 ip 컬럼 데이터가 192.168.1.*** 형태로 저장되어 관리자의 실수나 외부 유출로부터 사용자 정보를 보호하게 되었습니다.

- **신뢰할 수 있는 데이터 저장소**: 보안 정책을 준수하는 정제된 데이터셋이 구축되었습니다.

- **분석 효율성**: 개별 사용자는 구분하지 못하지만, 동일 대역 접속자들의 통계적 경향성은 유지할 수 있게 되었습니다.

--------------------------------------
## 💡 이 코드를 사용함으로써 얻은 인사이트 (Insight)
- **데이터 가공의 위치**: 데이터가 저장소에 들어가기 전(In-flight)에 가공하는 것이 사후 처리보다 데이터 보안 측면에서 훨씬 유리함을 배웠습니다.

- **데이터 유용성 vs 개인정보**: 데이터를 무조건 지우는 것이 아니라, 마스킹을 통해 '통계적 가치'와 '프라이버시' 사이의 균형을 맞추는 법을 학습했습니다.

- **확장성**: 동일한 원리로 이메일 주소 마스킹, 전화번호 뒷자리 가리기 등 다양한 개인정보 보호 로직을 파이프라인에 이식할 수 있다는 확신을 얻었습니다.
