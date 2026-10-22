PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# DA-verdict-c2 — MC 11125 (cycle c2, [type:review]) — DA gate over REV 2 of the feasibility plan

Target under gate: `research/02-feasibility-plan.md` rev 2, sha256 `fe92cb2e6076…ebd24` (re-hashed this
session), committed `a68e370` (re-read `git log` this session). Clone: `.tmp/clones/fineract` @
`355287c8feca747b3cd936a5b1cef1bd7804ffea` (`git rev-parse HEAD` re-checked). Predecessor gate
`20261022-1447-da-plan-gate/out/DA-verdict.md` read, not edited. T1 not re-litigated (knut-VERIFIED, c1-survived).
Every instrument below re-run by me THIS session (read-only; no installs; no pushes).
Classification: SUSTAINED (I reproduce it) / REFUTED (my instrument contradicts it) / WITHDRAWN (my suspicion failed).

## A. c1 fixes — landed? honest? (instrument re-runs)

| c1 finding | REV 2 claims | My measurement this session | Class |
|---|---|---|---|
| F-1/F-2 | First slice = accounting `glaccount`+`rule`; charge redefined as module + provider-resident `portfolio/charge/` **949 LOC / 4 files** | Slice exists and sizes exactly (see F-7 row). `find fineract-provider/…/portfolio/charge -name '*.java'` = **4 files, 949 LOC** (ChargeConfiguration 77, ChargeWrite…Impl 291, ChargeRead…Impl 502, ConvertChargeData… 79). Charge stage now names BOTH locations. The vacuous-S3 premise is gone: slice JDBC baseline is non-zero (see B/b) | SUSTAINED — fix landed, honest |
| F-4/F-5 | provider **278,878** `src/main` java LOC (module-wide 331,201); **36** `include` lines + dynamic `custom:*` | `find fineract-provider/src/main -name '*.java' -exec cat {} + \| wc -l` = **278,878**. `grep -cE '^include' settings.gradle` = **36**; the 37th match (`^\s*include`) is the indented dynamic `include ":custom:…"` in the discovery loop (settings.gradle:94) — the plan's wording "36 include lines (plus dynamic custom:* discovery)" is exactly right | SUSTAINED — fix landed, honest |
| F-6 | all code stages serial; worktree rule; S4 acceptance adds `:fineract-accounting:test` guard | Stage table: S0–S4, S6 "NO", only S5 "YES". Worktree rule present ("OWN `git worktree add` … never two seats in one tree"). `sed -n '28p' fineract-accounting/dependencies.gradle` = `implementation(project(path: ':fineract-charge'))` — compile edge real; guard is the right response to it | SUSTAINED — fix landed, honest |
| F-7 | planned slice = **4,261 LOC / 49 files (2,185 + 2,076)** | `glaccount` src/main: 25 files / **2,185**; `rule` src/main: 24 files / **2,076**; total **49 files / 4,261** (whole-module find incl. tests would give 51 / 4,429 — the plan's figure is src/main, matches exactly). Module figure 14,314 explicitly disclaimed as slice-sizing | SUSTAINED — fix landed, honest |
| F-8 | R1 = three owner options (docker / manual JDK-25 / foojay); "no in-repo resolver, auto-provision NOT a config flip" | `grep -ri foojay settings.gradle build.gradle gradle/` = **0**. `build.gradle:418` = `languageVersion = JavaLanguageVersion.of(25)`; host `java -version` = 21.0.12.1 → gap real. Docker daemon UP (ServerVersion 29.1.3); image pull honestly tagged INFERRED | SUSTAINED — fix landed, honest |

Bonus honest-bonus check on the F-7 sizing rationale ("charge pin leans on existing integration/e2e nets"):
charge-touching integration tests EXIST and are numerous — 56 `integration-tests` files with charge in the
filename (ChargesTest, ClientChargesTest, ClientChargeRoundingTest, 20+ LoanCharge*/Chargeback tests);
187 integration-test files mention charge. The cited net is real, not aspirational.

## B. REV 2 new claims — attack results

**B/a — §1 "fineract-accounting is SELF-CONTAINED: its read/write service impls live IN the module."**
REFUTED as written at module level, SUSTAINED for the planned slice. My count: **10** accounting
`*Impl.java` files resident in `fineract-provider/src/main` (`accrual/service/AccrualAccountingWritePlatformServiceImpl`,
`common/AccountingDropdownReadPlatformServiceImpl`, `journalentry/service/` ×6, `productaccountmapping/service/ProductToGLAccountMappingWritePlatformServiceImpl`,
`provisioning/service/ProvisioningEntriesWritePlatformServiceJpaRepositoryImpl`). But for the slice the plan
actually cuts: `find fineract-provider/src/main \( -path '*glaccount*' -o -path '*/rule/*' \)` → **0** accounting
files provider-resident; the cited example `GLAccountReadPlatformServiceImpl` and S2's exemplar
`AccountingRuleWritePlatformServiceJpaRepositoryImpl` (346 l) are both in-module. Does it change a stage's
work, acceptance, or risk? No — S1–S3 operate only on glaccount+rule (all true there), S4 is charge, and R3
caps the study to these slices. Classification: **P3 wording finding (overgeneralization), non-blocking**;
recommend one qualifier: "self-contained **for its glaccount+rule surface**; journalentry/provisioning/accrual/
mapping/dropdown impls are provider-resident — measure before any wider accounting slice." The sentence is a
live trap only for a future slice extension that R3 currently forbids.

**B/b — S3 baseline "7 calls: GLAccount read 3, GLAccount write 1, AccountingRule read 3" + file-list coverage.**
SUSTAINED, exact. Within the 49-file slice, `grep -c 'jdbcTemplate\.'`: `GLAccountReadPlatformServiceImpl` **3**,
`GLAccountWritePlatformServiceJpaRepositoryImpl` **1**, `AccountingRuleReadPlatformServiceImpl` **3** = **7**;
module-wide the count is 17 (closure 2 + financialactivity 2 + provisioning 6 outside the slice) — consistent.
Coverage hunt over the full slice with the broader variant net (`JdbcOperations|NamedParameterJdbc|ResultSet|
PreparedStatement|executeQuery|JdbcTemplate`) found only two extra files inside the 49:
1. `glaccount/jobs/updatetrialbalancedetails/UpdateTrialBalanceDetailsTasklet.java` (151 l) — **inside** the slice
   scope, 9 lowercase-`jdbcTemplate` lines (orchestrator's 9 reproduces), but **zero `jdbcTemplate.` calls**:
   it constructs `new JdbcTemplate(…)` at line 54, threads it through 4 method signatures + 2 call sites, and
   **never invokes a method on it** — I read the full body (l. 60–151); every DB access goes through the JPA
   repositories (`trialBalanceRepository`, `journalEntryRepository`, `trialBalanceRepositoryWrapper`).
   Verdict on "is S3 under-scoped?": **NO** — under the plan's stated acceptance pattern (`grep -c 'jdbcTemplate\.'`
   strictly reduced from 7) the baseline is exactly 7 and the 3-file list is exhaustive; the dead template is an
   **S2 dead-code-removal target** (S2 explicitly lists dead-code removal), worth one S2 note. P4.
2. `rule/starter/AccountingRuleConfiguration.java` — DI wiring (`JdbcTemplate` as a bean-factory parameter,
   the seam mechanism itself), not raw JDBC. Not in scope. P4-note only.

**B/c — S2 exemplar 346 l.** SUSTAINED: `wc -l` = **346**; it is the largest file in the 49-file slice (next
largest service 274 l; the 286-l GLAccountsApiResource is `api/`, untouched by S2). "Largest slice service" is accurate.

**B/d — S4 charge `ChargeReadPlatformServiceImpl` 502 l "14 `jdbcTemplate.` calls".** SUSTAINED: `wc -l` = 502;
occurrences of `jdbcTemplate.` = **14**; lines matching = **14** (all `this.jdbcTemplate.query…`). The number
reproduces by BOTH methods; the orchestrator's 15 used a different pattern (no dot / case) — not a plan defect. WITHDRAWN (my probe).

**B/e — S0 acceptance project names.** SUSTAINED: `settings.gradle:60` = `include ':fineract-accounting'`,
`:66` = `include ':fineract-charge'` — exact spellings; `./gradlew :fineract-accounting:test :fineract-charge:test`
is mechanically well-formed (task execution itself stays S0's gate — no build was run here, per constraints).

**B/f — new-number audit.** All reproduce: 49 files / 4,261 / 2,185 / 2,076 (A row F-7); Spotless 8.10.2 at
`build.gradle:112`, SpotBugs 6.5.4 at `:125` (also Boot 4.1.1 `:116`, Java 25 `:418`); R2 "919k" vs 01's 919,408 —
repo-wide `find . -name '*.java' -exec cat {} + | wc -l` = **919,408**, exact. Charge module 3,946 / 39 and
94 provider `*ServiceImpl` with `jdbcTemplate.(query|execute|update)` also re-reproduced this session.

## C. Verdict basis

Every c1 fix is landed and honest under re-run instruments; every REV-2 number I probed reproduces exactly,
including the one the orchestrator pre-probed (B/a). The sustained findings are wording-level: one P3
(self-containment sentence true for the planned slice, overgeneralized at module level — a trap only for an
R3-forbidden future slice) and two P4 notes (dead JdbcTemplate in the in-scope tasklet → S2 dead-code target;
DI-wiring file not in peel scope). Nothing here changes any stage's work, acceptance, or risk classification.
A competent seat can execute S0→S4 from REV 2 without being misled: S0 names real projects and real blockers
with owner options, S1–S3 operate on a correctly-sized slice with a real non-zero peel baseline and a RED-proof
characterization gate, S4 names both code locations, the real call count, and a compile-edge guard.

# VERDICT: SHIP
