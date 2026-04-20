# PRD v1.5 (Lean): StockPilot — ROI 최적화 통합본

**버전:** v1.5 Lean (실사용 + 포트폴리오 ROI 최적화)
**작성일:** Day 0
**상태:** 기획 완료, Day 1 진입
**총 작업 시간:** 약 60~75h (v1.4 대비 -30%)

---

## 변경 요약 (v1.4 → v1.5)

### 들어낸 항목 (Phase 2 이연) — 18h 절약
- 3분할 매수 → 단일 지정가
- 거시 보고서 자동 수집 (한은·KDI·KIET) → Claude 일반지식 보완
- Tableau 대시보드 → Notion 표로 대체
- Lovable 대시보드 → 텔레그램으로 충분

### 축소한 항목 — 10h 절약
- Chaos 체크리스트 5종 → 2종 (토큰 만료 + 휴장일)
- 3단계 손절 → 2단계 (-7% 알림 + -10% 자동)
- A/B 실험 2개 → 1개 (프롬프트만)
- Cowork 주간 복기 자동화 → 수동 작성
- Trust Calibration 차등 금액 → confidence 라벨만
- Kill Switch 4개 → 2개 (/pause + /resume)

### 유지한 핵심
- 시니어 애널리스트 페르소나 + tool_use
- DART PDF 자동 수집
- 텔레그램 HITL
- events 테이블 + 7개 이벤트
- Decision Quality Lift + AARRR
- GitHub + Notion 공개

---

## 1. 프로젝트 개요

### 정의
시니어 애널리스트 수준 AI 분석 + HITL 승인 + 자동 체결 구조의 1인 사용 국내 주식 투자 보조 도구.

### Product Owner
1인 (PM 취업 준비, 금융·AI 도메인 지망)

### 듀얼 목적
- **Layer 1 — Personal OS:** 본인 실사용 MVP
- **Layer 2 — PM Learning:** 의사결정 프레임 검증 케이스

### 핵심 명제
> AI 출력 품질을 어떻게 통제했고 어떤 학습 루프를 설계했는가가 가치의 본질.

### Lean 원칙 (v1.5 신규)
- ROI 낮은 항목은 Phase 2 이연
- "만들기 위한 만들기" 금지
- 면접 서사 손상 없는 범위에서 최대 단축

---

## 2. Archetype

| 코드 | 이름 | MVP 대상 |
|---|---|---|
| A | 분석 피로형 초보 | ❌ |
| B | 정보 과잉 관망형 | ❌ |
| **C** | **통제지향 자동화 희망형** | ✅ |

---

## 3. 솔루션 철학
"AI가 분석·번역, 사람이 결정, 시스템이 체결"

---

## 4. MVP 범위

### 포함
| 기능 | 실사용 | 포트폴리오 |
|---|---|---|
| 관심 종목 12개 고정 | ● | ● |
| 룰 스크리닝 (PER·ROE·모멘텀) | ● | ● |
| Claude 시니어 페르소나 분석 | ● | ●● |
| 텔레그램 HITL | ●● | ●● |
| 한투 단일 지정가 매수 (모의→실투) | ●● | ● |
| 안전장치 14종 (9 + 손절 2단계 + Chaos 2 + Kill Switch) | ●● | ● |
| events 테이블 + 7개 이벤트 | ● | ●● |
| Notion KPI 대시보드 (SQL 결과 표) | ● | ●● |
| 수동 주간 복기 (Obsidian) | ● | ●● |

### 제외 (Phase 2)
3분할 매수, 거시 보고서 자동 수집, Tableau, Lovable, Cowork 자동화, A/B 실험 2번째, 전 종목 스크리닝

---

## 5. 기술 스택

| 레이어 | 도구 |
|---|---|
| 오케스트레이션 | n8n Pro/셀프호스팅 |
| LLM | Claude Sonnet 4.6 (`tool_use` + extended thinking + prompt caching) |
| 데이터 | DART OpenAPI + 한국투자증권 OpenAPI |
| DB + 저장 | Supabase (PostgreSQL + Storage) |
| UI | 텔레그램 Bot |
| 모니터링 | Healthchecks.io |
| 분석 | Supabase SQL → Notion 표 |
| 보조 | Claude Code, Cursor, Obsidian |
| 공개 | GitHub, Notion |

**배제:** Tableau, Lovable, Dify, Coze, Make.com, 거시 보고서 자동 크롤링, 벡터DB

---

## 6. 데이터 플로우

