PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# 02 — Refactor-feasibility plan: apache/fineract as a clean-code study target

MC 11124 ([type:plan], child of 11122). Input: `research/01-target-selection.md` (T1b, pinned target
apache/fineract @ `355287c8`, clone `.tmp/clones/fineract`). All figures measured THIS session on that
clone (`find|xargs wc -l`, `grep -r`, `ls`, `java -version`); tags: VERIFIED (measured here, path cited) /
INFERRED (reasoned, incl. facts carried from 01). No code written; no build executed (see R1).

## 1. Target characterization (as-is)

| Aspect | Measured finding (all VERIFIED unless tagged) |
|---|---|
| Architecture | Modular monolith: 30 Gradle modules (`settings.gradle`); domain modules ALREADY carry an api/service/domain/serialization/handler/request/exception split (`fineract-charge/src/main/java/org/apache/fineract/portfolio/charge/…`). The legacy mass sits in `fineract-provider` (91,778 LOC / 2,776 files) and `fineract-core` (77,657 / 845). |
| Size hotspots | `fineract-core/…/commands/service/CommandWrapperBuilder.java` 4,221 lines; `fineract-provider/…/loanaccount/service/LoanWritePlatformServiceJpaRepositoryImpl.java` 3,703; `fineract-provider/…/loanproduct/serialization/LoanProductDataValidator.java` 3,024. Largest files repo-wide are TESTS (`integration-tests/…/ClientLoanIntegrationTest.java` 8,198) — the test corpus is a smell too. |
| Layer violation | 94 `*ServiceImpl` files under `fineract-provider/src/main` call `jdbcTemplate.query/execute/update` directly beside JPA repositories (grep -rl count) — the raw-JDBC-beside-JPA seam to peel. |
| Test safety net | 1,002 `*Test*.java`, 99 `*IntegrationTest*`, 336 e2e files (`fineract-e2e-tests-*`). BUT slice coverage is thin: `fineract-charge` has 2 test files (domain only), `fineract-accounting` 3 — characterization work is required, not optional. |
| CI / quality gates | 27 GitHub workflows incl. `build-quality-checks.yml` = Spotless 8.10.2 + Checkstyle + SpotBugs 6.5.4 (plugin ids in `build.gradle`); DB matrix (MariaDB/MySQL/PostgreSQL); API- and Liquibase-backward-compatibility verify workflows — the project itself gates refactors. |
| Build toolchain | Gradle wrapper; Java toolchain REQUIRED = 25 (`build.gradle:418` `JavaLanguageVersion.of(25)`); THIS host has JDK 21.0.12 (`java -version`) → build cannot run as-is (R1, the only hard blocker). `docker-compose-development.yml` present for the dev DB. Spring Boot 4.1.1 (carried from 01 §1, INFERRED here). |

## 2. What "modern clean code" means here — three strategies

| # | Strategy | For | Against |
|---|---|---|---|
| A | **Upstream-first**: clean-code PRs into apache/fineract itself | Real-world constraints; community review = strongest external skill proof; its gates (Spotless/Checkstyle/SpotBugs/tests) are already the judge | Pace set by community review; large prior refactors already in flight; contribution-process overhead can eat seat-days; study schedule hostage to strangers |
| B | **Fork-local slice refactor**: pin @355287c8, refactor module-by-module on our own branch lineage, keep the repo's own gates as judge | Full pace control; objective before/after metrics per slice (largest-file LOC, SpotBugs count, characterization green); slices are pre-cut by the module boundary — no artificial seam-drawing | Upstream drift (mitigate: pin, rebase only at slice boundaries); no outside reviewer unless A is bolted on later |
| C | **Strangler bounded rewrite**: keep legacy running, rewrite `fineract-charge` as a clean side-by-side module honoring the same API/DB contract | Strongest teaching artifact (legacy vs clean, same behavior, side by side) — the best "demonstrable skill" showpiece | Highest cost; contract risk (API + Liquibase compatibility is what Fineract gates hardest); drifts from *refactor* skill toward greenfield exercise |

