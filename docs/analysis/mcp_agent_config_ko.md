# mcp_agent.config.yaml 항목별 분석 (AI 관련 포함)

본 문서는 저장소에 실제 `mcp_agent.config.yaml`이 없고, `mcp_agent.config.yaml.example`만 존재하는 상태를 기준으로 작성했습니다.

## 파일 상태
- 실제 파일: **없음**
- 분석 대상: `mcp_agent.config.yaml.example`

## 최상위 구성
### 1) `$schema`
- **의미**: 설정 파일 스키마 경로
- **효과**: 에디터/검증 도구가 설정 형식을 검사할 때 사용

### 2) `execution_engine: asyncio`
- **의미**: MCP 실행 엔진이 asyncio 기반임을 지정
- **효과**: 비동기 LLM 호출 및 MCP 서버 연동에 맞는 실행 방식

### 3) `logger`
- **의미**: 로그 출력 정책
- **세부 항목**:
  - `type: console` → 콘솔 출력
  - `level: info` → 기본 로그 레벨
  - `batch_size: 100` → 로그 묶음 처리 단위
  - `flush_interval: 2` → 로그 플러시 주기(초)
  - `max_queue_size: 2048` → 로깅 큐 최대 크기
  - `http_endpoint`, `http_headers`, `http_timeout` → HTTP 로그 전송 시 사용(현재 비어있음)
- **AI 영향**: 간접적(LLM 호출/도구 호출 로그의 기록 방식만 영향)

## `mcp.servers` (MCP 도구 서버 목록)
LLM이 외부 도구를 호출할 때 연결되는 서버 정의입니다.

### 4) `webresearch`
- **command**: `npx`
- **args**: `@mzxrai/mcp-webresearch@latest`
- **효과**: 웹 리서치용 MCP 서버 실행
- **AI 영향**: LLM이 웹 리서치 도구를 호출할 수 있음

### 5) `firecrawl`
- **command**: `npx`
- **args**: `firecrawl-mcp`
- **env**:
  - `FIRECRAWL_API_KEY`: Firecrawl API 키
- **효과**: Firecrawl 기반 크롤링/추출 도구
- **AI 영향**: 외부 문서/웹페이지 수집 시 사용 가능

### 6) `kospi_kosdaq`
- **command**: `python3`
- **args**: `-m kospi_kosdaq_stock_server`
- **env**:
  - `KRX_ID`, `KRX_PW`: KRX 로그인 계정
  - `KRX_LOGIN_METHOD`: `krx` 또는 `kakao`
  - `KAKAO_ID`, `KAKAO_PW`: 카카오 로그인 옵션(주석 처리됨)
- **효과**: KRX 데이터 조회용 MCP 서버
- **AI 영향**: 주가/거래 데이터 조회 도구로 사용

### 7) `perplexity`
- **command**: `node`
- **args**: `perplexity-ask/dist/index.js`
- **env**:
  - `PERPLEXITY_API_KEY`: Perplexity API 키
- **효과**: 밸류 비교/리서치 질의용 도구 서버
- **AI 영향**: LLM 프롬프트에서 외부 리서치 호출 시 사용

### 8) `sqlite`
- **command**: `uv`
- **args**: `--directory sqlite run mcp-server-sqlite --db-path stock_tracking_db`
- **효과**: SQLite MCP 서버 실행 (보유 종목/저널/거래 이력 DB 접근)
- **AI 영향**: LLM이 포트폴리오/거래 기록을 조회하거나 업데이트 가능

### 9) `time`
- **command**: `uvx`
- **args**: `mcp-server-time`
- **효과**: 현재 시간 조회 서버
- **AI 영향**: 매매 판단 시 “장중/장마감” 구분에 사용

## `openai` (AI 모델 설정)
### 10) `openai.default_model: gpt-5.1`
- **의미**: OpenAI 기본 모델 지정
- **효과**: LLM 호출 시 기본 모델이 gpt-5.1로 설정

### 11) `openai.reasoning_effort: high`
- **의미**: 추론 강도 설정(높음)
- **효과**: LLM 응답 품질/비용/속도에 영향 가능

> 주의: 실제 API 키는 이 파일이 아니라 `mcp_agent.secrets.yaml`에 둠.

---
참고 파일
- `mcp_agent.config.yaml.example`
- `mcp_agent.secrets.yaml.example`
- `.env.example`

---
## AI 사용비용이 발생할 수 있는 항목 정리
이 파일 기준으로 **직접 비용이 발생할 수 있는 부분**은 아래입니다.

### 1) `openai` 섹션
- `openai.default_model: gpt-5.1`
  - **영향**: 호출되는 기본 모델이 고급 모델일수록 비용 상승
- `openai.reasoning_effort: high`
  - **영향**: 추론 강도가 높을수록 응답 품질은 좋아질 수 있으나 토큰 사용량/비용이 증가할 가능성

### 2) `mcp.servers.perplexity`
- `PERPLEXITY_API_KEY`
  - **영향**: Perplexity API 사용 시 과금
  - **비용 발생 조건**: LLM이 리서치 도구를 호출할 때만

### 3) `mcp.servers.firecrawl`
- `FIRECRAWL_API_KEY`
  - **영향**: Firecrawl 크롤링/추출 API 사용 시 과금
  - **비용 발생 조건**: LLM이 Firecrawl 도구를 호출할 때만

### 4) 비용이 거의 없는/직접 과금이 없는 항목
- `mcp.servers.webresearch` (로컬 npx 실행)
- `mcp.servers.sqlite`, `mcp.servers.time`, `mcp.servers.kospi_kosdaq`
  - **직접적인 AI API 비용은 없음**
  - 단, `kospi_kosdaq`는 외부 데이터 접근(로그인/API) 비용/제한이 별도일 수 있음

요약: **AI 비용은 주로 OpenAI 기본 모델/추론 강도, 그리고 Perplexity/Firecrawl 호출 여부에 의해 발생**합니다.