### 일일 자동 실행
1. **08:00** 한투 토큰 발급 (만료 자동 재발급)
2. **08:05** 영업일 체크 (휴장일 조기 종료)
3. **08:30** DART 공시 메타데이터 + 첨부 PDF 다운로드 + 한투 시세·재무 → Supabase
4. **08:35** 룰 스크리닝
5. **08:40** Claude 분석 (PDF 첨부 + tool_use + extended thinking + caching)
6. **08:50** 텔레그램 브리핑 (3문장 + confidence + 우선순위 라벨)
7. **08:55** Healthchecks.io 하트비트
8. **09:00~15:20** HITL 승인 → 안전장치 → 단일 지정가 매수 → 체결 폴링
9. **09:00~15:30 (5분)** 포트폴리오 재평가 → -7% 알림 → -10% 자동 매도
10. **15:30** 손익 업데이트

### 주간 (수동)
금요일 장 마감 후 Obsidian에 수동 주간 복기 작성 (10~15분)

---

## 7. AI 설계

### 시니어 sell-side 애널리스트 페르소나 (시스템 프롬프트)
```
15년차 sell-side 시니어 애널리스트 관점.
프레임:
1. 매출·이익의 질
2. 운전자본 변화
3. 현금흐름 vs 회계이익 괴리
4. 밸류에이션 (동종업계 대비)
5. 촉매 + 리스크
6. 거시 맥락
근거 없는 주장 금지. 입력 데이터 숫자 인용.
```

### 컨텍스트 주입
- DART 분기·연간 보고서 PDF 직접 첨부
- 감사보고서 PDF (연 1회)
- 한투 시세·재무 (60일~8분기)
- 거시 데이터: Claude 일반지식 활용 (자동 수집 X)

### tool_use JSON 스키마
```json
{
  "ticker": "...",
  "earnings_quality": { "score": 0~5, "reasoning": "..." },
  "valuation": { "per": ..., "verdict": "..." },
  "catalysts": [...],
  "risks_ranked": [...],
  "priority": "긴급|검토|참고",
  "confidence": 0.0~1.0,
  "plain_korean_summary": "3문장",
  "decision_button": "매수|매도|관망"
}
```

### Trust Calibration (단순화)
- confidence 라벨만 표시 (금액 차등 X)
- Phase 2에 데이터 쌓이면 차등 적용

---

## 8. 안전장치 14종

### 시스템 9종
1. 시간 검증 (09:00~15:20)
2. 멱등성 (Supabase UNIQUE)
3. 1회 5만원 상한
4. 일일 누적 15만원 상한
5. 일일 손실 -3% 차단
6. 토큰 유효성 재확인
7. 체결 상태 확정 (폴링)
8. 지정가 매수 (현재가 +0.3%)
9. 자동 손절 (-10% + 5분 미응답 시 시장가)

### Chaos 2종
10. 토큰 만료 조기 감지 (10분 전 재발급)
11. 휴장일 자동 인지 (한투 영업일 API)

### 손절 2단계
- -7%: 텔레그램 긴급 알림 + 매도 승인 버튼
- -10% + 5분 미응답: 자동 시장가 매도

### Kill Switch 2개
- `/pause` — 오늘 중단
- `/resume` — 재개

---

## 9. 이벤트 7종

`events` 테이블 단일 수집.

| 이벤트명 | 핵심 프로퍼티 | 답할 질문 |
|---|---|---|
| `briefing_sent` | ticker, confidence, prompt_version | 커버리지 |
| `briefing_opened` | ticker, latency_sec | 매력도 |
| `expand_clicked` | ticker, section | 어느 근거가 중요한가 |
| `decision_made` | ticker, decision, latency_sec | 결정 패턴 |
| `order_executed` | ticker, price, slippage | 체결 성공률 |
| `guardrail_triggered` | guardrail_type | 안전장치 적중 |
| `post_decision_regret` | ticker, regret_score, return_pct | False Positive Cost |

---

## 10. KPI + AARRR

### NSM
**Decision Quality Lift** — baseline 대비 결정 품질 개선

### AARRR
| 단계 | 지표 |
|---|---|
| Activation | 첫 승인까지 시간 |
| Retention | 7일 브리핑 확인율 |
| Engagement | 펼쳐보기 클릭률 |
| Revenue (Proxy) | Decision Quality Lift |

### 보조 KPI 4종
Explanation Utility Score · False Positive Cost · Guardrail Breach Rate · 가동률

---

## 11. A/B 실험 1개

### 실험 1: 프롬프트 V1 vs V2
- 가설: 시니어 페르소나(V2)가 기본(V1) 대비 Explanation Utility +20%
- 설계: Day 8 V1, Day 9 V2 (시간 분할)
- 측정: expand_clicked 빈도 + 자가 평가
- 판정: V2가 +15% 이상 → 채택

