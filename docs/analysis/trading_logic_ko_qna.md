# 추가 질문 답변: 리포트 PDF, 보유 슬롯, 레거시 매도 규칙

아래 답변은 현재 저장소 코드 기준(2026-01-13)으로 정리했습니다.

## 1) 주식 분석 리포트(PDF) 은 어디서 얻는 구조야?
**요약**: 외부에서 PDF를 “가져오는” 게 아니라, 내부에서 **Markdown 리포트 생성 → PDF 변환**으로 만들어집니다.

**흐름**
1. **리포트 생성(마크다운)**
   - `stock_analysis_orchestrator.py` → `generate_reports()`
   - 내부에서 `cores/main.py`의 `analyze_stock()` 호출
   - 결과를 `reports/` 폴더에 `*.md`로 저장

2. **PDF 변환**
   - `stock_analysis_orchestrator.py` → `convert_to_pdf()`
   - `pdf_converter.py`의 `markdown_to_pdf()` 사용
   - 결과를 `pdf_reports/` 폴더에 `*.pdf`로 저장

3. **PDF 사용처**
   - 텔레그램 전송: `stock_analysis_orchestrator.py` → `send_telegram_messages()`
   - 매매 판단 입력: `stock_tracking_agent.py` → `analyze_report()`
     - `pdf_converter.pdf_to_markdown_text()`로 PDF 내용을 텍스트로 추출 후 LLM 입력

즉, PDF는 **내부 생성물**이며, 핵심 생성 라인은 아래입니다.
- `stock_analysis_orchestrator.py` (리포트 생성 및 PDF 변환)
- `pdf_converter.py` (Markdown → PDF 변환)

## 2) 자리(보유 슬롯) 같은 설정파일은 어디서 수정해?
**설정 파일이 아니라 코드 상수로 고정**되어 있습니다.

**수정 위치(핵심)**
- `stock_tracking_agent.py`
  - `MAX_SLOTS = 10`  (최대 보유 슬롯)
  - `MAX_SAME_SECTOR = 3` (동일 섹터 보유 제한)
  - `SECTOR_CONCENTRATION_RATIO = 0.3` (섹터 집중도 제한)

**연동 주의사항**
- `stock_tracking_enhanced_agent.py`는 `StockTrackingAgent`를 상속하므로 동일 값 사용
- LLM 프롬프트에도 “max 10 slots” 표현이 들어있음
  - `cores/agents/trading_agents.py`에 `max 10 slots` 관련 문구가 있으니 **수정 시 함께 변경** 권장

요약: **슬롯 수 변경은 코드 수정**이며, 별도 YAML/ENV 설정은 없습니다.

## 3) 6-2. 대체(legacy) 규칙 특징은 현재 안쓰고 AI 매도 판단인거지?
**결론**: AI 매도 판단이 “우선”이고, **레거시는 실패 시 fallback**입니다. 즉, 레거시도 여전히 쓰입니다.

**근거**
- `stock_tracking_enhanced_agent.py` → `_analyze_sell_decision()`
  - AI 응답이 비었거나 JSON 파싱이 실패하면 `_fallback_sell_decision()`으로 전환
  - 따라서 “항상 AI”가 아니라, **AI 실패 시 레거시 사용**

**추가로 알아둘 점**
- `stock_tracking_agent.py`는 AI 매도 판단이 없고 **레거시 규칙만 사용**
- 실제 파이프라인에서는 `stock_analysis_orchestrator.py`가 `EnhancedStockTrackingAgent`를 사용하므로 **기본은 AI → 실패 시 레거시** 구조입니다.

---
참고 파일
- `stock_analysis_orchestrator.py`
- `pdf_converter.py`
- `stock_tracking_agent.py`
- `stock_tracking_enhanced_agent.py`
- `cores/agents/trading_agents.py`
