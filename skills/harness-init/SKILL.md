---
name: harness-init
description: 프로젝트를 심층 분석해 맞춤형 하네스(CLAUDE.md, 5+ 워크플로우 스킬, 도메인 에이전트, 패턴, 인덱스)를 자동 생성하는 오케스트레이터. "하네스 초기화", "하네스 만들어줘", "하네스 다시 초기화", "harness 다시 만들어줘", "프로젝트 분석해서 설정해줘", "이 프로젝트 Claude 설정해줘", "create harness", "initialize harness", "re-initialize harness", "generate project harness", "하네스 업데이트", "하네스 보완", "스킬만 다시 생성", "에이전트만 다시 생성", "validator만 다시 실행", "패턴 추출해줘", "pattern extract" 요청 시 사용. `.claude/skills/trace.md` 또는 `.claude/skills/analyze-impact.md`가 없으면 자동 트리거.
---

# Harness Initializer (Enhanced) — 팀 모드 오케스트레이터

프로젝트 코드베이스를 심층 분석해 *수정·개발·마이그레이션 작업까지 지원하는* 맞춤형 harness를 자동 생성한다.

기존 harness-new가 만들던 5종 + harness-fin이 추가하는 5종 + 인덱스 + 패턴까지 한 번에 생성.

**실행 모드:** 에이전트 팀 (TaskCreate 의존성 + `_workspace/` 파일 기반 산출물 전달)

**팀 구성 (확장):**
- 필수 파이프라인: analyzer → writer → validator → qa
- 선택 추가: pattern-extractor (writer 직후, 패턴 채우기)

---

## Phase 0: 컨텍스트 확인

### Step 1: 작업 디렉토리 확인
`pwd`로 절대 경로 확보.

### Step 2: 기존 하네스 감지

| 확인 대상 | 의미 |
|----------|------|
| `CLAUDE.md` 존재 + "## 변경 이력" 섹션 | 기존 하네스 있음 |
| `.claude/skills/trace.md` 존재 | 기존 스킬 있음 |
| `.claude/skills/analyze-impact.md` 존재 | harness-fin v1 이상 |
| `.claude/agents/domain-expert.md` | 도메인 에이전트 있음 |
| `_workspace/` 존재 | 이전 산출물 있음 |
| `_workspace/index/*.json` 존재 | 인덱스 있음 (incremental 가능) |

### Step 3: 실행 모드 분기

| 상황 | 모드 | 처리 |
|------|------|------|
| 기존 하네스 없음 | **초기 실행** | 전체 파이프라인 (analyzer init + writer + validator + qa + pattern-extractor) |
| 기존 + "다시"·"새로" | **재초기화** | `.claude/backup/[YYYYMMDD-HHMMSS]/`로 백업 후 전체 실행 (analyzer init 모드) |
| 기존 + "스킬만"·"에이전트만"·"validator만"·"qa만"·"패턴만" | **부분 재실행** | 해당 단계만, 이전 `_workspace/` 산출물 재사용 |
| 기존 + 일반 보완 | **업데이트** | 백업 후 analyzer incremental + 재실행 |
| 코드 변경 후 인덱스만 갱신 | **인덱스 리프레시** | analyzer incremental만 |

백업 절차:
- PowerShell: `Get-Date -Format "yyyyMMdd-HHmmss"`
- 백업 대상: `CLAUDE.md`, `.claude/skills/*.md` (harness-init.md 제외), `.claude/agents/*.md` (공통 에이전트는 제외, 프로젝트 전용만), `.claude/patterns/`
- 백업 위치: `.claude/backup/[YYYYMMDD-HHMMSS]/`

### Step 4: 작업공간 준비

`_workspace/` 디렉토리:
- 초기 실행/재초기화: `_workspace/`를 `_workspace_prev/`로 이동 후 새로 생성
- 부분 재실행: 기존 유지

산출물 파일명:
```
_workspace/01_analyzer_report.md      ← analyzer
_workspace/02_writer_files.md         ← writer
_workspace/03_validator_report.md     ← validator
_workspace/04_qa_report.md            ← qa
_workspace/05_patterns_extracted.md   ← pattern-extractor
_workspace/index/*.json               ← analyzer (인덱스)
```

---

## Phase 1: 공유 작업 계획

`TaskCreate`로 팀원별 작업 + 의존성 설정:

