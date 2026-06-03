# harness-ito

**ITO/SI 조직을 위한 확장 메타 하네스** — 코드베이스 분석에서 그치지 않고, 수정·개발·마이그레이션 작업까지 끊김 없이 지원한다.

[harness-new](https://github.com/Malburi/harness-new)의 4-에이전트 파이프라인 위에 ITO/SI 실무 에이전트와 워크플로우 스킬을 더했다.
[revfactory/harness](https://github.com/revfactory/harness) 메타 방법론 기반.

---

## 설치

Claude Code에서 두 줄로 설치:

```
/plugin marketplace add Malburi/harness-ito
/plugin install harness-ito@harness-ito
```

설치 후 대상 프로젝트 루트에서 Claude Code를 열고:

```
"하네스 초기화해줘"
```

> **수동 설치:** `git clone https://github.com/Malburi/harness-ito && cp -r harness-ito/agents harness-ito/skills /path/to/project/.claude/`

---

## harness-new와 비교

| 영역 | harness-new | harness-ito |
|------|-------------|-------------|
| 코드 분석 | 7단계 | 15단계 (의존성 그래프·데이터 흐름·트랜잭션·외부 통신·환경 분기·인증/인가·데드 코드 추가) |
| 인덱스 | 없음 | `_workspace/index/*.json` 8종 — 후속 에이전트가 grep 없이 직접 조회 |
| 워크플로우 스킬 | 3종 | 8종 |
| 에이전트 | 4종 | 14종 |
| 모델 | 단일 | opus 5종(심층 분석) / sonnet 9종(패턴·검증) |
| **적응 실행** | **없음** | **복잡도 자동 감지 → Lite/Standard/Full 분기** |
| 수정 안전성 | 없음 | impact-analyzer + change-safety 사전·사후 게이트 |
| 마이그레이션 | 없음 | migration-planner — 인벤토리·매핑·단계별 계획·롤백 자동 생성 |
| SQL 리뷰 | 없음 | sql-reviewer — 사용처·인덱스·N+1·인젝션·DDL 영향 종합 |
| 레거시 해석 | 없음 | legacy-decoder — 의도 역공학 |
| 문서 동기화 | 없음 | doc-syncer — 코드 ↔ 문서 일관성 점검 |

---

## 3-Tier 적응 실행

프로젝트 규모에 상관없이 전체 파이프라인을 도는 낭비를 없애기 위해, `harness-init`은 시작 전 **복잡도 점수**를 계산해 실행 범위를 자동 결정한다.

### 복잡도 점수 산정

| 항목 | 점수 |
|------|------|
| 소스 파일 수 (`.java` `.ts` `.py` `.vue` 등) | 파일 수 × 1 |
| DB / ORM 존재 (mybatis·jpa·typeorm·prisma 등) | +30 |
| 레거시 스택 (Struts·iBatis·JSP 50개+·전자정부·web.xml) | +40 |
| 멀티 모듈 (하위 `pom.xml`·`build.gradle`·`package.json` 2개+) | +20 |
| 외부 시스템 (RestTemplate·axios·kafka·feign 등) | +20 |

### Tier별 실행 구성

| Tier | 점수 | analyzer | writer | pattern | validator | QA |
|------|------|----------|--------|---------|-----------|-----|
| **Lite** | 0 ~ 50 | Phase A만 / sonnet | sonnet | 스킵 | sonnet | 스킵 |
| **Standard** | 51 ~ 120 | A + 스택 해당 B만 / sonnet | sonnet | 실행 | sonnet | 스킵 |
| **Full** | 121+ | A + B 전체 / opus | opus | 실행 | sonnet | 실행 |

### 수동 override

복잡도 점수 대신 요청 키워드로 Tier를 강제할 수 있다:

| 키워드 | 강제 Tier |
|--------|----------|
| "빠르게", "간단히", "quick" | **Lite** |
| "깊게", "심층", "마이그레이션", "레거시", "deep" | **Full** |

---

## 워크플로우 (Day 1 이후)

하네스 초기화 후 일상 작업에서 자연어로 트리거:

```
"OrderService.cancel 영향도 분석해줘"
  → analyze-impact → impact-analyzer

"이 변경 안전하게 적용해줘"
  → safe-modify → impact-analyzer + change-safety (GO / HOLD / STOP 판정)

"주문 취소 기능을 컨벤션 따라 만들어줘"
  → scaffold-feature → 전 레이어 생성 + 테스트 골격

"Spring Boot로 마이그레이션 계획 짜줘"
  → plan-migration → migration-planner (인벤토리·매핑·단계·롤백)

"이 SQL 점검해줘"
  → review-sql → sql-reviewer (사용처·인덱스·N+1·인젝션·DDL)

"이 PL/SQL 뭐하는 거야"
  → legacy-decoder → 역공학 리포트

"변경 사항 문서 동기화"
  → doc-syncer → CLAUDE.md / README / ADR 업데이트 권고

"주문 취소 로직 어떻게 돼?"
  → trace-logic → logic-tracer (진입점 → Controller → Service → DB 흐름)

"결제 관련 코드 어디 있어?"
  → find-feature → feature-finder (레이어별 파일·클래스·SQL 목록)
```

---

## 지원 스택

ITO/SI 현장에서 만나는 레거시 스택을 포함해 자동 탐지한다.

| 카테고리 | 탐지 스택 |
|---------|----------|
| Java EE 레거시 | Struts 1.x/2.x, Spring 3~4, iBatis, EJB 2, JSP/JSTL |
| Spring | Spring Boot 2/3, Spring MVC, Spring Data JPA, MyBatis, Spring Security |
| 전자정부 | egovframework (한국 공공) |
| Node.js | Express, NestJS, Next.js, Fastify, Koa |
| 프런트엔드 | Vue 2/3, Nuxt 2/3, Pinia/Vuex, Vue Router, Vite, Vue CLI, React, Angular 15+, AngularJS 1.x, Svelte/SvelteKit |
| Python | FastAPI, Django, Flask |
| .NET | .NET Framework 2~4, .NET Core, .NET 5~8, ASP.NET Core |
| DB | Oracle (PL/SQL), PostgreSQL, MySQL/MariaDB, Tibero, Altibase, SQL Server |
| 마이그레이션 대상 | Struts→Spring, iBatis→MyBatis/JPA, EJB→Spring, JSP→React/Vue, .NET FW→.NET Core, Oracle→PostgreSQL, AngularJS→Angular, Vue 2→Vue 3, Vuex→Pinia, Nuxt 2→Nuxt 3, Vue CLI→Vite |

상세 매트릭스: [`docs/stack-matrix.md`](docs/stack-matrix.md)

---

## 에이전트 & 스킬 목록

### 워크플로우 스킬 (8종)

| 스킬 | 역할 |
|------|------|
| `harness-init` | 분석 → 생성 → 검증 → QA 오케스트레이터 (3-Tier 적응 실행) |
| `analyze-impact` | 변경 영향도 분석 |
| `safe-modify` | 사전 영향 + 적용 + 사후 안전성 게이트 |
| `scaffold-feature` | 컨벤션 기반 신규 기능 전 레이어 생성 |
| `plan-migration` | 스택 마이그레이션 계획 |
| `review-sql` | SQL 종합 리뷰 |
| `trace-logic` | 기능·API 처리 흐름 추적 |
| `find-feature` | 기능명·키워드로 관련 코드 위치 탐색 |

### 에이전트 (14종)

| 에이전트 | 모델 | 역할 |
|---------|------|------|
| `analyzer` | opus / sonnet¹ | 코드베이스 심층 분석 + 인덱스 생성 |
| `writer` | opus / sonnet¹ | CLAUDE.md·스킬·에이전트·패턴 생성 |
| `pattern-extractor` | sonnet | 코드 컨벤션 패턴 추출 |
| `validator` | sonnet | 하네스 구조 검증 |
| `qa` | sonnet | 6개 경계면 교차 비교 (Full Tier만) |
| `impact-analyzer` | opus | 변경 영향도 분석 |
| `change-safety` | sonnet | 변경 안전성 평가 (GO / HOLD / STOP) |
| `migration-planner` | opus | 마이그레이션 계획 수립 |
| `test-generator` | sonnet | 회귀 테스트 골격 생성 |
| `sql-reviewer` | sonnet | SQL 다각도 리뷰 |
| `legacy-decoder` | opus | 레거시 코드 역공학 |
| `doc-syncer` | sonnet | 코드 ↔ 문서 동기화 점검 |
| `logic-tracer` | sonnet | 진입점→DB 처리 흐름 추적 |
| `feature-finder` | sonnet | 기능명·키워드 코드 위치 탐색 |

¹ Tier에 따라 결정 (Lite/Standard: sonnet, Full: opus)

---

## 인덱스 레이어

분석 결과를 JSON 인덱스로 저장해 후속 에이전트가 매번 grep하지 않도록 한다.

| 인덱스 파일 | 활용 에이전트 |
|------------|-------------|
| `call_graph.json` | impact-analyzer — 직접 영향 추적 |
| `symbols.json` | 전체 클래스·메서드·함수 위치 |
| `sql_usage.json` | sql-reviewer, impact-analyzer |
| `schema.json` | sql-reviewer DDL 영향 분석 |
| `transactions.json` | change-safety 사이드이펙트 평가 |
| `external_io.json` | migration-planner 위험 평가 |
| `env_branches.json` | impact-analyzer 환경별 영향 |
| `dead_code.json` | migration-planner 제외 대상 |

> **증분 갱신:** `analyzer incremental` 모드로 변경 파일만 재분석해 인덱스를 부분 갱신한다.

스키마 상세: [`docs/index-spec.md`](docs/index-spec.md)

---

## 생성 결과물 구조

```
프로젝트/
├── CLAUDE.md                          ← 프로젝트 개요 + 워크플로우 스킬 포인터
└── .claude/
    ├── skills/
    │   ├── trace.md                   ← 요청 흐름 추적
    │   ├── scaffolder.md              ← 신규 파일 체크리스트
    │   ├── find-logic.md              ← 로직 위치 탐색
    │   ├── analyze-impact.md          ← 영향도 분석
    │   ├── safe-modify.md             ← 안전 변경
    │   ├── scaffold-feature.md        ← 컨벤션 신규 생성
    │   ├── plan-migration.md          ← 마이그레이션 (조건부)
    │   └── review-sql.md              ← SQL 리뷰 (조건부)
    ├── agents/
    │   └── domain-expert.md           ← 프로젝트 도메인 지식
    └── patterns/
        ├── controller_pattern.md
        ├── service_pattern.md
        ├── dao_pattern.md
        ├── error_handling_pattern.md
        ├── validation_pattern.md
        ├── test_pattern.md
        └── client_side_pattern.md     (프런트엔드 존재 시)

_workspace/                            ← 런타임 산출물 (.gitignore 권장)
├── 01_analyzer_report.md
├── 02_writer_files.md
├── 03_validator_report.md
├── 04_qa_report.md                    (Full Tier만)
├── 05_patterns_extracted.md           (Standard/Full만)
├── index/
│   ├── call_graph.json
│   ├── symbols.json
│   ├── sql_usage.json
│   ├── schema.json
│   ├── transactions.json
│   ├── external_io.json
│   ├── env_branches.json
│   └── dead_code.json
├── impact_<slug>.md
├── safety_<slug>.md
├── sql_review_<slug>.md
├── decoded_<slug>.md
├── tests_<slug>.md
├── docs_sync_<slug>.md
└── migration/
    ├── 00_context.md
    ├── 00_inventory.md
    ├── 01_mapping_table.md
    ├── 02_phased_plan.md
    ├── 03_risk_register.md
    ├── 04_test_strategy.md
    ├── 05_rollback_plan.md
    └── checkpoints/phase[1-4].md
```

---

## ITO/SI 운영 모드

자연어 키워드로 change-safety의 평가 가중치를 자동 조정한다:

| 키워드 | mode | 영향 |
|--------|------|------|
| "운영 패치", "프로덕션" | production | 보안·롤백 가중치 ↑ |
| "긴급 핫픽스" | hotfix | 변경 라인 임계 ↓ (작은 변경만 GO) |
| "레거시 손보기" | legacy | 컨벤션 가중치 ↓ |
| "고객 데모 직전" | customer_facing | 외부 영향 가중치 ↑ |
| "야간 배치" | batch | 대량 처리 가중치 ↓ |

---

## 참고

- [Malburi/harness-new](https://github.com/Malburi/harness-new) — 기반 4-에이전트 파이프라인
- [revfactory/harness](https://github.com/revfactory/harness) — 메타 하네스 설계 원칙
- [Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code/overview)
- [docs/workflows.md](docs/workflows.md) — 스킬별 시나리오 모음
- [docs/stack-matrix.md](docs/stack-matrix.md) — 지원 스택 상세 매트릭스
