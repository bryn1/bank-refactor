PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# DA-verdict — MC 11125 (display 11121.1.2, [type:review]) — DA gate over T1+T2

Targets refuted-or-survived: `research/01-target-selection.md` (T1) and `research/02-feasibility-plan.md` (T2),
apache/fineract clone @ `355287c8feca747b3cd936a5b1cef1bd7804ffea` (re-pinned this session),
mainframe-bank @ `d3123224`. Every instrument below was re-run by me THIS session (read-only; no installs; no pushes).
Classification: SUSTAINED (I reproduce it) / REFUTED (my instrument contradicts it) / WITHDRAWN (my suspicion failed).

## A. T1 load-bearing claims — instrument re-runs

| # | T1 claim | My measurement this session | Class |
|---|---|---|---|
| 1 | `github.com/Atgeit/teller` 404 | `api.github.com/repos/Atgeit/teller` → 404; additionally `users/Atgeit` → 404 (user absent too) | SUSTAINED |
| 2 | GitLab `atg38/teller38` not publicly readable; 404-vs-403 indistinguishability | API `projects/atg38%2Fteller38` → 404 "Project Not Found"; `groups/atg38` → 404; `users?username=atg38` → 200 with body `[]` (no user); web `gitlab.com/atg38/teller38` and control `gitlab.com/gitlab-org/bogus-control-xyz` both 302→sign_in→final 403; API control bogus-project in EXISTING public group also 404 ⇒ private-vs-nonexistent genuinely indistinguishable anonymously, and "not publicly readable" holds directly | SUSTAINED |
| 3 | `seyhunak/mainframe-bank` has NO license | `git ls-tree -r HEAD --name-only \| grep -i licen` → 0 matches; broader `licen\|copying\|copyright` → 0; GitHub API `"license": null` | SUSTAINED |
| 4 | god files 4,221 / 3,703 / 3,024 | `wc -l` on the three named paths: 4221, 3703, 3024 | SUSTAINED |
| 5 | 94 `*ServiceImpl` in provider/src/main calling jdbcTemplate | grep -rl count = 94 under BOTH T1's (`query\|execute`) and T2's (`query\|execute\|update`) patterns | SUSTAINED |
| 6 | charge 3,946 LOC / 39 files; 2 test files | `find fineract-charge -name '*.java' -exec cat {} + \| wc -l` = 3,946; count = 39; `src/test` = 2 files (plain JUnit, no Spring/DB) | SUSTAINED |
| 7 | accounting 14,314 LOC / 174 files; 3 test files | module java = 14,314; count = 174; `src/test` = 3 files | SUSTAINED |
| 8 | 1,002 `*Test*.java` + 99 `*IntegrationTest*` | find counts = 1,002 and 99 | SUSTAINED |
| 9 | 30 Gradle modules — see F-5 (T2's claim) | T1 itself: repo 919,408 java LOC / 7,047 files reproduced robustly (`cat {} + \| wc -l`); savings 26,482 and provider-main 278,878 reproduced | SUSTAINED |
| 10 | Apache-2.0 via `LICENSE_SOURCE` | head of `LICENSE_SOURCE` quotes "Apache License Version 2.0, January 2004"; LICENSE_SOURCE/LICENSE_RELEASE + NOTICE_* present | SUSTAINED |

T1 findings (not refutations of its conclusion):

- **F-T1a — OVERSTATED (P2), sustained finding.** §2 "Safety net for refactoring: … characterization tests already
  exist for behavior-preservation gates" is misleading for the SELECTED slices: the charge slice has 2 tests
  (domain-only) and accounting 3 (Mockito, service-vs-mocked-repository); the 1,002 files live overwhelmingly in
  `integration-tests/` + loan-domain + e2e. T2 §1 itself corrects this ("slice coverage is thin") — the two docs
  disagree in framing; T2's framing is the correct one.
- **F-T1b — methodology hazard (sustained, became F-4's cause).** Both docs cite `find | xargs wc -l | tail -1`.
  On file sets large enough for xargs to chunk, `tail -1` reports only the LAST chunk. I demonstrated it on this
  clone: the method prints 91,778 for `fineract-provider` while `find … -exec cat {} + | wc -l` gives 331,201.
  Every T1 number I was asked to re-run reproduces — the trap bit T2, not T1.

## B. T2 attack findings

- **F-1 — REFUTED (P1): S3 is vacuous on the charge module — named exemplar refutes the stage premise.**
  `grep -rin 'jdbctemplate' fineract-charge/src/main` → 0 hits; `grep -rlE 'JdbcTemplate|JdbcOperations|NamedParameterJdbc|ResultSet'`
  over the module → 0 files. The named example `…/charge/service/ChargeDropdownReadPlatformServiceImpl.java` (154 l)
  is pure enum-listing code with NO database access. The acceptance check "module `jdbcTemplate` grep count strictly
  reduced" is unmeetable-by-construction from a baseline of 0 (it can only grow, never shrink). The genuine raw-JDBC
  charge material sits OUTSIDE the module: `fineract-provider/src/main/java/org/apache/fineract/portfolio/charge/service/ChargeReadPlatformServiceImpl.java`
  (14 `jdbcTemplate.` calls, 502 l).
- **F-2 — REFUTED (P1): "slices are pre-cut by the module boundary" is FALSE for the first slice.** The charge
  domain's write/read service implementations live in `fineract-provider` (`ChargeWritePlatformServiceJpaRepositoryImpl.java`
  291 l; `ChargeReadPlatformServiceImpl.java` 502 l — both under `fineract-provider/…/portfolio/charge/service/`),
  not in `fineract-charge` (interfaces + domain + api + deserializer + handlers only). A charge-module-bounded S1–S3
  therefore characterizes and cleans the wrapper, not the behavior. (accounting IS self-contained — its read/write
  impls are inside the module — so the premise fails for charge specifically, the slice the plan starts with.)
- **F-3 — overstated (P2), sustained.** S2 "split oversized service/deserializer classes": the charge module's largest
  files are `Charge.java` 761 l (a JPA entity — splitting an entity is different work than splitting a service) and
  the deserializer 509 l; there is no "oversized service class" in the module (largest service file 154 l). The stage
  has at most one real referent (the 509-l deserializer).
- **F-4 — REFUTED (P2): the headline provider-size figure is wrong.** T2 §1 "fineract-provider (91,778 LOC / 2,776 files)":
  robust recount = 331,201 java LOC module-wide, 278,878 under `src/main` (T1's figure, which reproduces); 2,776 files
  is right. 91,778 is the F-T1b xargs artifact — a ~3.6× understatement, tagged "VERIFIED", and it directly contradicts
  T1's 278,878 with no reconciliation between the two docs.
- **F-5 — REFUTED (P3): "30 Gradle modules (`settings.gradle`)" does not reproduce.** Measured: 36 `^include` lines
  (plus dynamic `custom:*` discovery), 37 build.gradle files at depth ≤2, 33 top-level `fineract-*` dirs. No variant gives 30.
- **F-6 — SUSTAINED (P2): the S4 ∥ S2/S3 parallelism claim is unsafe as written.** (a) Internal contradiction: S4's
  own row says depends-on "pattern proven in S1–S2", which forbids overlap with S2; (b) build-graph coupling:
  `fineract-accounting/dependencies.gradle:28` — `implementation(project(path: ':fineract-charge'))` — S4 compiles
  against the very artifact S2/S3 mutate; "zero file overlap" (true, disjoint dirs) is not zero build coupling;
  (c) two concurrent seats on one pinned clone contend on the git index/branch; no worktree guidance appears anywhere.
- **F-7 — SUSTAINED (P2): seat-day sizing overstated at the level it is stated.** T1 §2 prices accounting at "14,314
  LOC ≈ 2–3 seat-days" with 3 thin tests; characterization-from-3-tests is the expensive part and seed test
  `GLAccountWritePlatformServiceJpaRepositoryImplTest` is Mockito-vs-mocked-repository (pins service logic, not DB
  behavior — imports verified). T2's actual S4 slice `glaccount`+`rule` measures 4,261 LOC (2,185 + 2,076), which IS
  plausibly 2–3 seat-days — but then the 14.3k sizing claim doesn't size the planned work; the docs should say which.
- **F-8 — SUSTAINED (P2): hidden sub-gap behind R1 — "Gradle auto-provisioned toolchain" is not configured in-repo.**
  No `foojay` (toolchain resolver) in `settings.gradle`/`build.gradle`/`gradle/`, so Gradle cannot auto-download JDK 25
  as-is; S0 option-1 = a repo edit (add the resolver) or manual install. R1's two options are not both config-flips.
  The rest of the second-gap hunt came up CLEAN (see WITHDRAWN).
- **F-9 — WITHDRAWN: "T2 assumes a T1-INFERRED fact as true".** T2 keeps "no build executed", holds every green claim
  INFERRED behind S0, and tags Spring Boot 4.1.1 as carried-INFERRED when it is actually VERIFIABLE
  (`build.gradle:116 'org.springframework.boot' version '4.1.1'`). Conservative, not dishonest. Lineage claim not load-bearing in T2.
- **F-10 — WITHDRAWN (strawman check): A/B/C are real strategies, honestly costed** (A's and C's "against" cells are
  real constraints, not filler). "B spine + C's write-up format" is a coherent pick of B's execution + C's doc format,
  with A explicitly barred from the schedule — not a hedge. The RECOMMENDATION survives.
- **Mechanical-check audit (what IS genuinely mechanical, re-verified):** R1 exact (`build.gradle:418` is
  `languageVersion = JavaLanguageVersion.of(25)`; host `java -version` = 21.0.12.1 → build gap real); Spotless 8.10.2
  (build.gradle:112), SpotBugs 6.5.4 (:125); 27 workflows incl. `build-quality-checks.yml`,
  `verify-api-backward-compatibility.yml`, `verify-liquibase-backward-compatibility.yml` (R4 tripwires exist);
  S2's guard path `fineract-charge/src/main/java/org/apache/fineract/portfolio/charge/api/` exists; S4 seed test exists;
  e2e "336 files" reproduces as the .java-only count (506 total files); `ClientLoanIntegrationTest.java` = 8,198 l;
  loan = 69,025 LOC; docker daemon UP + `docker-compose-development.yml` present + Gradle wrapper 9.7.1 (modern) —
  no second hard blocker found beyond F-8's sub-gap; S0's check (`:fineract-charge:test` green) is mechanical and the
  2 charge tests need no DB (plain JUnit) — S0 is sound once JDK 25 lands.

## C. What survives vs what must go back

Survives intact: the target (Fineract @ pinned SHA), license analysis, the A/B/C option space, the B-spine +
write-up-format recommendation, S0-as-first-blocker (R1), the guardrails R2–R7, and nearly every T1 number.

Must go back to the producer (T2 stage table + two figures):
1. Re-scope or delete S3 — its premise and its named exemplar are refuted (F-1); the honest peel target is the
   provider-resident `ChargeReadPlatformServiceImpl` (14 `jdbcTemplate.` calls) — which breaks the "module = slice"
   premise (F-2) the whole B-spine sequencing rests on for the FIRST slice; accounting (self-contained) is the safer
   first slice, or charge must be defined as charge-module + provider `portfolio/charge` files with a matching
   acceptance path.
2. Fix "91,778 LOC" → 278,878 (src/main java) or 331,201 (module-wide), and state the method (F-4).
3. Fix S4's parallelism cell: "∥ S3 only, own git worktree, accounting→charge compile-dep must be frozen or the slice
   reordered" (F-6).
4. Fix the module count (36 include-lines, or state what 30 meant) (F-5); add foojay/resolver step to S0 (F-8).

These are corrections the plan's own author can make from this document; none changes the recommended strategy, but
two P1-class defects stand inside the first-work stages, so this cannot ship as-is.

# VERDICT: FIX
