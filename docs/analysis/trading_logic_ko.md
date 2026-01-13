# PRISM-INSIGHT 국내주식 매수/매도 로직 분석 (초등학생 설명 포함)

이 문서는 PRISM-INSIGHT 프로젝트의 국내주식 매수/매도 흐름과 실제 주문 실행 로직을 쉽게 설명하고, GPT 사용 지점을 Gemini로 바꾸려면 어디를 수정해야 하는지 정리합니다.

## 1) 핵심 파일 지도
- 매매 의사결정(시나리오/점수/진입): `cores/agents/trading_agents.py`
- 매수·매도 결정 흐름(기본): `stock_tracking_agent.py`
- 매도 결정(고급, AI+대체 로직): `stock_tracking_enhanced_agent.py`
- 실제 주문 실행(KIS API): `trading/domestic_stock_trading.py`
- 포트폴리오/섹터 제약, 가격/거래대금 보조: `tracking/helpers.py`
- 자동매매/실전모드 설정: `trading/config/kis_devlp.yaml` (예시 파일)

## 2) 전체 흐름(도식)
```
[주식 분석 리포트(PDF)]
        |
        v
[report 읽기 -> 시나리오 생성(LLM)]
        |
        v
[진입 여부 + 점수/목표가/손절가 결정]
        |
        |--(보유 중)--> [건너뜀]
        |
        |--(진입)--> [포트폴리오/섹터/슬롯 체크]
                          |
                          v
                    [DB 기록 + 주문 실행]
                          |
                          v
                    [KIS API 주문 전송]

[보유 중 종목]
        |
        v
[현재가 갱신 + 매도 규칙 평가]
        |
        |--(매도)--> [DB 기록 + 주문 실행]
        |
        `--(보유)--> [계속 보유]