```
T-A (analyzer):          → _workspace/01_analyzer_report.md + _workspace/index/*.json
T-W (writer):            → _workspace/02_writer_files.md           (blockedBy: T-A)
T-V (validator):         → _workspace/03_validator_report.md       (blockedBy: T-W)
T-Q (qa):                → _workspace/04_qa_report.md              (blockedBy: T-V)
T-P (pattern-extractor): → _workspace/05_patterns_extracted.md     (blockedBy: T-W, optional)
```

T-P는 T-W 완료 후 T-V/T-Q와 병렬 실행 가능.

---

## Phase 2: 팀원 실행

### 2-1. analyzer 호출

부분 재실행 + `_workspace/01_analyzer_report.md` 존재 시 스킵.

```
Agent(
  subagent_type="general-purpose",
  description="프로젝트 심층 분석",
  prompt="<analyzer 에이전트 지침에 따라 분석. 프로젝트 루트: [절대경로]. mode: init. 결과: _workspace/01_analyzer_report.md + _workspace/index/*.json>",
  model="opus"
)
```

`.claude/agents/analyzer.md`의 지침 따름. 완료 후 결과 파일 존재 확인.

### 2-2. writer 호출

`_workspace/01_analyzer_report.md` 존재 확인 후:

```
Agent(
  subagent_type="general-purpose",
  description="하네스 파일 생성",
  prompt="<writer 에이전트 지침에 따라 하네스 파일 작성. 프로젝트 루트: [절대경로]. 입력: _workspace/01_analyzer_report.md. 출력: 하네스 파일들 + _workspace/02_writer_files.md>",
  model="opus"
)
```

### 2-3. pattern-extractor 호출 (NEW, 병렬 가능)

writer 완료 후 patterns/ 스켈레톤이 생성되어 있다. pattern-extractor가 본문을 채운다.

```
Agent(
  subagent_type="general-purpose",
  description="패턴 추출",
  prompt="<pattern-extractor 에이전트 지침. 프로젝트 루트: [절대경로]. 입력: .claude/patterns/*.md 스켈레톤 + _workspace/01_analyzer_report.md. 출력: 패턴 파일 본문 + _workspace/05_patterns_extracted.md>",
  model="opus"
)
```

writer의 `02_writer_files.md`에 "patterns 스켈레톤 생성"이 명시된 경우에만 호출.

### 2-4. validator 호출

`_workspace/02_writer_files.md` 확인 후:

```
Agent(
  subagent_type="general-purpose",
  description="하네스 구조 검증",
  prompt="<validator 에이전트 지침. 프로젝트 루트: [절대경로]. 입력: _workspace/01_analyzer_report.md, _workspace/02_writer_files.md, _workspace/index/. 출력: _workspace/03_validator_report.md>",
  model="opus"
)
```

### 2-5. qa 호출 (경계면 교차 비교)

validator 통과 후. **반드시 `general-purpose` 타입.**

```
Agent(
  subagent_type="general-purpose",
  description="경계면 교차 비교 QA",
  prompt="<qa 에이전트 지침. 프로젝트 루트: [절대경로]. 입력: _workspace/01~03 + _workspace/index/. 출력: _workspace/04_qa_report.md>",
  model="opus"
)
```

QA 우회 조건: `_workspace/03_validator_report.md`의 신뢰도 < 50 → qa는 "구조 검증 실패로 미실행" 한 줄만 작성하고 종료.

---

## Phase 3: 결과 종합 및 보고

`_workspace/03_validator_report.md`, `_workspace/04_qa_report.md`, `_workspace/05_patterns_extracted.md`를 읽어 사용자에게 다음 형식으로 보고:

