# PRD v1.5.1: StockPilot — AI 기반 의사결정 품질 개선 도구

**버전:** v1.5.1 (통합본 — v1.5 Lean + v1.5.1 서비스 정체성 재정의 반영)
**작성일:** Day 1
**상태:** 기획 완료, Day 2 진입 준비
**총 작업 시간:** 약 60~75h

---

## 1. 프로젝트 개요

### 정의
> **시니어 애널리스트 수준 체크리스트 자동화 + 편향 없는 2차 관점 + 본인 의사결정 패턴 학습 도구.**
>
> **주가 예측 시스템이 아니며, 의사결정 품질 개선이 목적.**

### Product Owner
1인 (PM 취업 준비, 금융·AI 도메인 지망)

### 듀얼 목적
- **Layer 1 — Personal OS:** 본인 실사용 MVP
- **Layer 2 — PM Learning:** 의사결정 프레임 검증 케이스

### 핵심 명제
> 주가 예측은 불가능하다는 전제에서 출발. **통제 가능한 것은 '의사결정 품질'뿐이며, 이 시스템은 그 품질을 측정·개선하는 프레임이다.**

### 3가지 가치

**① 체크리스트 수행자 (예측자 X)**
- "삼성전자 오를까?"는 구조적으로 답 불가
- "삼성전자 재무·공시에서 놓친 시그널 있나?"는 답 가능
- AI는 애널리스트가 수동으로 돌리는 "판단 전 확인해야 할 50가지"를 매일 자동 수행

**② 감정 없는 2차 관점**
- 인간은 FOMO·확증편향·손실회피에 갇힘
- AI는 편향 없이 숫자만 봄 (다른 종류의 한계지만 상호 보완)
- 본인이 놓쳤거나 애써 무시한 리스크를 상기

**③ 본인을 기록하는 거울**
- `events` 테이블에 매일 축적:
  - 어떤 AI confidence에서 매수했는가
  - 어떤 리스크를 무시했는가
  - T+3 후회 점수와 실제 수익률의 관계
- 6개월 뒤 본인 의사결정 패턴 데이터화
- **이 데이터가 시장 평균 수익보다 큰 가치** (본인 약점 교정)

### Lean 원칙
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

**MVP 범위 선언:** Archetype C(통제지향)만 정조준. HITL 구조가 핵심인 이유는 C 세그먼트의 통제감 요구에 정확히 대응하기 때문.

---

## 3. 솔루션 철학

> **"AI가 체크리스트, 사람이 판단, 시스템이 기록."**

세 역할의 명확한 분리:
- **AI (Claude Sonnet 4.6):** 재무·공시 **체크리스트 수행** + 초보자 번역 (예측 X, 편향 없는 재해석 O)
- **사람:** 블랙스완·시장 심리·개인 위험 감내도 기반 **최종 판단**
- **시스템 (n8n + Supabase):** 안전 검증·주문 + **모든 의사결정 이벤트 기록**

### AI가 대체하는 것 / 대체하지 않는 것

| AI 영역 | 인간 영역 |
|---|---|
| 재무제표 행간 읽기 | 블랙스완 이벤트 인식 |
| 공시 원문 해석 | 시장 심리·단기 추세 |
| 운전자본·현금흐름 분석 | 미공개 정보·인사이더 판단 |
| 밸류에이션 동종 비교 | 본인 위험 감내도 |
| 숨은 리스크 목록화 | 최종 매수·매도 결정 |

**= HITL이 존재하는 본질적 이유.** 두 영역은 대체 관계가 아닌 보완 관계.

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
| 보조 | Antigravity, Obsidian |
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

### 지표 3층 구조 (참고)
- **과정 지표 (1층):** 이벤트 7종 — 매일 행동
- **품질 지표 (2층):** Decision Quality Lift·False Positive Cost — 시스템 평가
- **결과 지표 (3층):** 수익률·Sharpe Ratio — **참고용만** (소표본 통계 무의미)

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
| 3 | 데이터 수집 파이프라인 (DART PDF + 한투) | 4h |
| 4 | 스크리닝 + 적재 검증 | 5h |
| 5 | AI 분석 (페르소나·tool_use·caching) | 6h |
| 6 | HITL + 안전장치 14종 + 손절 2단계 + Kill Switch | 8h |
| 7 | 단일 지정가 주문 + 체결 폴링 + 5분 재평가 | 4h |
| 8~9 | 모의 정규장 3~5영업일 + A/B 1개 | 8h |
| 10 | Notion 표 대시보드 + AI 기여도 분해표 + 면접 답변서 | 4h |

