# Project Audit Report Template

Use this template to generate `docs/PROJECT_AUDIT.md`.
Fill in sections based on diagnostic checklist results.

---

```markdown
# 🔍 프로젝트 진단 리포트

> 진단일: YYYY-MM-DD
> 대상: [project name / repo URL]
> 기술 스택: [detected tech stack]
> 커밋 수: [total commits]

---

## 📊 종합 점수

| 카테고리 | 점수 | 상태 |
|----------|------|------|
| A. Claude Code 인프라 | N/11 | [🟢/🟡/🔴] |
| B. 스킬 & 에이전트 | N/8 | [🟢/🟡/🔴] |
| C. 명세 문서 | N/7 | [🟢/🟡/🔴] |
| D. 운영 문서 | N/5 | [🟢/🟡/🔴] |
| E. 컨텍스트 효율 | N/5 | [🟢/🟡/🔴] |
| F. 테스트 & 품질 | N/5 | [🟢/🟡/🔴] |
| **전체** | **N/41** | **[등급]** |

등급: 🟢 80%+ | 🟡 50-79% | 🔴 < 50%

---

## 🏗️ A. Claude Code 인프라

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| A1 | CLAUDE.md 존재 (< 200줄) | [✅/⚠️/❌] | [details] |
| A2 | 빌드/테스트 명령어 포함 | [✅/⚠️/❌] | [details] |
| A3 | 아키텍처 개요 포함 | [✅/⚠️/❌] | [details] |
| A4 | 코딩 컨벤션 포함 | [✅/⚠️/❌] | [details] |
| A5 | 세션 시작/종료 프로토콜 | [✅/⚠️/❌] | [details] |
| A6 | 컨텍스트 관리 규칙 | [✅/⚠️/❌] | [details] |
| A7 | 스킬 트리거 테이블 | [✅/⚠️/❌] | [details] |
| A8 | settings.json (훅 설정) | [✅/⚠️/❌] | [details] |
| A9 | SessionStart 훅 | [✅/⚠️/❌] | [details] |
| A10 | Stop 훅 | [✅/⚠️/❌] | [details] |
| A11 | settings.local.json 정돈 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안

(For each missing item, include:)

#### [A-N]: [Item Name]

**왜 필요한가**: [1-2 sentence explanation in Korean]

**생성할 파일**: `[exact file path]`

**내용**:
```
[Ready-to-use file content — English for skills/hooks/rules, Korean for business docs]
```

---

## 🤖 B. 스킬 & 에이전트

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| B1 | 계획 수립 스킬 | [✅/⚠️/❌] | [details] |
| B2 | 진행 기록 스킬 | [✅/⚠️/❌] | [details] |
| B3 | 컨텍스트 관리 스킬 | [✅/⚠️/❌] | [details] |
| B4 | 프로젝트 기술스택 매칭 | [✅/⚠️/❌] | [details] |
| B5 | 코드 리뷰 에이전트 | [✅/⚠️/❌] | [details] |
| B6 | 버그 분석 에이전트 | [✅/⚠️/❌] | [details] |
| B7 | 프로젝트별 컨벤션 참조 | [✅/⚠️/❌] | [details] |
| B8 | 슬래시 커맨드 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안
(same structure as above)

---

## 📋 C. 명세 문서

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| C1 | PRD / 요구사항 문서 | [✅/⚠️/❌] | [details] |
| C2 | 기능별 명세서 | [✅/⚠️/❌] | [details] |
| C3 | API 명세서 | [✅/⚠️/❌] | [details] |
| C4 | DB 스키마 명세서 | [✅/⚠️/❌] | [details] |
| C5 | 완료 기준 / 테스트 시나리오 | [✅/⚠️/❌] | [details] |
| C6 | 관련 코드 파일 참조 | [✅/⚠️/❌] | [details] |
| C7 | 명세 크기 < 500줄 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안
(same structure)

---

## 📝 D. 운영 문서

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| D1 | 진행 상황 추적 (progress) | [✅/⚠️/❌] | [details] |
| D2 | 기능/버그 추적 (features-list) | [✅/⚠️/❌] | [details] |
| D3 | 변경 이력 (CHANGELOG) | [✅/⚠️/❌] | [details] |
| D4 | 아키텍처 결정 기록 (ADR) | [✅/⚠️/❌] | [details] |
| D5 | 에이전트 팀 브리핑 문서 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안
(same structure)

---

## ⚡ E. 컨텍스트 효율

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| E1 | 대용량 문서 없음 (> 50KB) | [✅/⚠️/❌] | [list oversized files] |
| E2 | README 외 컨텍스트 소스 | [✅/⚠️/❌] | [details] |
| E3 | 기능 레벨 문서 존재 | [✅/⚠️/❌] | [details] |
| E4 | 대용량 파일 읽기 경고 | [✅/⚠️/❌] | [details] |
| E5 | compact/clear 전략 문서화 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안
(same structure)

---

## 🧪 F. 테스트 & 품질

| # | 항목 | 상태 | 비고 |
|---|------|------|------|
| F1 | 테스트 프레임워크 설정 | [✅/⚠️/❌] | [framework name] |
| F2 | 테스트 파일 존재 | [✅/⚠️/❌] | [count] |
| F3 | E2E 테스트 설정 | [✅/⚠️/❌] | [framework name] |
| F4 | 테스트 명령어 문서화 | [✅/⚠️/❌] | [details] |
| F5 | CI/CD 파이프라인 | [✅/⚠️/❌] | [details] |

### ❌ 누락 항목 상세 및 수정 방안
(same structure)

---

## 🚀 적용 실행 계획

### Phase A: 즉시 (1시간) — 세션 인수인계 + 훅

| # | 작업 | 시간 | 파일 |
|---|------|------|------|
| 1 | [task] | Nmin | [file path] |
| ... | | | |

### Phase B: 다음 날 (1시간) — 스킬 + 에이전트 + 명세

| # | 작업 | 시간 | 파일 |
|---|------|------|------|
| ... | | | |

### Phase C: 점진적 — 기능별 명세 추가

| # | 작업 | 방법 |
|---|------|------|
| ... | | |

---

## 💡 핵심 원칙

1. **한 번에 다 하지 마세요** — Phase A만으로도 가장 급한 문제 해결
2. **작업하는 기능의 명세부터** — 모든 기능의 명세를 한 번에 쓰지 마세요
3. **같은 수정 3번 이상 시도 금지** — 컨텍스트만 소모, 결과는 같음
4. **문서와 코드는 함께 커밋** — 코드만 커밋하면 명세가 밀린다
```
