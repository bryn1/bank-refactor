# LEDGER — bank-refactor (OSS legacy bank backend refactor feasibility)

MC: anchor #11121, parent 11122 (display 11121.1). Workspace: /home/claudecode/Kassan/bank-refactor (git; compat symlink /home/claudecode/bank-refactor → it, MC 11144).
Owner choice (dsh chat 2026-10-22, option selected): "Public/OSS legacy code as a practice
target (Recommended)" — study + demonstrable clean-code refactor skill. Public sources only;
R2 guardrail satisfied by construction.

## Goal
docs/FEASIBILITY.md — cited feasibility report: chosen OSS target + rejected alternatives,
>=2 modernization strategies with trade-offs + recommendation, staged sequence, risks.
Gated by DA `# VERDICT: SHIP`.

## Children (phase 1)
- T1 card 11126 [type:research] target selection — CANCELLED 2026-10-22: filed DoD probe
  `^[0-9a-f]{64}$` unmatchable (sha256sum output carries the filename → `$` never lands).
  Lesson: research-card DoD probe = `ls -ld <abs> => <name>\.md$` (existence-stable);
  bind content via closer `PROBE: sha256sum … => ^<hex>` in close evidence; never write the
  `COMMIT=<sha>` token unless the repo is registered in knut projects.yaml.
- T1b card 11127 = same artifact research/01-target-selection.md (committed 29a7d72, author
  research (11126)) — CLOSED, knut round-1 **PASS / VERIFIED**. Verdict: apache/fineract.
  KB fb61034b647ab8f8ba14025284f9490a60e7ed7d; vault note Research/MC-11126-…-target-selection.md.
- T2 card 11124 [type:plan]: artifact research/02-feasibility-plan.md committed 320db66
  (author plan (11124)); child stalled then was woken and authored in-place (no re-dispatch).
  Orchestrator re-measured: validator 3,024 l EXACT, host JDK 21.0.12 vs required Java 25
  (build.gradle:418) EXACT, sha 09cee271 bound in evidence. Close attempted → knut routed to
  review_close.py (MC 10062.139.21): docs-class cards ([type:plan|review|design|tech-writer])
  close ONLY on DA-verdict SHIP/PASS + DONE.md under card-<N>.out. So 11124 closes AFTER T3
  SHIP (verdict copies + DONE.md rows are the wiring). R1 flag for owner: JDK-25 toolchain call
  gates any executed-build claim.
- T3 card 11125 [type:review] DA gate over T1+T2 — SPAWNED 2026-10-22 ~16:45, run dir
  .audits/20261022-1447-da-plan-gate/out/DA-verdict.md (copies mandated into
  card-11124.out/ and card-11125.out/). Ruling vs leaf request for a DA pass over T1 alone:
  ONE gate over BOTH artifacts — one mechanism per concern; leaf's seed claims folded into brief.

## Cycles used
- phase-1 DA loop: 1/3 — c1 FIX 2026-10-22 (DA-verdict.md sha 5cc3c6ed…, copies in
  card-11124.out/ + card-11125.out/). T1 survived all refutations; T2 got 5 sustained findings
  (S3 vacuous — charge/src/main has ZERO jdbcTemplate, exemplar 154 l pure enum; charge IMPLs
  live in provider — ChargeReadPlatformServiceImpl ~15 jdbcTemplate lines; provider LOC 91,778
  wrong → 278,878 src/main; 36 modules not 30; S4 ∥ unsafe — accounting compile-depends on
  :fineract-charge @dependencies.gradle:28; sizing overstated; no foojay resolver → JDK-25 is
  a real install decision). Orchestrator re-measured ALL five: SUSTAINED. FIX round sent to the
  plan producer (a081ab16, in-place, with the verdict path). c2 gate re-spawns after the revised
  artifact; c2 verdict file DA-verdict-c2.md (review_close picks newest by mtime).
- FIX round 1 delivered: rev 2 committed a68e370 (sha fe92cb2e…, 74 lines). First slice =
  accounting glaccount+rule (4,261 = 2,185+2,076, re-verified); S3 baseline 7 calls/3 files
  (3/1/3) reproduced EXACTLY by orchestrator occurrence count; provider 278,878 / 36 modules
  held; R1 = 3 owner options (docker JDK-25 build recommended).
