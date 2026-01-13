# 현재 프로젝트 상태 확인 + 기록 (Ubuntu 서버 설치 전 준비)

이 문서는 요청에 따라 현재 저장소 상태를 확인한 결과와, 앞서 제안한 서버 설치/운영 요약을 함께 기록합니다.

## 1) 현재 프로젝트 상태(실측)
확인 기준: `C:\Users\o123005\frism\prism-insight`

### 존재하는 파일/설정(확인됨)
- `docker-compose.yml` (Docker 기반 운영 가능)
- `mcp_agent.config.yaml.example`
- `mcp_agent.secrets.yaml.example`
- `.env.example`
- `trading/config/kis_devlp.yaml.example`

### 아직 생성되지 않은 파일/디렉터리(확인됨)
- `.env`
- `mcp_agent.config.yaml`
- `mcp_agent.secrets.yaml`
- `trading/config/kis_devlp.yaml`
- `stock_tracking_db.sqlite`
- `reports/`, `pdf_reports/`, `telegram_messages/`

> 위 항목들은 **실행 시 자동 생성되거나** 직접 만들어야 합니다.

## 2) 앞서 제안한 운영/배포 요약 기록
### Ubuntu + Docker 기준
- Docker 사용 가능
- 텔레그램은 기본 미사용(필요 시 활성화 가능)
- 실행은 컨테이너 내부에서 `stock_analysis_orchestrator.py` 수행
- 자동 실행이 목표가 아니라면 스케줄링은 필수 아님

### 스케줄링 관련 의견(요약)
- **수동 실행도 가능**
- **자동으로 매일 돌리려면** 크론/스케줄러 필요

### 안전 운영 권장안(요약)
- 초기에는 `auto_trading: false`, `default_mode: demo` 상태로 검증 후 전환

## 3) 다음 작업 체크리스트(권장)
1. `.env` 생성 (`.env.example` 복사 후 값 입력)
2. `mcp_agent.config.yaml` 생성 (`mcp_agent.config.yaml.example` 복사 후 수정)
3. `mcp_agent.secrets.yaml` 생성 (`mcp_agent.secrets.yaml.example` 복사 후 API 키 입력)
4. `trading/config/kis_devlp.yaml` 생성 (KIS API 정보 입력)
5. Docker 실행
   - `docker compose up -d --build`
6. 분석 실행(텔레그램 미사용)
   - `docker exec -it prism-insight-container python stock_analysis_orchestrator.py --mode morning --no-telegram`
7. 필요 시 스케줄링(크론/시스템 서비스)

---
파일 생성/내용 입력은 다음 문서 참고
- `docs/analysis/deployment_and_keys_ko.md`
- `docs/analysis/mcp_agent_config_ko.md`