```

## 3) 매수 로직 (초등학생도 이해하기)
쉽게 말하면, “선생님(LLM)이 숙제를 보고 점수를 매겨서, 기준 점수보다 높고 자리(슬롯)가 남아 있으면 사는” 방식입니다.

1. **리포트 읽기**: PDF 분석 보고서를 읽습니다.
2. **선생님에게 물어보기**: LLM이 “사도 될지(진입/관망)”, “몇 점인지”, “목표가/손절가”를 말해줍니다.
3. **자리(보유 슬롯) 확인**: 이미 10개 꽉 찼으면 더 못 삽니다.
4. **같은 분야 너무 많으면 쉬기**: 같은 업종이 너무 많으면 이번엔 패스합니다.
5. **점수 기준 통과**: “진입”이고 점수가 최소 기준 이상이면 매수합니다.
6. **진짜 주문(자동매매가 켜져 있을 때만)**: 실제 계좌로 주문을 보냅니다.

## 4) 매수 로직 (상세 규칙)
### 4-1. 시나리오 생성(LLM)
- 호출 위치: `stock_tracking_agent.py` -> `_extract_trading_scenario()`
- LLM 모델: `OpenAIAugmentedLLM` + `model="gpt-5.2"`
- 입력 정보:
  - 분석 보고서 내용
  - 최근 거래대금 랭킹 변화(보조 지표)
  - 현재 보유 종목 수, 섹터 분포, 투자 기간 분포
  - (옵션) 매매 저널/과거 성과 힌트
- 출력(JSON): `decision`, `buy_score`, `min_score`, `target_price`, `stop_loss`, `investment_period`, `sector`, `trading_scenarios` 등
- LLM 판단 규칙(프롬프트 기반):
  - 시장 상황(강세/약세) 분리
  - 손절 기준과 리스크/리워드 비율
  - 조건부 대기 금지(지금 진입/관망만 선택)
  - 분할 매매 없음(올인/올아웃)
  - 목표가/손절가를 숫자 형식으로 출력
  - 점수(1~10)로 매수 강도 평가
  - 상세 규칙은 `cores/agents/trading_agents.py`

### 4-2. 매수 결정 조건(기본 에이전트 기준)
- 호출 위치: `stock_tracking_agent.py` -> `process_reports()`
- 필수 조건:
  - 이미 보유 중이면 제외
  - 보유 슬롯 < 10
  - 시나리오의 `max_portfolio_size`를 넘지 않음
  - 섹터 집중 제한 통과(`tracking/helpers.py`)
  - 시나리오 `decision == "진입"`
  - `buy_score >= min_score`

### 4-3. 실제 주문 실행
- 호출 위치: `stock_tracking_agent.py` -> `buy_stock()` -> `AsyncTradingContext.async_buy_stock()`
- 실행 모듈: `trading/domestic_stock_trading.py`
- 매수 수량 계산: `floor(매수금액 / 현재가)`
- 시간대별 주문 방식:
  - 09:00~15:30: 시장가 매수
  - 15:40~16:00: 종가 매수
  - 그 외: 예약 주문
- 자동매매 OFF면 주문을 보내지 않음 (`auto_trading: false`)

## 5) 매도 로직 (기본 에이전트)
- 위치: `stock_tracking_agent.py` -> `_analyze_sell_decision()`
- 핵심 규칙 요약:
  1. **손절가 도달**: 즉시 매도
  2. **목표가 도달**: 매도
  3. **단기 투자**: 15일+ 5% 이익이면 매도, 10일+ -3% 손실이면 방어 매도
  4. **일반 규칙**: +10% 이익 매도, -5% 손실 매도
  5. **장기 보유 시**: 30일 손실 지속 또는 60일 3% 이익이면 매도
  6. **장기 투자**: 90일 이상 손실이면 정리

- 매도 실행:
  - `sell_stock()`이 DB 기록 후
  - `AsyncTradingContext.async_sell_stock()`로 실제 주문
  - 실제 주문은 “전량 매도” (분할 매도 없음)

## 6) 매도 로직 (향상 에이전트: AI + 대체 규칙)
- 위치: `stock_tracking_enhanced_agent.py`
- 흐름:
  1. AI가 매도 결정(JSON) 시도
  2. 실패하면 “대체(legacy) 규칙”으로 판단

### 6-1. AI 매도 판단
- LLM: `OpenAIAugmentedLLM` + `model="gpt-5.2"`
- 입력: 현재가, 목표가/손절가, 보유 기간, 섹터, 포트폴리오 분포, 시나리오 정보
- 출력: `should_sell`, `sell_reason`, `confidence`, `portfolio_adjustment` 등

### 6-2. 대체(legacy) 규칙 특징
- 추세 분석(7일 선형 회귀)로 강한 상승 추세면 손절/목표가 도달에도 보유 연장 가능
- 약세장 + 하락 추세면 이익 실현을 더 빠르게
- 손실 -5% 이상이면 (손절가가 없어도) 매도 고려
- 나머지 기준은 기본 에이전트와 유사

## 7) 주문/실전 모드 설정
- 파일: `trading/config/kis_devlp.yaml`
- 핵심 설정:
  - `auto_trading: true/false` (실제 주문 전송 여부)
  - `default_mode: demo/real` (모의/실전)
  - `default_unit_amount`: 1종목당 매수 금액

## 8) GPT -> Gemini 변경 포인트
현재 코드는 OpenAI 전용 클래스(`OpenAIAugmentedLLM`)와 GPT 모델명을 직접 사용합니다.
즉, **Gemini 전환은 “클래스 교체 + 모델명 변경 + 설정/비밀키 추가”**가 필요합니다.

### 8-1. GPT 사용 위치(핵심)
- 매수 시나리오 생성: `stock_tracking_agent.py`
- AI 매도 결정(고급): `stock_tracking_enhanced_agent.py`
- 분석 리포트 생성: `cores/report_generation.py`
- 전체 분석 오케스트레이터: `stock_analysis_orchestrator.py`
- 텔레그램 요약/번역: `telegram_summary_agent.py`, `cores/agents/telegram_translator_agent.py`, `tracking/telegram.py`
- 트레이딩 저널/압축: `tracking/journal.py`, `tracking/compression.py`
- 기타 예시/도구: `examples/translation_utils.py`, `events/jeoningu_trading.py` 등

### 8-2. 설정/키 파일
- 기본 모델 설정: `mcp_agent.config.yaml.example` (openai.default_model)
- API 키: `mcp_agent.secrets.yaml.example` (openai.api_key)
- 의존성: `requirements.txt` (openai 패키지)

### 8-3. 실제로 바꿔야 하는 것(요약)
1. **LLM 클래스 교체**
   - 예: `OpenAIAugmentedLLM` -> Gemini용 LLM 클래스
   - 현재 저장소에는 Gemini용 클래스가 보이지 않음(직접 추가 또는 mcp-agent의 Gemini 지원 필요)

2. **모델명 변경**
   - 예: `gpt-5.2`, `gpt-5-nano`, `gpt-4.1` -> Gemini 모델명

3. **설정/시크릿 추가**
   - OpenAI 키 대신 Gemini 키를 읽도록 구성

4. **의존성 업데이트**
   - OpenAI SDK 대신 Gemini SDK(예: google-generativeai 또는 Vertex SDK) 추가

## 9) 한 줄 요약
이 프로젝트는 “분석 보고서 + LLM 시나리오 + 슬롯/섹터 제한 + 목표/손절 규칙”으로 매수/매도를 결정하고, 실제 주문은 KIS API로 시간대별 시장가/종가/예약 주문을 사용합니다.

---
문서 작성 기준: 2026-01-13 저장소 상태
