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
- 2026-10-23 MC 11150 — S1 BASELINE (characterization + slice map): map research/03-s1-accounting-map.md (rev 2) + evidence .audits/20261023-0045-s1-accounting-baseline/ (S1-evidence.txt VERIFY_EXIT=0; runs bb62c44 + 74c6b96 + this fix commit). Tests fork study: 8444614 (7 files, 69 tests/10 classes) + gate-fix 3150b9d (tag-resolution + usage-guard pins; **74 tests / 11 classes / 0 fail**, docker JDK-25 harness VERSIONING_DISABLE=true; red-proof createRejectsNameLongerThan200 + sanity-red dispatch pin captured; pin clone untouched). Suspects: 12 pinned (14 sites) + 2 doc-only, §6 union. Gate cycle-1: TEST PASS (2 independent mutants caught), DA FIX, ARCH FIX — fix items F-1..F-3 (DA) + F-1..F-4 (ARCH) land here; cycle-2 re-judge pending.