```
하네스 초기화 완료 (harness-fin v1)

생성된 파일:

[Core]
- CLAUDE.md
- .claude/skills/trace.md, scaffolder.md, find-logic.md
- .claude/agents/domain-expert.md
- .claude/patterns/[목록]

[Workflow Skills (NEW)]
- .claude/skills/analyze-impact.md
- .claude/skills/safe-modify.md
- .claude/skills/scaffold-feature.md
- .claude/skills/plan-migration.md          (생성 조건 충족 시)
- .claude/skills/review-sql.md              (DB 사용 시)

[Indexes (NEW)]
- _workspace/index/call_graph.json (노드: N, 엣지: M)
- _workspace/index/symbols.json
- _workspace/index/transactions.json
- _workspace/index/external_io.json
- _workspace/index/sql_usage.json           (DB 사용 시)
- _workspace/index/schema.json              (DB 접속 가능 시)
- _workspace/index/dead_code.json
- _workspace/index/env_branches.json

구조 검증 (validator):
[신뢰도 점수 + 보완 권장 항목]

경계면 교차 비교 (qa):
🔴 HIGH: [개수 + 상세]
🟡 MEDIUM: [...]
🟢 LOW: [...]

패턴 추출 (pattern-extractor):
- 처리한 패턴 파일: N개
- 신뢰도: [HIGH: A, MEDIUM: B, LOW: C]
- 안티패턴 발견: K건

이제 다음 작업이 가능합니다:
  "이 함수 영향도 분석해줘"          → analyze-impact
  "이 변경 안전하게 적용"            → safe-modify
  "[기능] 패턴 따라 만들어줘"        → scaffold-feature
  "Spring Boot로 마이그레이션"       → plan-migration
  "이 SQL 리뷰해줘"                  → review-sql
  "이 코드 뭐하는 거야"              → legacy-decoder (직접 호출)
  "문서 동기화"                      → doc-syncer (직접 호출)

다음 단계:
  git add CLAUDE.md .claude/ && git commit -m "docs: add project harness (harness-fin v1)"

피드백 요청:
결과에서 개선할 부분이 있나요? 워크플로우 스킬 트리거 조정이 필요한가요?
```

HIGH 우선순위 항목이 있으면 사용자에게 명시적 안내. 자동 수정 X.

---

## 에러 핸들링

원칙: **1회 재시도 후 재실패 시 결과 없이 진행하고 보고서에 누락 명시. 상충 데이터는 출처 병기.**

| 상황 | 대응 |
|------|------|
| analyzer가 산출물 미생성 | 1회 재실행. 재실패 시 "분석 실패 — 수동 분석 필요" 보고 후 중단 |
| analyzer가 인덱스 일부만 생성 | writer/validator/qa는 진행, 누락 인덱스에 의존하는 워크플로우 스킬은 "인덱스 누락" WARN |
| writer 일부 파일만 생성 | 누락 목록 보고. validator는 생성된 파일에만 검증. 누락 워크플로우 스킬 명시 |
| pattern-extractor 실패 | patterns/ 는 스켈레톤 상태 유지. "pattern-extractor 재실행 권고" 안내 |
| validator 보안 위험 발견 | 자동 수정 금지. 위치 명시, 사용자 직접 처리 |
| qa DEAD/ORPHAN 발견 | 자동 수정 금지. 우선순위 표시, 사용자 직접 처리 |
| validator 신뢰도 < 50 | qa 스킵. "validator 권고 우선 처리 후 재실행" 안내 |
| 작업 디렉토리 권한 오류 | 즉시 중단, 권한 확인 요청 |
| `_workspace/` 생성 실패 | 1회 재시도. 실패 시 중단 |

상충 데이터: writer가 두 패턴 발견 시 출처 병기, validator/qa가 우선순위 권고 (자동 결정 X).

---

## 팀 통신 프로토콜

이 하네스는 `TeamCreate`/`SendMessage` 도구 없음. 대신:

| 채널 | 도구 | 용도 |
|------|------|------|
| 작업 조율 | `TaskCreate`/`TaskUpdate` | 진행 추적, 의존성 |
| 산출물 전달 | `_workspace/` 파일 | 분석 리포트·생성 파일·검증·인덱스 |

각 에이전트는 자기 `.md`에 명시된 입력 파일을 읽고 출력 파일을 작성. 오케스트레이터는 의존성 순서로 호출하고 산출물 존재 확인.

---

## 테스트 시나리오

### 정상 흐름 (초기 실행)
1. 사용자가 신규 프로젝트에서 "하네스 초기화"
2. Phase 0: 없음 확인, `_workspace/` + `_workspace/index/` 생성
3. Phase 1: TaskCreate로 의존성 체인 (T-A → T-W → T-V → T-Q, T-P는 T-W 직후)
4. Phase 2: 모두 실행, 인덱스 + 패턴 본문 생성
5. Phase 3: 종합 보고

### 부분 재실행: 패턴만 다시
1. 사용자 "패턴만 다시 추출"
2. Phase 0: 부분 모드, 기존 `_workspace/01, 02` 보존
3. Phase 2-3 (pattern-extractor)만 실행
4. Phase 3: 패턴 추출 결과만 보고

### 인덱스 리프레시
1. 사용자 "인덱스만 갱신해줘"
2. Phase 0: incremental 모드
3. Phase 2-1 (analyzer incremental)만 실행 — 변경 파일만 재분석
4. Phase 3: 인덱스 변화량 보고
