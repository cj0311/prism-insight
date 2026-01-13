# 서버 배포/키 설정/AI API 키 정리 (추가 질문 답변)

아래 내용은 PRISM-INSIGHT 저장소 기준(2026-01-13)으로 정리했습니다.

## 1) 서버에 올릴 때 할 일 (권장 순서)
### A. Docker로 올리는 방법(권장)
1. **서버 준비**
   - Linux 서버 + Docker/Docker Compose 설치

2. **저장소 복사**
   - `git clone` 후 프로젝트 폴더로 이동

3. **설정/시크릿 준비**
   - `.env` 생성: `.env.example` 복사 후 값 입력
   - `mcp_agent.secrets.yaml` 생성: `mcp_agent.secrets.yaml.example` 복사 후 OpenAI/Anthropic 키 입력
   - `mcp_agent.config.yaml` 생성: `mcp_agent.config.yaml.example` 복사 후 Perplexity/서버 설정 수정
   - `trading/config/kis_devlp.yaml` 생성: `trading/config/kis_devlp.yaml.example` 복사 후 KIS 키 입력

4. **컨테이너 빌드/실행**
   - `docker-compose.yml` 기준으로 빌드 및 실행
   - 컨테이너 기본 명령은 `tail -f /dev/null`이므로, 컨테이너 내부에서 파이썬 명령을 실행하거나 별도 스케줄러 사용

5. **스케줄링**
   - `run_stock_analysis.sh`를 크론에 등록하거나, 서버 스케줄러에서 `stock_analysis_orchestrator.py` 실행

### B. 직접 설치(venv/pyenv)로 올리는 방법
1. **Python 환경 구성**
   - 가급적 venv/pyenv 사용 후 `pip install -r requirements.txt`

2. **설정/시크릿 준비** (Docker와 동일)
   - `.env`, `mcp_agent.secrets.yaml`, `mcp_agent.config.yaml`, `trading/config/kis_devlp.yaml`

3. **실행/스케줄링**
   - 예: `python stock_analysis_orchestrator.py --mode morning`
   - 크론 또는 시스템 서비스로 주기 실행

> 참고 파일: `docker-compose.yml`, `run_stock_analysis.sh`, `stock_analysis_orchestrator.py`

## 2) 텔레그램/주식매수매도 API 키 입력 위치
### 2-1. 텔레그램
- 기본 텔레그램 채널 및 봇 토큰:
  - `.env` 파일에 입력
    - `TELEGRAM_BOT_TOKEN`
    - `TELEGRAM_CHANNEL_ID`
  - 로딩 위치: `telegram_config.py` (환경변수에서 읽음)

- AI 대화봇(별도 토큰):
  - `.env` 파일에 입력
    - `TELEGRAM_AI_BOT_TOKEN`
  - 로딩 위치: `telegram_ai_bot.py`

- 다국어 채널(옵션):
  - `.env` 파일에 입력
    - `TELEGRAM_CHANNEL_ID_EN`, `TELEGRAM_CHANNEL_ID_JA` 등

### 2-2. 국내주식 매수/매도(KIS API)
- 파일: `trading/config/kis_devlp.yaml`
  - `trading/config/kis_devlp.yaml.example`을 복사해서 작성
  - 실전/모의 앱키, 시크릿, 계좌번호 등 입력

### 2-3. KRX 데이터 로그인(옵션)
- `.env` 파일
  - `KRX_ID`, `KRX_PW`, `KRX_LOGIN_METHOD`
- MCP 서버 사용 시 `mcp_agent.config.yaml`의 `kospi_kosdaq` 환경변수도 필요할 수 있음

## 3) AI API 키 정리 (사용되는 AI API 키 전체)
아래는 **AI 호출에 직접 사용되는 키들만** 정리했습니다.

### 3-1. OpenAI API (GPT/Whisper)
- **용도**: 주식 분석/매수 시나리오/번역/요약/Whisper 등
- **입력 위치**: `mcp_agent.secrets.yaml`
  - `openai.api_key: "sk-..."`
- **예시 파일**: `mcp_agent.secrets.yaml.example`

### 3-2. Anthropic API (Claude)
- **용도**: 텔레그램 상담/보고서 평가 등 Claude 기반 기능
- **입력 위치**: `mcp_agent.secrets.yaml`
  - `anthropic.api_key: "sk-ant-..."`
- **예시 파일**: `mcp_agent.secrets.yaml.example`

### 3-3. Perplexity API (리서치/밸류 참고)
- **용도**: 밸류 비교 등 외부 리서치 보조
- **입력 위치**: `mcp_agent.config.yaml`
  - `mcp.servers.perplexity.env.PERPLEXITY_API_KEY`
- **예시 파일**: `mcp_agent.config.yaml.example`

> 참고 파일: `mcp_agent.secrets.yaml.example`, `mcp_agent.config.yaml.example`, `.env.example`

---
필요하면 배포 방식(도커/직접설치) 중 하나를 선택해서 체크리스트를 더 상세하게 만들어줄 수 있습니다.