**총: 60~75h**

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
├── docs/PRD_v1_5_1.md
├── docs/ADR/
├── src/workflows/  (n8n JSON)
├── src/sql/
├── .env.example
└── .gitignore
```

### Notion (메인 허브)
- 프로젝트 한 줄 + disclaimer (**"주가 예측 시스템 아님"** 명시)
- PRD 이력 (v1.0~v1.5.1)
- Hypothesis Table, Experiment Log
- AI 기여도 분해표
- KPI 대시보드 (SQL 결과 표)
- 면접 답변서

### 가드레일 3종
시크릿 절대 X · 규제 표현 disclaimer · 이해상충 점검

---

## 15. 리스크 및 한계

- **LLM 환각:** tool_use 스키마 강제 + `<untrusted_input>` 태그, 완전 제거 불가
- **API 장애:** Healthchecks.io Dead Man's Switch
- **백테스팅 부재:** Forward Testing(10영업일 모의) + 안전장치 14종
- **규제:** 본인 1인은 합법. 멀티유저 + 자동체결 = 투자일임업(자본금 10억)
- **n8n Starter 한도:** Pro 또는 셀프호스팅
- **LLM 비용:** Anthropic 월 $50 하드 리밋, prompt caching 활용
- **1인 의존:** Kill Switch (`/pause`)
- **공개 채널 운영:** 주간 자동 업데이트 워크플로우

---

## 16. 확장 로드맵

### Phase 1 — MVP Lean (Day 0~10) [현재]

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
| 옵션 | 조건 |
|---|---|
| A. B2C 정보 제공 | 자동 체결 제거 + 유사투자자문업 신고 |
| B. B2B 리서치 자동화 | 투자자문사·운용사 대상, 라이선스 불필요 |
| C. 투자일임업 라이선스 | **배제** (자본금 10억, 1인 불가) |

---

## 17. 프로젝트 관리

- Obsidian 단일 도구
- 운영 알림: 텔레그램 + Healthchecks.io
- ADR 형식 의사결정 기록
- 프롬프트 버전 히스토리 관리
- Day 10 케이스 스터디 문서화

---

## 18. 면접 답변 템플릿

**Q. 결국 이 시스템으로 수익 예측 되나요?**

A: "아니요. **주가 예측은 구조적으로 불가능**하다고 봅니다. 효율적 시장 가설·블랙스완·변수 무한 문제 때문입니다. 그래서 이 프로젝트를 '예측 시스템'이 아닌 **'의사결정 품질 개선 시스템'**으로 정의했습니다. AI는 제가 혼자 놓쳤을 재무 시그널·공시 맥락을 매일 자동 해석하고, 저는 그 위에서 감정·편향 없이 판단합니다. 핵심 측정 지표도 수익률이 아닌 Decision Quality Lift로 설정했습니다. **시스템의 목적을 현실적으로 정의하는 것 자체가 PM의 기본**이라고 생각합니다."

---

## Change Log (v1.0 → v1.5.1)

| 버전 | 핵심 변경 |
|---|---|
| v1.0 | 초안 |
| v1.1 | 3개 AI 리뷰 통합 |
| v1.2 | DART PDF + 거시 자동 수집 |
| v1.3 | 이벤트·A/B·AARRR·공개 채널 |
| v1.4 | 실전 사례 8종 흡수 (안전장치 9→14) |
| v1.5 | ROI 최적화 — 6개 이연·6개 축소, 일정 -30% |
| **v1.5.1** | **서비스 정체성 재정의 — "예측자"에서 "체크리스트+2차 관점+거울"로. "주가 예측 시스템 아님" 명시. AI/인간 영역 분리표·지표 3층 구조·면접 답변 템플릿 추가** |

---

**문서 종료. 다음 갱신은 Day 5 이후 실제 Claude 응답 품질 확인 후 v1.5.2로.**