- c2 gate SPAWNED 2026-10-22 (fresh DA child ba8624ea, run dir ~20261022-1535-da-plan-gate-c2;
  verdict DA-verdict-c2.md; c1 file untouched). Pre-seeded items for c2: §1 "accounting is
  self-contained" module-level FALSE (10 provider-resident accounting Impl files: journalentry/*,
  provisioning/productaccountmapping writes, accrual, dropdown) while PLANNED slice glaccount+rule
  IS fully in-module (VERIFIED: 0 provider files; AccountingRuleWrite...Impl in-module);
  UpdateTrialBalanceDetailsTasklet (9 jdbcTemplate lines, glaccount path) may under-scope S3;
  14-vs-15 call count ChargeReadPlatformServiceImpl.

- c2 gate RETURNED **SHIP** 2026-10-22 (DA-verdict-c2.md sha b03b1592…, run dir
  .audits/20261022-1535-da-plan-gate-c2/out/, DONE.md written there by orchestrator).
  Orchestrator verified c2's new claims EXACT (tasklet 9 refs/0 calls; 56 charge integration
  files; verdict tail). Cards 11124+11125 closed via `mc-job review` manifest verb → terminal
  `reviewed` (verdict files bound by sha). LESSON-PAIR (ledger): (1) docs-class cards never close
  via mc-job close (knut leg refuses, exit 4) — use `mc-job review <id> <manifest>` naming
  DA_VERDICT= + DONE_MD= ABSOLUTE paths (dispatcher wipes card .out on close attempts; pre-
  placed files get destroyed — manifest paths survive); (2) `reopen` requires a reason arg;
  (3) closer probe `=> ^<hex>` NEVER with trailing $ (filename echo) — the rule bit twice,
  both caught pre-close (11127) and at close (11122, fixed same turn).
- PHASE 1 DELIVERABLE: docs/FEASIBILITY.md (sha 220b4066…) committed; parent 11122 closed to
  completed_unverified, knut round-1 verification RUNNING at time of writing (knut-dispatch
  --card 11122 --wait 480 to re-poll). Verdict: FEASIBLE — B spine + C format, S0–S6, single
  blocker R1 (JDK-25: docker build recommended). Owner decisions §6 of FEASIBILITY.

## STATUS: PHASE 1 CLOSED 2026-10-22 (hermes-standby, parent 11122; awaiting knut round back)
- 2026-10-22 MC 11144 — relocated into session workspace Kassan/ (whole-dir mv, HEAD 07b2708 preserved; symlink at old path for immutable probes)

