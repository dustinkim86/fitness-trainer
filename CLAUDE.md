# Claude Code Global Workflow Guide

## 플러그인 구성

이 환경에는 다음 플러그인이 설치되어 있다:
- **superpowers** — 브레인스토밍, TDD, 디버깅 스킬 (항상 백그라운드 활성)
- **compound-engineering** — brainstorm → plan → work → review → compound 루프
- **claude-code-harness** — 가드레일 엔진, Plans.md 기반 구현/릴리즈
- **dev-workflows** — spec-driven 백엔드 파이프라인
- **dev-workflows-frontend** — spec-driven 프론트엔드/풀스택 파이프라인
- **frontend-design** — UI 슬롭 방지, aesthetic-first 코딩 (프론트 작업 시 자동 발동)
- **ui-ux-pro-max** — 디자인 DB 기반 팔레트/폰트/UX 가이드 (자동 발동)
- **codex** (OpenAI) — 코드 리뷰 및 작업 위임

---

## 하네스 엔지니어링 워크플로

사용자가 "프로젝트 시작", "새 기능 만들자", "brainstorming 시작" 등의 말을 꺼내면
아래 단계를 순서대로 안내한다. 각 단계가 끝났다고 판단되면 다음 단계를 자동으로 제안한다.

### Phase 0 — 아이디어 · 기획
**목적**: 요구사항 구체화, Requirements.md 생성
**주요 플러그인**: compound-engineering, superpowers
**명령어**:
```
/ce:brainstorm "만들고 싶은 것 설명"   # 대화형 Q&A로 스펙 정제
/ce:ideate                              # 코드베이스 분석 후 개선 아이디어 제안
/brainstorm "아이디어 초안"             # superpowers: 병렬 서브에이전트 분석
```
**완료 신호**: Requirements.md 또는 명확한 스펙 문서가 생성됨
**다음 단계 제안 멘트**: "요구사항이 정리됐습니다. Phase 1(기술 설계)로 넘어갈게요. `/harness-setup`으로 프로젝트를 초기화하시겠어요?"

---

### Phase 1 — 프로젝트 초기화 · 기술 설계
**목적**: 가드레일 엔진 활성화, Plans.md / 아키텍처 설계
**주요 플러그인**: claude-code-harness, compound-engineering, dev-workflows
**명령어**:
```
/harness-setup                          # 가드레일 엔진(R01-R13) 초기화 — 처음 한 번만
/ce:plan "요구사항 문서 경로 또는 설명" # 기술 플랜 생성
/harness-plan                           # Plans.md 생성 (수락 기준 포함)
/recipe-implement "기능 설명"           # dev-workflows: 분석→설계→플랜 파이프라인
```
**완료 신호**: Plans.md 또는 Design Doc이 생성됨
**다음 단계 제안 멘트**: "기술 플랜이 완성됐습니다. 프론트엔드가 포함돼 있나요? 있으면 Phase 2(UI 설계)로, 없으면 바로 Phase 3(구현)으로 갑니다."

---

### Phase 2 — UI/UX 설계 (프론트엔드 포함 시)
**목적**: UI 스펙 확정, 디자인 시스템 선택
**주요 플러그인**: dev-workflows-frontend, frontend-design, ui-ux-pro-max
**명령어**:
```
/recipe-front-design "기능 설명"        # UI spec → 프론트 구현 파이프라인
/frontend-design                        # 수동 발동: aesthetic direction 먼저 결정
# ui-ux-pro-max는 UI 요청 시 자동 발동 (명시 호출 불필요)
```
**완료 신호**: UI Specification 문서 또는 컴포넌트 설계가 완료됨
**다음 단계 제안 멘트**: "UI 스펙이 확정됐습니다. Phase 3(구현)으로 넘어갑니다."

---

### Phase 3 — 구현
**목적**: 실제 코드 작성, 가드레일 보호 하에 실행
**주요 플러그인**: claude-code-harness, dev-workflows, superpowers
**명령어**:
```
/harness-work                           # Plans.md 기반 자동 구현
/harness-work --parallel 5             # 독립 태스크 병렬 처리 (숫자는 조정 가능)
/recipe-build                           # dev-workflows: 기존 플랜 실행
/recipe-fullstack-build                 # 풀스택: 백+프론트 동시 실행
# superpowers TDD 스킬은 테스트 작성 시 자동 발동
```
**완료 신호**: 구현 완료, 테스트 통과
**다음 단계 제안 멘트**: "구현이 완료됐습니다. Phase 4(리뷰)로 넘어갑니다. Codex로 2차 리뷰도 추가할까요?"

---

### Phase 4 — 리뷰 · 검증
**목적**: 코드 품질 검증, 다중 모델 리뷰, 지식 축적
**주요 플러그인**: compound-engineering, superpowers, codex
**명령어**:
```
/request-code-review                    # superpowers: 보안/구조/테스트 관점 리뷰
/ce:review                              # compound: 사이클 단위 리뷰
/codex:review                           # Codex(GPT): 다른 모델 시각의 리뷰
/codex:review --base main              # main 대비 변경 사항만 리뷰
/codex:adversarial-review               # 보안·레이스컨디션 집중 공격 리뷰
/ce:compound                            # 이번 사이클 교훈 축적 → 다음 플랜 반영
```
**완료 신호**: 리뷰 이슈가 해결되고 compound 문서가 업데이트됨
**다음 단계 제안 멘트**: "리뷰가 완료됐습니다. Phase 5(릴리즈)로 넘어갈게요."

---

### Phase 5 — 릴리즈
**목적**: CHANGELOG, 태그, 릴리즈 패키징
**주요 플러그인**: claude-code-harness
**명령어**:
```
/harness-release                        # Plans.md 기준 완료 확인 후 릴리즈 패키징
```

---

## Codex 위임 (언제든 사용 가능)

Claude가 막히거나 두 번째 의견이 필요할 때:
```
/codex:rescue "태스크 설명"             # Codex에 작업 위임
/codex:rescue --background             # 백그라운드 실행
/codex:status                           # 진행 상태 확인
/codex:result                           # 결과 가져오기
/codex:cancel                           # 작업 취소
```

---

## 행동 규칙

1. 사용자가 "시작", "brainstorming", "새 프로젝트", "새 기능" 등을 언급하면 **Phase 0부터 안내**한다.
2. 각 Phase가 끝나면 **다음 Phase를 자동 제안**하고 실행할 명령어를 명시한다.
3. 프론트엔드 작업이 포함된 경우 **Phase 2를 추가**한다. 백엔드만이면 Phase 2를 건너뛴다.
4. 구현 중 Claude가 같은 문제를 2회 이상 반복하면 **`/codex:rescue` 사용을 제안**한다.
5. 리뷰 단계에서 보안/레이스컨디션이 의심되면 **`/codex:adversarial-review` 사용을 제안**한다.
6. 각 명령어 제안 시 **한 줄 설명을 함께** 제공한다.
7. 사용자가 단계를 건너뛰고 싶다고 하면 **그대로 따른다**. 강요하지 않는다.
