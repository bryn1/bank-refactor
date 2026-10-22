PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# 02 — Refactor-feasibility plan: apache/fineract as a clean-code study target (rev 2, DA FIX round 1)

MC 11124 ([type:plan], child of 11122). Input: `research/01-target-selection.md` + DA gate
`.audits/20261022-1447-da-plan-gate/out/DA-verdict.md` (# VERDICT: FIX; findings F-1..F-8 addressed here,
F-9/F-10 withdrawn/survived). Target pinned apache/fineract @ `355287c8feca747b3cd936a5b1cef1bd7804ffea`
(clone `.tmp/clones/fineract`). **Method law this rev: every LOC figure recounted with
`find <dir> -name '*.java' -exec cat {} + | wc -l`** — rev 1's `find | xargs wc -l | tail -1` chunked and
understated provider (F-4 root cause); all figures below re-run THIS session. Tags: VERIFIED (measured here,
path cited) / INFERRED. No code written; no build executed (S0/R1).

## 1. Target characterization (as-is, rev-2 corrected)

| Aspect | Measured finding (all VERIFIED this session unless tagged) |
|---|---|
| Architecture | Modular monolith: **36 `include` lines** in `settings.gradle` (plus dynamic `custom:*` discovery) (F-5 fix). Domain modules carry an api/service/domain/serialization/handler split. Legacy mass: `fineract-provider` **278,878 LOC `src/main` java** (module-wide 331,201 / 2,776 files — the 91,778 in rev 1 was the xargs artifact, F-4) + `fineract-core` 77,657 / 845. |
| Self-containment (decides slice order) | **`fineract-accounting` is self-contained**: its read/write service impls live IN the module (e.g. `…/accounting/glaccount/service/GLAccountReadPlatformServiceImpl.java`). **`fineract-charge` is NOT**: module = interfaces/domain/api/deserializer/handlers only, `grep -ric jdbctemplate fineract-charge/src/main` = **0 hits** (F-1); the charge behavior lives in `fineract-provider/…/portfolio/charge/` (949 LOC / 4 files): `ChargeReadPlatformServiceImpl.java` 502 l with **14 `jdbcTemplate.` calls**, `ChargeWritePlatformServiceJpaRepositoryImpl.java` 291 l (F-2 fix: a slice must name BOTH locations or pick a self-contained module — this plan picks accounting first, §3). |
| Size hotspots | `fineract-core/…/commands/service/CommandWrapperBuilder.java` 4,221 l; `fineract-provider/…/LoanWritePlatformServiceJpaRepositoryImpl.java` 3,703 l; `…/LoanProductDataValidator.java` 3,024 l (DA re-ran, SUSTAINED). Repo-wide largest files are TESTS (`ClientLoanIntegrationTest.java` 8,198 l). |
| Layer violation | 94 `*ServiceImpl` under `fineract-provider/src/main` call `jdbcTemplate.query/execute/update` directly (grep -rl; DA re-ran both patterns, SUSTAINED). Within the planned first slice (accounting glaccount+rule): 7 `jdbcTemplate.` lines in 3 files — a real, non-zero peel baseline (S3 is NOT vacuous like rev 1's was, F-1 lesson). |
| Test safety net | 1,002 `*Test*.java`; 99 `*IntegrationTest*`; 336 `.java` e2e files. Slice coverage thin (F-T1a framing adopted): charge 2 domain-only JUnit tests (no DB); accounting 3 (1 Mockito test pinning service-vs-mocked-repository — NOT DB behavior; 2 serialization). Characterization is the expensive part of every slice. |
| CI / quality gates | 27 workflows incl. `build-quality-checks.yml` = Spotless 8.10.2 (`build.gradle:112`) + Checkstyle + SpotBugs 6.5.4 (`:125`); DB matrix; `verify-api-backward-compatibility` / `verify-liquibase-*` — the repo's own gates are the refactor judge. |
| Build toolchain | Java **25** required (`build.gradle:418`); host JDK **21.0.12** → hard gap. Gradle wrapper 9.7.1. **No toolchain resolver in-repo** (`grep foojay` = 0 across `settings.gradle`/`build.gradle`/`gradle/`, F-8) → auto-provision is NOT a config flip; options in R1. Spring Boot 4.1.1 (`build.gradle:116`, VERIFIED — rev 1's carried-INFERRED upgraded, F-9). `docker-compose-development.yml` present; docker daemon UP (DA mechanical audit). |

## 2. What "modern clean code" means here — three strategies (DA F-10: option space + pick survive; restated with F-2 correction)

| # | Strategy | For | Against |
|---|---|---|---|
| A | Upstream-first clean-code PRs into apache/fineract | Real-world constraints; community review = strongest skill proof; repo gates already judge | Pace hostage to community review; contribution overhead eats seat-days |
| B | Fork-local slice refactor (pin @355287c8; slice-by-slice on our branch lineage; repo's own gates as judge) | Full pace control; module boundaries ALMOST pre-cut slices — "almost" is load-bearing (F-2): self-containment must be MEASURED per module before a slice is cut there | Upstream drift (mitigated by pin, R5); no outside review unless A bolted on |
| C | Strangler bounded rewrite of one domain, legacy kept running | Strongest teaching showpiece (legacy vs clean side-by-side) | Highest cost; API+Liquibase contract risk; drifts toward greenfield exercise, away from refactor skill |

RECOMMENDATION (unchanged, DA-sustained): **B spine + C's side-by-side write-up format + one capstone (S6)**;
A only as opportunistic polish, never a schedule dependency. Study goal = demonstrable skill + teaching
material, not shipping — B buys pace control on a real 919k-LOC corpus while Spotless/Checkstyle/SpotBugs +
tests keep "clean" objective.

## 3. Staged sequence (rev 2: first slice = self-contained accounting, per F-1/F-2 fix option (b))

| Stage | Concern | Depends on | Acceptance check | Parallel? |
|---|---|---|---|---|
| S0 | Toolchain + dev DB + green baseline (owner call, R1 three options) | owner decision | `./gradlew :fineract-accounting:test :fineract-charge:test` green on UNMODIFIED clone | NO — blocks all |
| S1 | Accounting baseline: map `glaccount`+`rule` deps + public API surface; characterization incl. DB-backed behavior pins (seed Mockito test pins service logic only) | S0 | characterization green on unmodified code AND proven RED (one planted-bad run flips it) | NO |
| S2 | Accounting cleanup r1 (4,261-LOC slice): split largest slice services behind existing interfaces (`AccountingRuleWritePlatformServiceJpaRepositoryImpl.java` 346 l), dead-code removal, api/ untouched | S1 | module Spotless+Checkstyle+SpotBugs green; characterization green; zero diff under accounting `api/` dirs | NO |
| S3 | Accounting JDBC peel: the 3 slice files with `jdbcTemplate.` (7 calls: GLAccount read 3, GLAccount write 1, AccountingRule read 3) → repository/mapper seam | S2 (same files) | gates + characterization green; slice `grep -c 'jdbcTemplate\.'` strictly reduced from 7 (non-zero baseline — meetable, unlike rev 1) | NO |
| S4 | Charge slice — REDEFINED per F-2: `fineract-charge` module (3,946) **plus** provider-resident `portfolio/charge/` (949 / 4 files); characterization first, then peel `ChargeReadPlatformServiceImpl` (14 `jdbcTemplate.` calls, 502 l) | S3 discipline proven | module gates + provider compiles + charge-touching integration/e2e green + `:fineract-accounting:test` green (compile guard, see parallelism note) | NO |
| S5 | Teaching artifact per slice: before/after measured metrics (largest-file LOC, SpotBugs count, gate output) | slice it documents | doc cites re-measured before/after numbers | YES — docs only |
| S6 | Capstone (optional): split `CommandWrapperBuilder.java` (4,221 l) behind a facade in core | S1–S3 discipline proven | gates + command-surface characterization green | last, serial |

**Sizing — what is sized to what (F-7 fix):** the PLANNED first slice is `glaccount`+`rule` = **4,261 LOC /
49 files** (2,185 + 2,076, VERIFIED), ≈ 2–3 seat-days with characterization the dominant cost — NOT the
14,314-LOC whole module (that figure sizes the module, not the plan). Charge stage = 3,946 + 949 = 4,895 LOC
across two modules ≈ 2–3 seat-days (behavior pin leans on existing integration/e2e nets, not module tests).

**Parallelism — honest (F-6 fix): every code stage is serial.** `fineract-accounting/dependencies.gradle:28`
= `implementation(project(path: ':fineract-charge'))` — a real compile edge; ordering accounting→charge means
the charge stage changes a dependency accounting compiles against, hence S4's acceptance runs accounting's
tests as a guard. "Zero file overlap" ≠ zero build coupling; rev 1's S4 ∥ cell is withdrawn. Only S5 docs run
∥. **Two-seat hygiene (if ever staffed):** each seat works in its OWN `git worktree add` off the pinned clone
(separate index/branch, own Gradle daemon) — never two seats in one tree.

## 4. Risks & guardrails

- **R1 (P0, only hard blocker) — build gap, three honest owner options (F-8 fix).** Host JDK 21 vs required 25; no in-repo resolver, so "auto-provision" would itself be a repo edit. (i) **Docker-based build** — run Gradle in a JDK-25 container with the repo mounted: zero host installs, pinned clone stays byte-identical; image pull not yet confirmed this session (INFERRED; daemon is UP). **Recommended.** (ii) Approved manual JDK-25 install on VM350 — host change, no repo divergence. (iii) Add `foojay-resolver` to `settings.gradle` — a mechanism ADDITION to a pinned upstream clone (breaks pin discipline R5 + new supply-chain surface; only acceptable if the owner wants host-native dev later). Until one lands, no slice may claim any green build — all such claims stay INFERRED.
- **R2 — build cost:** 919k-LOC tree; scope every run to `:module:test` (+ the S4 guard), never whole-repo `build` per seat-day; time-box slices.
- **R3 — scope creep:** study CAPPED to accounting + charge slices (+ optional S6); loan (69,025 LOC, 8,198-l test files) off-limits without a new owner ruling.
- **R4 — behavior preservation:** no Liquibase changelog, API DTO shape, or command-handler contract changes; the repo's `verify-api-*` / `verify-liquibase-*` workflows are the tripwires.
- **R5 — upstream drift:** pinned at the full SHA above; no mid-slice rebase; rebase only at slice boundaries if a slice later ships upstream.
- **R6 — license/attribution:** Apache-2.0 (01 §4): public write-ups carrying code keep LICENSE/NOTICE + attribution; process-only write-ups carry none.
- **R7 — ghost completion:** a characterization suite counts only after one planted-bad run proves it goes RED; "acceptance check unmeetable-by-construction" (rev 1 S3, F-1) is the formal failure mode — every reduction check now names a non-zero baseline.

## RESULT
ARTIFACT: /home/claudecode/bank-refactor/research/02-feasibility-plan.md
OUTCOME: rev 2 — DA FIX round 1 addressed in full: first slice re-scoped to self-contained accounting (option b, F-1/F-2), §1 figures recounted with the robust method (provider 278,878 src/main; modules 36; F-4/F-5), S4 parallelism withdrawn + worktree rule added (F-6), sizing stated at the 4,261-LOC planned-slice level (F-7), R1 now a three-option owner choice with docker-build recommended (F-8). Strategy/recommendation unchanged (F-10 sustained). No code; no build executed.