## PHASE 2 (funded 2026-10-22, owner: "go with recomended then start")
- 2026-10-22 MC 11147 — S0 GREEN (knut VERIFIED): docker JDK-25 harness (eclipse-temurin:25-jdk + gradle 9.7.1) runs :fineract-accounting:test + :fineract-charge:test green (16 tests/0 fail, 2m10s) on a cp-a copy of the pin; no dev DB needed for these 5 classes (boot tests excluded — recorded). CARRY to all stages: shallow tagless pin => -e VERSIONING_DISABLE=true. Evidence: .audits/20261022-2156-s0-toolchain/. CORRECTION (orchestrator): close-11147.txt said "LEDGER row appended same commit" — child commit carried only the run dir; this row + WORKSPACE.md fork line land in the fix commit.
- Working artifact repo for S1-S5: ../fineract-fork (own git repo, clean fork of pin @355287c8; refactor + characterization-test commits live there = the phase-2 deliverable; pinned clone stays byte-identical reference).
- 2026-10-23 MC 11150 — S1 BASELINE (characterization + slice map): map research/03-s1-accounting-map.md (rev 2) + evidence .audits/20261023-0045-s1-accounting-baseline/ (S1-evidence.txt VERIFY_EXIT=0; runs bb62c44 + 74c6b96 + this fix commit). Tests fork study: 8444614 (7 files, 69 tests/10 classes) + gate-fix 3150b9d (tag-resolution + usage-guard pins; **74 tests / 11 classes / 0 fail**, docker JDK-25 harness VERSIONING_DISABLE=true; red-proof createRejectsNameLongerThan200 + sanity-red dispatch pin captured; pin clone untouched). Suspects: 12 pinned (14 sites) + 2 doc-only, §6 union. Gate cycle-1: TEST PASS (2 independent mutants caught), DA FIX, ARCH FIX — fix items F-1..F-3 (DA) + F-1..F-4 (ARCH) land here; GATES CLOSED 2026-10-23: TEST PASS (c2; +3rd mutant class tag-swap), DA SHIP (c3; c2 caught fix-round-created §5 metric defect), ARCH PASS (c2; every table row independently recounted exact). 3 cycles of 3. JUDGED final 0ed007c0.
- 2026-10-23 MC 11153 — S2 CLEANUP ROUND 1 (behind fixed interfaces): decisions in research/04-s2-suspect-decisions.md — **FIX 6.1/6.3/6.4 (dead TAGID validator wired; negative/zero tagId now rejected at command layer; core 0L-collapse frozen/untouched), 6.5 (anyOfNotNull guard = full updatable set — incl. DA-pass completion to the nine params, manualEntriesAllowed), 6.6+D-2 (HEADER->DETAIL demotion guard; dead GLAccountInvalidUsageException DELETED — concern dup), 6.8 (rule changes-map = applied state), 6.9 (fresh-entity NPE null-safe), D-1 (rule-owned officeId enum, zero-delta)** / **KEEP 6.2, 6.7, 6.10, 6.11, 6.12** (banking-safety/compat rationale per row). Fork study: 7 commits 92b216b b34f115 50673b7 d48327f 9904822 07732e8 1aa4566 (one per suspect/cluster; 7 pins moved WITH their rows, same commits; 3 positive pins added). **77 tests / 11 classes / 0 fail** green at every commit boundary + final (docker JDK-25, VERSIONING_DISABLE=true); planted-red run reverts C1/C2 in build copy -> exactly the 5 fix-proving pins fail, rest green; self-DA pass at handback caught + fixed the 6.5 guard-set incompleteness (1aa4566) and REFUTED the 6.4 echo-client risk (read model exposes tagId as CodeValueData/null, never 0). End-states VERIFIED: fork porcelain 0, `355287c8..study` files all under fineract-accounting/src, pin clone porcelain 0. Evidence .audits/20261023-0400-s2-cleanup1/S2-evidence.txt VERIFY_EXIT=0. **GATES CLOSED 2026-10-23:** TEST PASS (3 own mutants, all named) / DA SHIP (fresh finding F-2 P3: 6.8 compound-path applied-but-unreported -> S3 scope) / ARCH c1 FIX -> c995634 -> c2 PASS. Final JUDGED 4350ad02. Integrity: report-then-dirty window owned+resolved (P3); doctrine stored.
- 2026-10-23 MC 11163 — S3 JDBC PEEL (data-access peel of the teaching slice + S2 carry-overs): 7 slice sites behind module repos/mappers — GLAccountRead x3 -> new `glaccount/domain/GLAccountReadRepository`, write disable-guard x1 -> existing Spring Data `ProductToGLAccountMappingRepository.countAttachedProducts` (native @Query), AccountingRuleRead x3 -> new `rule/domain/AccountingRuleReadRepository`; execution seam = new module-local `AccountingNativeSqlExecutor` (mirrors JdbcTemplate's collaborators/contracts; provider-agnostic — module excludes Hibernate) — decision + scope rows in research/04 S3 section (S3-M, S3-scope). SQL+mappers moved BYTE-IDENTICAL. Fork study: 4 commits `416a4ae` `d9a7c41` `d3f3fa2` `51f9817`; :fineract-accounting:test green at every boundary (77/11; **78/11** from the F-2 pass). 1 pin MOVED with its R-2 row (disable-guard re-stub) + 1 NEW pin (R-3, F-2 applied==reported compound path, 51f9817); F-3 planted-red captured+restored (reject-pin RED, control GREEN; f3-red-capture.log). End states VERIFIED: slice jdbc grep 0 x3, module remaining EXACTLY the 10 non-slice sites (list printed), fork porcelain 0, pin clone @355287c8 porcelain 0, 355287c8..study all under fineract-accounting/src. Evidence .audits/20261023-0936-s3-jdbc-peel/S3-evidence.txt VERIFY_EXIT=0.
- 2026-10-23 MC 11163 S3 gate-c1 — TEST PASS (78/11 fresh + byte-identity gates + F-2 mutation RED-by-name; F-T1 P3 pre-existing coverage gap) / ARCH PASS / DA FIX F-A P2 (two-mechanism state + zero-tested executor) -> FIX LANDED: executor unit tests 6 pins fork `a932d91` (planted bind-index mutation flips exactly the 2 bind pins, rest green, restore byte-verified), suite 78->**84/12 green**; S4-T terminus row + S4-D debt rows appended here (research/04 S3 section at the time; moved to 05 at the gate-fix), S4 must decide converge-vs-relax (replace, not coexist). Evidence: c5-green-run.log + f5-executor-red-capture.log in the S3 run dir. **GATES CLOSED 2026-10-23:** cycle-2 TEST PASS (84/12 fresh; both MAIN-source mutations red-by-name) / DA SHIP (F-A closed; pins mirror contract not mock) / ARCH c2 FIX -> 308d6e0 -> c3 PASS (live-pin byte-exact, superseded hash zero-hit). Convergence JUDGED d5376948; card 11163 knut-VERIFIED. Carried: S4-T/S4-D rows + charge-module peel.
- 2026-10-23 MC 11183 — S4a CHARGE SLICE CHARACTERIZATION BASELINE (S4 pass-1, TASK A ONLY; S4b/c/d follow): charge slice pinned AS-IS, zero fixes — fork `study` commits `5cf5598` `a39cf6e` `6c92ff2` (module: deserializer 35, enums/dropdown/API 25, Charge entity 12 tests) + `4df027c` `292fcae` `d4b1250` `068c8f4` (provider: read-service 14, write-service 12, util 3, starter 2 tests). Seams WITHOUT a DB: mocked JdbcTemplate/NamedParameterJdbcTemplate (SQL captured, EXISTS guards byte-pinned), captured ChargeMapper driven off a dynamic-proxy ResultSet (label/index+wasNull lockstep per spring-jdbc 7.0.9 dispatch), ThreadLocal business dates. Boundaries: charge 11/2->46/3->71/7->**83/8**, accounting guard 84/12 at EVERY boundary; provider targeted 31/4 green at tip. Suspects 24 (S-C1..S-C15, S-R1..S-R4, S-W1..S-W4, S-U1) mapped in research/05 "S4 suspect map (S4a rows)"; named DB-gaps DB-1..DB-5 (INFERRED level). Planted-red (build copy): write-impl :147 `isActive()` -> `true` flips EXACTLY the S-W3 pin red, 11 others green, restore byte-verified. Scope VERIFIED: diff a932d91..study test-tree-only (0 main files); fork porcelain 0; pin clone @355287c8 porcelain 0. Evidence .audits/20261023-1333-s4a-charge-baseline/S4a-evidence.txt VERIFY_EXIT=0. **STATUS: S4 OPEN — S4a landed; S4b suspect decisions, S4c peel, S4d gates follow; S4-T (converge-vs-relax) still owed by S4.**
- 2026-10-23 MC 11183 — S4b CHARGE READ-SIDE SQL PEEL (S4 pass-1 TASK B, parent-authorized): 14 JdbcTemplate + 3 NamedTemplate sites of ChargeReadPlatformServiceImpl behind new provider-internal `portfolio/charge/domain/ChargeSqlReadRepository` WRAPPING the same Spring JdbcTemplate instances (route RELAX-in-provider — no new executor, no charge->accounting edge, S3 executor NOT adopted; two-mechanism state named honestly; S4-T accounting terminus still open). Mechanism row research/05 S4-B. Fork `study`: 3 checkpoint commits `7fd7b92` `d666cee` `83dc366` (core / loan / savings-share-WCL-client); SQL+ChargeMapper+in-clause+exclusion clause BYTE-IDENTICAL (literal-set probe [] missing); service jdbc probe 0 / repo 17; ctor + interface frozen; ZERO test files touched (S-R1..R4 + all A pins pass UNCHANGED at every boundary — counts steady charge 83/8, accounting guard 84/12, provider charge 31/4). Peel planted-red: one SQL-word mutation flips EXACTLY the retrieveAllCharges schema pin (14/1), restored. Scope: delta 068c8f4..83dc366 = 2 provider main files only; fork porcelain 0; pin clone porcelain 0. Evidence .audits/20261023-1355-s4b-charge-peel/S4b-evidence.txt VERIFY_EXIT=0. **S4 OPEN — A+B landed; C/D (write-side + gate wave) per orchestrator.**
- 2026-10-23 MC 11183 — S4 gate cycle-1 correction line (ARCH F-B/F-C, P4; what git says): commit subjects `4df027c`/`292fcae` say "11 tests"; the classes carry 14/12 @Test (LEDGER + evidence already said 14/12 — subjects are immutable, left as-is). S4a evidence "7 new classes": git `a932d91..068c8f4` adds **10 new test classes = 6 module + 4 provider**; the module boundary 8 classes = 6 new + 2 pre-existing (the evidence line's "8" is the module class total, its "7 new" is wrong). enums/dropdown/API cluster `a39cf6e` = 25 tests (3+7+8+7; the S4a LEDGER row's "14" corrected in the same round, boundaries 46->71 refute it). Doc routing: research/04 split at 138 L (current fork tip after the gate fixes: `470896d`) -> research/05-s4-charge-decisions.md carries all S4 rows (S4c/S4d read 05 first); ARCHITECTURE index updated.
