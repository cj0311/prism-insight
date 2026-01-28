# PRISM-INSIGHT 실시간 트레이딩 시그널 구독 가이드 (Redis Streams)

PRISM-INSIGHT의 AI 기반 실시간 매매 시그널을 Redis Streams를 통해 받아볼 수 있습니다.

## 📋 개요

- **무료 제공**: PRISM-INSIGHT 측 비용 없음 (Upstash Free Tier 사용)
- **실시간 스트림**: 매수/매도 시그널을 즉시 수신
- **커스터마이징 가능**: 받은 시그널로 자체 로직 구현 가능
- **샘플 코드 제공**: Python 예제 코드 포함
- **Graceful Degradation**: Redis 미설정 시 기존 로직에 영향 없이 정상 동작

## 💰 비용 안내

**참고**: GCP Pub/Sub 기반 Pub/Sub도 지원합니다. 자세한 내용은 `docs/EXTERNAL_SUBSCRIBER_GUIDE.md`를 참조하세요.

### PRISM-INSIGHT 측
- 무료 (Upstash Free Tier 사용)

### 구독자 측
- **Upstash Redis 요금**: https://upstash.com/pricing
- **무료 할당량**: 월 10,000 커맨드, 100MB 데이터 저장 (개인 사용 충분)
- **예상 비용**: 시그널이 적어 대부분 무료 범위 내

## 🚀 빠른 시작

### 1. Upstash Redis 생성

1. Upstash Console에 가입합니다.
2. 새 Redis 데이터베이스를 생성합니다.
3. 생성된 데이터베이스의 "REST API" 탭에서 `UPSTASH_REDIS_REST_URL`과 `UPSTASH_REDIS_REST_TOKEN`을 복사합니다.

### 2. 환경 변수 설정

프로젝트 루트의 `.env` 파일에 다음을 추가합니다:

```bash
# Redis Streams (Optional - for trading signal pub/sub)
UPSTASH_REDIS_REST_URL=https://your-redis.upstash.io
UPSTASH_REDIS_REST_TOKEN=your_token_here
```

### 3. 예제 코드 실행

#### Python 환경 설정

```bash
# 저장소 클론
git clone https://github.com/dragon1086/prism-insight.git
cd prism-insight

# 가상환경 생성 (권장)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 패키지 설치
pip install upstash-redis python-dotenv
```

#### 구독자 실행

```bash
# 새 메시지만 수신 (기본값)
python examples/messaging/redis_subscriber_example.py

# 처음부터 모든 메시지 수신
python examples/messaging/redis_subscriber_example.py --from-beginning

# 시뮬레이션 모드 (실제 매매 X)
python examples/messaging/redis_subscriber_example.py --dry-run

# 실제 자동매매 연동 (주의!)
python examples/messaging/redis_subscriber_example.py
```

## 📊 수신되는 데이터 형식

Redis Streams에 발행되는 시그널은 JSON 문자열 형태로 `data` 필드에 저장됩니다.

### 매수 시그널 (BUY)

```json
{
  "type": "BUY",
  "ticker": "005930",
  "company_name": "삼성전자",
  "price": 82000,
  "timestamp": "2025-01-15T10:30:00",
  "target_price": 90000,
  "stop_loss": 75000,
  "investment_period": "단기",
  "sector": "반도체",
  "rationale": "AI 반도체 수요 증가",
  "buy_score": 8,
  "source": "AI분석",
  "trade_success": true,
  "trade_message": "매수 완료"
}
```

### 매도 시그널 (SELL)

```json
{
  "type": "SELL",
  "ticker": "005930",
  "company_name": "삼성전자",
  "price": 90000,
  "timestamp": "2025-01-20T14:20:00",
  "buy_price": 82000,
  "profit_rate": 9.76,
  "sell_reason": "목표가 달성",
  "source": "AI분석",
  "trade_success": true,
  "trade_message": "매도 완료"
}
```

### 이벤트 시그널 (EVENT)

```json
{
  "type": "EVENT",
  "ticker": "005930",
  "company_name": "삼성전자",
  "price": 82000,
  "timestamp": "2025-01-15T12:00:00",
  "event_type": "YOUTUBE",
  "event_description": "신규 영상 업로드",
  "source": "유튜버_홍길동"
}
```

## 💡 활용 예시

`examples/messaging/redis_subscriber_example.py` 스크립트를 참고하여 다양한 커스텀 로직을 구현할 수 있습니다.

### 1. 커스텀 알림 시스템

- 텔레그램, 슬랙, 이메일 등으로 실시간 매매 시그널 알림

### 2. 자동매매 봇 연동

- 다른 증권사 API 또는 여러 계좌에 분산 매매
- 시그널 기반으로 비중 조절, 필터링 등 커스텀 매매 로직 구현

### 3. 데이터 수집 및 분석

- 시그널을 데이터베이스에 저장하여 백테스팅, 성과 분석

### 4. 새로운 시그널 생성 로직 연동

- PRISM-INSIGHT의 로직 외에 유튜브, 뉴스 등 외부 트리거를 통해 `EVENT` 타입 시그널을 발행하여 연동

## 🛠️ 문제 해결

### 1. `upstash-redis` 패키지 설치 오류

```bash
pip install upstash-redis
```

### 2. Redis 연결 정보 누락

- `.env` 파일에 `UPSTASH_REDIS_REST_URL`과 `UPSTASH_REDIS_REST_TOKEN`이 올바르게 설정되었는지 확인합니다.
- 또는 `redis_subscriber_example.py` 실행 시 `--redis-url` 및 `--redis-token` 옵션으로 직접 전달합니다.

### 3. 메시지가 수신되지 않음

- `redis_subscriber_example.py` 실행 시 `--from-beginning` 옵션을 사용하여 스트림의 처음부터 모든 메시지를 수신해봅니다.
- Upstash Redis 콘솔에서 스트림 (`prism:trading-signals`)에 메시지가 발행되고 있는지 확인합니다.

## 📞 지원 및 문의

- **GitHub Issues**: https://github.com/dragon1086/prism-insight/issues
- **Telegram 채널**: @stock_ai_agent
- **문서**: https://github.com/dragon1086/prism-insight/docs

## ⚠️ 면책 조항

- 본 시그널은 AI 기반 분석 결과이며 투자 권유가 아닙니다.
- 모든 투자 결정과 손실에 대한 책임은 전적으로 투자자 본인에게 있습니다.
- 실제 매매 전 충분한 검토와 테스트를 권장합니다.
- PRISM-INSIGHT는 시그널 정확성을 보장하지 않습니다.

## 🔄 업데이트 내역

- 2025-11-22: Redis Streams Pub/Sub 시스템 도입

---

**Happy Trading! 📈**
