# harness-ito

**ITO/SI 조직을 위한 확장 메타 하네스** — 코드베이스 분석에서 그치지 않고, *수정·개발·마이그레이션 작업*까지 끊김 없이 지원한다.

[Malburi/harness-new](https://github.com/Malburi/harness-new)의 4-에이전트 파이프라인 위에 ITO/SI 실무에 필요한 P0/P1/P2 에이전트와 워크플로우 스킬을 더했다.

## 설치 (Claude Code 플러그인)

Claude Code에서 한 줄로 설치:

```
/plugin marketplace add Malburi/harness-ito
/plugin install harness-ito
```

설치되면 14개 에이전트(`analyzer`, `writer`, `validator`, `qa`, `impact-analyzer`, `change-safety`, `migration-planner`, `test-generator`, `sql-reviewer`, `legacy-decoder`, `doc-syncer`, `pattern-extractor`, `logic-tracer`, `feature-finder`)와 8개 워크플로우 스킬(`harness-init`, `analyze-impact`, `safe-modify`, `scaffold-feature`, `plan-migration`, `review-sql`, `trace-logic`, `find-feature`)이 자동 로드된다.

대상 프로젝트에서 바로:

```
"하네스 초기화해줘"
```

### 수동 설치 (플러그인 없이)

```bash
git clone https://github.com/Malburi/harness-ito.git
cp -r harness-ito/.claude /path/to/your/project/
cp harness-ito/CLAUDE.md /path/to/your/project/
```

## 무엇이 다른가

기존 harness-new가 *"코드베이스 → 하네스 파일 생성"* 한 가지만 자동화했다면, harness-fin은 *"하네스 생성 + 수정·개발·마이그레이션 작업의 안전 게이트"*까지 모두 자동화한다.

| 영역 | harness-new | harness-fin |
|------|------------|-------------|
| 코드베이스 분석 | 7단계 분석 | 15단계 (의존성 그래프, 데이터 흐름, 트랜잭션 경계, 외부 통신, 환경 분기, 인증/인가, 데드 코드 추가) |
| 인덱싱 | 없음 (일회성 마크다운) | `_workspace/index/*.json` 8종 (call_graph, symbols, sql_usage, schema, transactions, external_io, env_branches, dead_code) |
| 생성 스킬 | 3종 (trace, scaffolder, find-logic) | 8종 (기존 3종 + analyze-impact, safe-modify, scaffold-feature, plan-migration, review-sql, **trace-logic, find-feature**) |
| 에이전트 | 4종 (analyzer, writer, validator, qa) | 14종 (기존 4종 + pattern-extractor, impact-analyzer, change-safety, migration-planner, test-generator, sql-reviewer, legacy-decoder, doc-syncer, **logic-tracer, feature-finder**) |
| 모델 최적화 | 단일 모델 | opus 5종 (심층 분석·추론) / sonnet 9종 (패턴 처리·검증) |
| 수정 안전성 | 없음 | impact-analyzer + change-safety로 사전·사후 게이트 |
| 마이그레이션 | 없음 | migration-planner로 인벤토리·매핑·단계별 계획·롤백 자동 생성 |
| SQL 리뷰 | 없음 | sql-reviewer로 사용처·인덱스·N+1·인젝션·DDL 영향 종합 |
| 레거시 코드 | 없음 | legacy-decoder로 의도 역공학 |
| 문서 동기화 | 없음 | doc-syncer로 코드 ↔ 문서 일관성 점검 |

## 빠른 시작

플러그인 설치 후 (위 *설치* 섹션 참고), 대상 프로젝트 루트에서 Claude Code를 실행하고:

```
"하네스 초기화해줘"
```

`harness-init` 스킬이 자동 트리거되어 *분석 → 생성 → 검증 → QA → 패턴 추출* 파이프라인을 실행한다.

## 워크플로우 (Day 1 이후)

하네스 초기화 후 일상 작업에서:

```
"OrderService.cancel 영향도 분석해줘"
  → analyze-impact 트리거 → impact-analyzer 실행

"이 변경 안전하게 적용해줘"
  → safe-modify 트리거 → impact-analyzer + 적용 + change-safety 실행

"주문 취소 기능을 컨벤션 따라 만들어줘"
  → scaffold-feature 트리거 → 컨벤션 기반 전 레이어 생성 + 테스트 골격

"Spring Boot로 마이그레이션 계획 짜줘"
  → plan-migration 트리거 → migration-planner 실행

"이 SQL 점검해줘"
  → review-sql 트리거 → sql-reviewer 실행

"이 PL/SQL 뭐하는 거야"
  → legacy-decoder 직접 호출 → 역공학 리포트

"변경 사항 문서 동기화"
  → doc-syncer 호출 → CLAUDE.md/README/ADR 업데이트 권고

"주문 취소 로직 어떻게 돼?"
  → trace-logic 트리거 → logic-tracer 실행 → 진입점→Controller→Service→DB 흐름 리포트

"결제 관련 코드 어디 있어?"
  → find-feature 트리거 → feature-finder 실행 → 레이어별 파일·클래스·SQL 목록
```

## 지원 스택 (자동 탐지 확장)

ITO/SI 현장에서 흔히 만나는 레거시 포함:

| 카테고리 | 탐지 가능 스택 |
|---------|------------|
| Java EE 레거시 | Struts 1.x/2.x, Spring 3~4, iBatis, EJB 2, JSP/JSTL |
| Spring | Spring Boot 2/3, Spring MVC, Spring Data JPA, MyBatis, Spring Security |
| 전자정부 표준프레임워크 | egovframework (한국 공공) |
| Node.js | Express, NestJS, Next.js, Fastify, Koa |
| 프런트엔드 | Vue 2/3, Nuxt 2/3, Pinia/Vuex, Vue Router, Vite, Vue CLI, React, Angular 15+, AngularJS 1.x, Svelte/SvelteKit |
| Python | FastAPI, Django, Flask |
| .NET | .NET Framework 2~4, .NET Core, .NET 5~8, ASP.NET Core |
| DB | Oracle (PL/SQL), PostgreSQL, MySQL/MariaDB, Tibero, Altibase, SQL Server |
| 마이그레이션 대상 (특별 처리) | Struts → Spring, iBatis → MyBatis/JPA, EJB → Spring, JSP → React/Vue, .NET FW → .NET Core, Oracle → PostgreSQL, AngularJS → Angular, **Vue 2 → Vue 3, Vuex → Pinia, Nuxt 2 → Nuxt 3, Vue CLI → Vite** |

자세한 매트릭스: [`docs/stack-matrix.md`](docs/stack-matrix.md)

## 생성 결과물

```
프로젝트/
├── CLAUDE.md                              ← 프로젝트 개요 + 워크플로우 스킬 포인터
└── .claude/
    ├── skills/
    │   ├── trace.md                       ← 요청 흐름 추적 (기본)
    │   ├── scaffolder.md                  ← 신규 파일 체크리스트 (기본)
    │   ├── find-logic.md                  ← 로직 위치 탐색 (기본)
    │   ├── analyze-impact.md              ← 영향도 분석 (NEW)
    │   ├── safe-modify.md                 ← 안전 변경 (NEW)
    │   ├── scaffold-feature.md            ← 컨벤션 신규 생성 (NEW)
    │   ├── plan-migration.md              ← 마이그레이션 (NEW, 조건부)
    │   └── review-sql.md                  ← SQL 리뷰 (NEW, 조건부)
    ├── agents/
    │   └── domain-expert.md               ← 프로젝트 도메인 지식
    └── patterns/
        ├── controller_pattern.md          ← pattern-extractor가 채움
        ├── service_pattern.md
        ├── dao_pattern.md
        ├── error_handling_pattern.md
        ├── validation_pattern.md
        ├── test_pattern.md
        └── client_side_pattern.md         (해당 시)

_workspace/                                ← 런타임 산출물 (gitignore)
├── 01_analyzer_report.md
├── 02_writer_files.md
├── 03_validator_report.md
├── 04_qa_report.md
├── 05_patterns_extracted.md
├── index/
│   ├── call_graph.json
│   ├── symbols.json
│   ├── sql_usage.json
│   ├── schema.json
│   ├── transactions.json
│   ├── external_io.json
│   ├── env_branches.json
│   └── dead_code.json
├── impact_<slug>.md                       ← 영향도 분석 결과
├── safety_<slug>.md                       ← 안전성 평가 결과
├── sql_review_<slug>.md                   ← SQL 리뷰 결과
├── decoded_<slug>.md                      ← 레거시 해석 결과
├── tests_<slug>.md                        ← 테스트 생성 요약
├── docs_sync_<slug>.md                    ← 문서 동기화 권고
└── migration/                             ← 마이그레이션 산출물
    ├── 00_context.md, 00_inventory.md
    ├── 01_mapping_table.md
    ├── 02_phased_plan.md
    ├── 03_risk_register.md
    ├── 04_test_strategy.md
    ├── 05_rollback_plan.md
    └── checkpoints/phase[1-4].md
```

## ITO/SI 운영 모드

자연어 키워드로 평가 가중치 자동 조정:

| 키워드 | mode | 영향 |
|--------|------|------|
| "운영 패치", "프로덕션" | production | 보안·롤백 가중치 ↑ |
| "긴급 핫픽스" | hotfix | 변경 라인 임계 ↓ (작은 변경만 GO) |
| "레거시 손보기" | legacy | 컨벤션 가중치 ↓ |
| "고객 데모 직전" | customer_facing | 외부 영향 가중치 ↑ |
| "야간 배치" | batch | 대량 처리 가중치 ↓ |
| "OLTP" | oltp | 락 가중치 ↑ |
| "OLAP" | olap | 성능 가중치 ↑, 락 ↓ |

## 인덱스 레이어 (NEW)

분석 결과를 *재사용 가능한 JSON 인덱스*로 저장해 후속 작업이 매번 grep하지 않도록 한다:

| 인덱스 | 용도 |
|--------|------|
| `call_graph.json` | 호출자/호출 대상 그래프 — analyze-impact가 직접 영향 추적 |
| `symbols.json` | 모든 클래스/메서드/함수 위치 |
| `sql_usage.json` | SQL ID ↔ 호출 위치 — review-sql, impact-analyzer 활용 |
| `schema.json` | DB 스키마 스냅샷 — review-sql DDL 영향 분석 |
| `transactions.json` | 트랜잭션 경계 — change-safety 사이드이펙트 평가 |
| `external_io.json` | 외부 통신 — migration-planner 위험 평가 |
| `env_branches.json` | 환경 분기 — impact-analyzer 환경별 영향 |
| `dead_code.json` | 데드 코드 후보 — migration-planner 제외 대상 |

증분 모드: `analyzer`를 `incremental` 모드로 호출하면 변경 파일만 재분석해 인덱스를 부분 갱신한다.

스키마 상세: [`docs/index-spec.md`](docs/index-spec.md)

## 워크플로우 가이드

각 스킬을 *언제 어떻게* 쓰는지 시나리오 모음: [`docs/workflows.md`](docs/workflows.md)

## QA 에이전트 — 6가지 경계면 검증

기존 harness-new의 4-Boundary에 추가:

| Boundary | 검증 대상 |
|---------|---------|
| 1~4 | 스택별 (Struts↔Service↔Bean, Service↔Query, 스킬↔코드, forward↔JSP 등) |
| 5 (NEW) | 인덱스 ↔ 실제 코드 일관성 |
| 6 (NEW) | 워크플로우 스킬 ↔ 인덱스 의존성 |

원칙은 동일 — 양쪽을 동시에 읽고 Set 연산으로 mismatch 탐지.

## 협업 제약

이 하네스는 `TeamCreate`/`SendMessage` 도구가 없는 환경 기준. 다음 두 채널로 협업:

| 채널 | 도구 | 용도 |
|------|------|------|
| 작업 조율 | `TaskCreate`/`TaskUpdate` | 진행 추적, 의존성 |
| 산출물 전달 | `_workspace/` 파일 (마크다운 + JSON 인덱스) | 분석·생성·검증·평가 |

## 에러 핸들링

핵심 원칙: **1회 재시도 후 재실패 시 결과 없이 진행하고 보고서에 누락 명시. 자동 수정 금지.**

| 상황 | 대응 |
|------|------|
| analyzer 산출물 누락 | 1회 재실행, 재실패 시 보고 후 중단 |
| writer 일부 파일 누락 | 누락 목록 보고, validator는 생성된 파일만 검증 |
| pattern-extractor 실패 | patterns/ 스켈레톤 유지, 재실행 권고 |
| validator 보안 위험 | 자동 수정 금지, 위치 명시 |
| qa DEAD/ORPHAN | 자동 수정 금지, 우선순위 표시 |
| validator 신뢰도 < 50 | qa 스킵, 권고 우선 처리 후 재실행 |
| 인덱스 일부 누락 | 영향 받는 워크플로우 스킬은 WARN, 사용자에게 incremental 재실행 권고 |
| analyze-impact 인덱스 stale | 사용자에게 알리고 incremental 재실행 권고 |

## 참고

- [Malburi/harness-new](https://github.com/Malburi/harness-new) — 기반 4-에이전트 파이프라인
- [revfactory/harness](https://github.com/revfactory/harness) — 메타 하네스 설계 원칙 원본
- [Claude Code 공식 문서](https://docs.claude.com/en/docs/claude-code/overview)