RECOMMENDATION: **B as the spine** (slice-by-slice, the repo's own gates as the objective judge), adopting
**C's side-by-side write-up format** as the per-slice teaching deliverable, with **one capstone** (§3 S6).
A is opportunistic polish: contribute a slice upstream only if it is already clean — never a schedule
dependency. Rationale: the study goal is demonstrable skill + teaching material; B buys pace control on a
real 919k-LOC legacy corpus while its CI gates keep "clean" objective instead of opinion (weighed
explicitly against the not-shipping goal).

## 3. Staged sequence (each stage = one concern, ~1 seat-day)

| Stage | Concern | Depends on | Acceptance check | Parallelizable |
|---|---|---|---|---|
| S0 | Toolchain bring-up: JDK 25 (Gradle toolchain or owner-approved install) + dev DB via `docker-compose-development.yml` | — (owner call, R1) | `./gradlew :fineract-charge:test` green on unmodified clone | NO — blocks all |
| S1 | Charge slice baseline: map module deps + public API surface; write characterization tests for current charge behavior | S0 | characterization suite green against UNMODIFIED code, and proven able to go RED (planted-behavior check flips it) | NO |
| S2 | Charge internal cleanup r1: split oversized service/deserializer classes behind existing interfaces, dead-code removal, api/ package untouched | S1 | Spotless+Checkstyle+SpotBugs green on module; characterization green; `git diff` shows zero changes under `…/charge/api/` | serial with S3 (same files) |
| S3 | Charge read-path peel: move raw `jdbcTemplate` reads (e.g. `…/charge/service/ChargeDropdownReadPlatformServiceImpl.java`) behind a repository/mapper seam | S2 | gates + characterization green; module `jdbcTemplate` grep count strictly reduced | NO |
| S4 | Accounting slice: repeat S1+S2 pattern on `accounting/glaccount` + `accounting/rule` (seed: existing `GLAccountWritePlatformServiceJpaRepositoryImplTest`) | S0; pattern proven in S1–S2 | same acceptance checks scoped to `:fineract-accounting` | YES — ∥ S2/S3 (zero file overlap with charge) |
| S5 | Teaching artifact per slice: before/after doc with measured metrics (largest-file LOC, SpotBugs count, gate output) | S2 (charge), S4 (accounting) | doc exists citing before/after numbers re-measured from the tree | YES — ∥ anything after the slice it documents |
| S6 | Capstone (optional): split `CommandWrapperBuilder.java` (4,221 lines) behind a facade in core | S1–S3 discipline proven | gates + command-surface characterization green | last, serial |

Ordering law honored: S2/S3 share module files ⇒ serial; S4 shares no files with the charge slice ⇒ parallel.

## 4. Risks & guardrails

- **R1 — build gap (P0, the only hard blocker).** Required JDK 25 vs host JDK 21 (both VERIFIED §1). Guardrail: S0 is an owner decision — Gradle auto-provisioned toolchain or approved install; until S0 passes, every "builds/tests green" claim stays INFERRED and no slice may be reported done.
- **R2 — build cost on a shared host.** 919k-LOC Gradle tree; scope every run to `:module:test`, never whole-repo `build` per seat-day; time-box each slice.
- **R3 — scope creep.** `fineract-loan` is 69,025 LOC and its integration tests 8,198 lines/file; study is CAPPED to charge + accounting (+ optional S6); any loan-domain work needs a new owner ruling.
- **R4 — behavior preservation.** No changes to Liquibase changelogs, API DTO shapes, or command-handler contracts; the project's own `verify-api-backward-compatibility` / `verify-liquibase-*` workflows are the tripwire to keep green.
- **R5 — upstream drift.** Study pinned at `355287c8`; no mid-slice rebase; if a slice later ships upstream (A), rebase only at a slice boundary.
- **R6 — license/attribution.** Apache-2.0 (01 §4, VERIFIED there): any public write-up carrying code keeps LICENSE/NOTICE + attribution; process-only write-ups carry no such obligation.
- **R7 — ghost completion.** "Tests pass ≠ refactor correct": a characterization suite counts only after one planted-bad run proves it goes RED (fleet gate rule); green-everything on first sight is the tell to re-check.

## RESULT
ARTIFACT: /home/claudecode/bank-refactor/research/02-feasibility-plan.md
OUTCOME: plan complete — RECOMMENDATION: Strategy B spine + C's write-up format, S0–S5 with optional S6 capstone; first blocker = S0 JDK-25 owner call (R1). 4 strategies-vs-gates facts re-measured this session; no code, no build executed.
