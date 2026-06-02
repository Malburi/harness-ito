# CLAUDE.md

**harness-fin** — ITO/SI 조직을 위한 확장 메타 하네스 템플릿.

## 이 저장소의 역할

`.claude/` 폴더에 *코드베이스 분석 + 수정/개발/마이그레이션 작업까지* 지원하는 에이전트 팀과 워크플로우 스킬이 포함되어 있다.

대상 프로젝트의 코드베이스를 분석해 맞춤형 CLAUDE.md / 워크플로우 스킬 / 도메인 에이전트 / 패턴 / 인덱스를 한 번에 생성하고, 이후 *수정·개발·마이그레이션 작업*까지 끊김 없이 지원한다.

[Malburi/harness-new](https://github.com/Malburi/harness-new)의 4-에이전트 파이프라인을 기반으로 [revfactory/harness](https://github.com/revfactory/harness)의 메타 방법론을 확장 적용했다.

## 실행 모드

**에이전트 팀** — `TaskCreate` 의존성 + `_workspace/` 파일 기반 산출물 전달

**팀 구성:**
- 분석/생성 파이프라인: `analyzer` → `writer` → (`pattern-extractor`) → `validator` → `qa`
- 작업용 에이전트: `impact-analyzer`, `change-safety`, `migration-planner`, `test-generator`, `sql-reviewer`, `legacy-decoder`, `doc-syncer`

## 파일 구조

| 경로 | 역할 |
|------|------|
| `.claude/skills/harness-init.md` | 메인 오케스트레이터 (분석 → 생성 → 검증 → QA → 패턴) |
| `.claude/skills/analyze-impact.md` | 영향도 분석 워크플로우 |
| `.claude/skills/safe-modify.md` | 안전 변경 워크플로우 (사전 영향 + 사후 안전성) |
| `.claude/skills/scaffold-feature.md` | 컨벤션 기반 신규 기능 스캐폴딩 |
| `.claude/skills/plan-migration.md` | 마이그레이션 계획 워크플로우 |
| `.claude/skills/review-sql.md` | SQL 종합 리뷰 워크플로우 |
| `.claude/agents/analyzer.md` | Phase 2-1: 심층 분석 (스택 + 의존성 그래프 + 데이터 흐름 + 트랜잭션 + 외부 통신 + 인덱스 생성) |
| `.claude/agents/writer.md` | Phase 2-2: 하네스 파일 + 워크플로우 스킬 생성 |
| `.claude/agents/pattern-extractor.md` | Phase 2-2.5: 컨벤션 추출 (writer 직후) |
| `.claude/agents/validator.md` | Phase 2-3: 구조 검증 + 인덱스 무결성 |
| `.claude/agents/qa.md` | Phase 2-4: 경계면 교차 비교 (Boundary 1~6) |
| `.claude/agents/impact-analyzer.md` | 변경 영향도 분석 |
| `.claude/agents/change-safety.md` | 변경 안전성 평가 (GO/HOLD/STOP) |
| `.claude/agents/migration-planner.md` | 스택 마이그레이션 계획 |
| `.claude/agents/test-generator.md` | 회귀 테스트 골격 생성 |
| `.claude/agents/sql-reviewer.md` | SQL 다각도 리뷰 |
| `.claude/agents/legacy-decoder.md` | 레거시 코드 역공학 |
| `.claude/agents/doc-syncer.md` | 코드 ↔ 문서 동기화 점검 |

## 자동 워크플로우

| 상황 | 트리거 스킬 |
|------|---------|
| 하네스 초기화 / 재초기화 | `harness-init` |
| 변경 영향도 분석 | `analyze-impact` |
| 안전한 변경 진행 | `safe-modify` |
| 컨벤션 따라 신규 기능 생성 | `scaffold-feature` |
| 스택 마이그레이션 계획 | `plan-migration` |
| SQL 리뷰 | `review-sql` |
| 레거시 코드 해석 | `legacy-decoder` 직접 호출 |
| 문서 동기화 | `doc-syncer` 직접 호출 |

## 에이전트 수정

에이전트 개선은 `.claude/agents/` 파일을 직접 수정한다.  
변경사항은 아래 변경 이력 테이블에 기록한다 (revfactory/harness Phase 5-4 템플릿).

새 에이전트 추가:
1. `.claude/agents/[name].md` 작성 (frontmatter + 본문)
2. 호출하는 오케스트레이터 스킬에 등록
3. 변경 이력 테이블에 기록

## 변경 이력

| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-06-02 | harness-new 기반 확장 — P0(impact-analyzer, change-safety) + P1(pattern-extractor, migration-planner) + P2(test-generator, sql-reviewer, legacy-decoder, doc-syncer) + 워크플로우 스킬 5종(analyze-impact, safe-modify, scaffold-feature, plan-migration, review-sql) + analyzer 심층 분석(의존성 그래프, 데이터 흐름, 트랜잭션, 외부 통신, 환경 분기, 데드 코드) + 인덱스 레이어(_workspace/index/*.json) | 전체 | ITO/SI 조직의 수정/개발/마이그레이션 작업까지 끊김 없이 지원하기 위함 |