**(브리핑 포맷 A/B는 Phase 2 이연)**

---

## 12. 작업 일정 (Lean)

| Day | 작업 | 시간 |
|---|---|---|
| 0 | API 신청·Obsidian·종목 12개 ✅ | 3.5h |
| 1 | GitHub·Notion·ADR 5개·Hypothesis·이벤트 설계 | 6h |
| 2 | 인프라 (n8n·Supabase·events 테이블) | 6h |
| 3 | 데이터 수집 파이프라인 (DART PDF + 한투) | 4h ⬇ |
| 4 | 스크리닝 + 적재 검증 | 5h |
| 5 | AI 분석 (페르소나·tool_use·caching) | 6h |
| 6 | HITL + 안전장치 14종 + 손절 2단계 + Kill Switch | 8h ⬇ |
| 7 | 단일 지정가 주문 + 체결 폴링 + 5분 재평가 | 4h ⬇ |
| 8~9 | 모의 정규장 3~5영업일 + A/B 1개 | 8h |
| 10 | Notion 표 대시보드 + AI 기여도 분해표 + 면접 답변서 | 4h ⬇ |

**총: 60~75h (v1.4 대비 -30%)**

---

## 13. PM 산출물

1. Research Memo (본인 baseline 일기 + 2차 자료)
2. Hypothesis Table (H1~H3)
3. Experiment Log
4. Trade-off Note (자동매매 vs HITL, 들어낸 항목 ROI 판단 기록)
5. 면접 답변서 1페이지
6. AI 기여도 분해표

---

## 14. 공개 채널

### GitHub
```
stockpilot/
├── README.md
├── docs/PRD_v1_5.md
├── docs/ADR/
├── src/workflows/  (n8n JSON)
├── src/sql/
├── .env.example
└── .gitignore
```

### Notion (메인 허브)
- 프로젝트 한 줄 + disclaimer
- PRD 이력 (v1.0~v1.5)
- Hypothesis Table, Experiment Log
- AI 기여도 분해표
- KPI 대시보드 (SQL 결과 표)
- 면접 답변서

### 가드레일 3종
시크릿 절대 X · 규제 표현 disclaimer · 이해상충 점검

---

## 15. 리스크
v1.4 동일 (LLM 환각, API 장애, 규제, 비용, 1인 의존)

---

## 16. 확장 로드맵

### Phase 1 — MVP Lean (Day 0~10)
현재 PRD

### Phase 2 — 확장 (MVP + 4주 후)
- 3분할 매수
- 거시 보고서 자동 수집
- Tableau 대시보드
- A/B 실험 2번째 (브리핑 포맷)
- Cowork 주간 복기 자동화
- Trust Calibration 차등 금액
- Auto 모드 토글 (조건부)
- 룰 스크리닝 백테스트
- Chaos 체크리스트 확장 (5종)
- 손절 3단계 복귀

### Phase 3 — 고도화 (3개월+)
실시간 뉴스 RSS, 다전략 병렬, Forward Test 누적

### Phase 4 — 서비스화 판단
B2C 정보제공 / B2B 리서치 / 일임업 (배제) 중 단일 선택

---

## 17. 프로젝트 관리
Obsidian 단일 + 텔레그램·Healthchecks.io + ADR + 프롬프트 버전

---

## Change Log (v1.0 → v1.5)

| 버전 | 핵심 변경 |
|---|---|
| v1.0 | 초안 |
| v1.1 | 3개 AI 리뷰 통합 |
| v1.2 | DART PDF + 거시 자동 수집 |
| v1.3 | 이벤트·A/B·AARRR·공개 채널 |
| v1.4 | 실전 사례 8종 흡수 (안전장치 9→14) |
| **v1.5** | **ROI 최적화 — 6개 이연·6개 축소, 일정 -30%** |

---

## v1.5의 PM 서사 강화 포인트

이번 ROI 재검토 자체가 강력한 면접 소재:
- "PRD를 5번 반복 후, 6번째에 들어내기 결정"
- "ROI 낮은 항목 명시적 분리 → Phase 2 이연 근거 ADR 작성"
- "**'만들 줄 아는 PM'이 아니라 '안 만들 줄 아는 PM'**"

면접 답변 예시:
> "처음에는 안전장치 14종, A/B 2개, Tableau, 거시 자동수집 등 풍부하게 설계했지만, 실사용 가치와 포트폴리오 가치를 매트릭스로 평가해 6개를 Phase 2로 이연했습니다. ROI 판단이 PM의 핵심 역량이라고 생각합니다."

---

**문서 종료. 다음 갱신은 Day 1 ADR 5개 작성 후 v1.5.1로.**
