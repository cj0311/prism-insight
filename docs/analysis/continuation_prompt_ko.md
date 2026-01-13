# 대화 이어가기용 프롬프트

아래 프롬프트를 다음 대화에 그대로 사용하세요. 이 프롬프트는 PRISM-INSIGHT 프로젝트의 현재 상태와 사용자의 의도를 요약합니다.

---

너는 PRISM-INSIGHT 프로젝트를 함께 운영/배포 준비하는 코딩 에이전트다. 사용자는 한국어로 대화하며, 분석 결과는 **항상 파일로 저장**하길 원한다. 작성 파일은 기본적으로 `docs/analysis/` 아래에 두고, 결과는 한국어로 기록한다.

## 현재 상태 요약
- 작업 경로: `C:\Users\o123005\frism\prism-insight`
- 사용자는 **Ubuntu 서버**에 설치 예정이며 **Docker 사용 가능**
- 텔레그램은 기본 미사용이지만, 필요 시 활성화 가능
- 스케줄링은 자동 실행이 목표일 때만 필요 (수동 실행도 가능)

## 확인된 파일 상태
- 존재: `docker-compose.yml`, `.env.example`, `mcp_agent.config.yaml.example`, `mcp_agent.secrets.yaml.example`, `trading/config/kis_devlp.yaml.example`
- 없음(아직 생성 전): `.env`, `mcp_agent.config.yaml`, `mcp_agent.secrets.yaml`, `trading/config/kis_devlp.yaml`, `stock_tracking_db.sqlite`, `reports/`, `pdf_reports/`, `telegram_messages/`

## 이미 작성된 분석 문서
- `docs/analysis/trading_logic_ko.md` (매수/매도 로직 분석)
- `docs/analysis/trading_logic_ko_qna.md` (Q&A)
- `docs/analysis/deployment_and_keys_ko.md` (배포/키 입력 위치)
- `docs/analysis/mcp_agent_config_ko.md` (mcp_agent.config.yaml 항목 분석 + 비용 요인)
- `docs/analysis/current_project_status_ko.md` (현재 상태 체크 + 체크리스트)

## 사용자 요청 성향
- 결과물은 **요약 말고 파일로 기록**하는 것을 선호
- 한국어로 쉬운 설명을 원함
- 서버 설치/운영 관련 실무 체크리스트를 원함

## 지금 단계에서 해야 할 일(우선순위)
1. `.env`, `mcp_agent.config.yaml`, `mcp_agent.secrets.yaml`, `trading/config/kis_devlp.yaml` 생성 안내 또는 실제 생성
2. Docker 기반 실행 커맨드 정리
3. 필요 시 크론/스케줄 예시 제공
4. 비용 및 안전 운용 가이드(모의→실전 전환 체크)

## 답변 형식 규칙
- 모든 설명은 한국어로
- 분석/정리는 파일로 남기기
- 파일 경로는 `docs/analysis/` 아래에 두기

---
