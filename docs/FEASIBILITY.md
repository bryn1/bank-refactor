PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# FEASIBILITY — refactoring a legacy bank backend to modern clean code (OSS practice target)

MC 11121/11122 · owner goal (dsh chat 2026-10-22): investigate refactoring a legacy **bank
backend** into modern, clean-code, best-practice form — target class chosen by owner:
**public/OSS legacy code as a practice target** (employer-prep / demonstrable skill; NOT
shipping software; no proprietary code; R2 guardrail satisfied by construction).
Evidence artifacts (all in this repo): `research/01-target-selection.md` (knut VERIFIED, card
11127), `research/02-feasibility-plan.md` rev 2 (gate-SHIPped, cards 11124/11125 reviewed),
`.audits/20261022-1447-da-plan-gate/` (c1 FIX), `.audits/20261022-1535-da-plan-gate-c2/` (c2 SHIP).

## VERDICT

**FEASIBLE — YES, with the shape below.** One real production banking corpus (Apache Fineract)
admits a staged, measurable clean-code refactor study sized at ~1 seat-day per stage, gated by
the project's own CI (Spotless/Checkstyle/SpotBugs + 1,002 test files). One owner decision gates
the FIRST stage (build toolchain, §R1); nothing else blocks.

## 1. The target (evidence: research/01, gate-refuted and survived)

**Apache Fineract** — real, active, production core-banking platform (Mifos lineage), 919,408
Java LOC, Apache-2.0 (private derivative + public write-up both permitted; keep LICENSE/NOTICE
if code is ever published). Selection re-ran adversarially: every instrument reproduced.
Rejected (verified dead or unfit): ATG **Teller38** COBOL estate — **not publicly readable**
(Atgeit/teller 404; GitLab 404/anon-private); `seyhunak/mainframe-bank` — perfect size but
**UNLICENSED** (disqualified for derivatives); OpenCBS-Cloud — GPL-3.0 but a 2022-era rewrite,
not a legacy corpus; OFBiz — ERP GL, wrong domain, too big; gnoferrari/dukesbank/mifos-platform
— 404 (Mifos legacy lineage IS Fineract).

## 2. Strategy (three options weighed; evidence: research/02 §2)

| | Strategy | Verdict |
|---|---|---|
| A | Upstream-first clean-code PRs into apache/fineract | opportunistic polish only — pace hostage to community review |
| **B** | **Fork-local slice refactor** (pin @355287c8; module-by-module; the repo's own gates = objective judge) | **RECOMMENDED SPINE** — pace control on a real corpus; "clean" is measured, not opined |
| C | Strangler side-by-side rewrite of one slice | too costly as spine; adopted as the per-slice **teaching format** (legacy-vs-clean, same behavior) + optional capstone |

## 3. Staged sequence (each ≈ 1 seat-day; acceptance checks are mechanical; DA-c2 SHIPped)

| Stage | Concern | Acceptance (mechanical) |
|---|---|---|
| S0 | Toolchain + dev DB + green baseline (**owner decision R1**) | `./gradlew :fineract-accounting:test :fineract-charge:test` green on UNMODIFIED clone |
| S1 | Accounting baseline (`glaccount`+`rule`, 4,261 LOC / 49 files): dep/API map + characterization tests incl. DB pins | green on unmodified AND proven RED by one planted-bad run |
| S2 | Cleanup r1 behind existing interfaces (largest file 346 l), dead-code removal (incl. dead `new JdbcTemplate` tasklet) | module gates green; characterization green; zero diff under `api/` |
| S3 | Raw-JDBC peel: the slice's **7 `jdbcTemplate.` calls in 3 files** → repository/mapper seam | gates+characterization green; count strictly reduced from 7 |
| S4 | Charge slice = module (3,946) **plus provider-resident** `portfolio/charge/` (949/4; peel target `ChargeReadPlatformServiceImpl` 14 calls/502 l) | gates + provider compiles + charge integration/e2e (56 files) + `:fineract-accounting:test` compile-edge guard |
| S5 | Teaching artifact per slice (before/after measured metrics) | doc cites re-measured numbers — the only parallelizable stage |
| S6 | Optional capstone: split `CommandWrapperBuilder.java` (4,221 l) behind a facade | gates + command-surface characterization green |

All code stages **serial** (accounting compile-depends on charge, `dependencies.gradle:28`);
concurrent seats would each use their own `git worktree`. Slice order is **measured, not
assumed**: accounting first because its planned slice is fully in-module; charge second because
its behavior straddles two Gradle modules (the c1 gate's P1 catch — see §5).

## 4. Risk register (condensed; full text research/02 §4)

- **R1 (P0 — the only hard blocker): JDK gap.** Build requires Java 25 (`build.gradle:418`);
  VM350 has 21.0.12; no in-repo toolchain resolver → auto-provisioning is NOT a config flip.
  Owner options: **(i) docker-based JDK-25 build — RECOMMENDED** (zero host installs, pinned
  clone byte-identical; image pull INFERRED, daemon UP), (ii) approved manual JDK-25 install,
  (iii) add foojay-resolver (a repo-diverging mechanism addition — discouraged).
  Until S0 lands, every "green" claim stays INFERRED — no slice may be reported done.
- R2 build cost (scope to `:module:test`); R3 scope cap (accounting+charge+capstone only; loan
  domain 69k LOC off-limits without a new ruling); R4 behavior preservation via the repo's own
  api/liquibase-backward-compat workflows; R5 pin discipline; R6 license/attribution; R7 ghost
  completion (characterization suites count only after a proven RED run).

## 5. How much confidence — gate history (the honest part)

Adversarial loop, capped 3, used 2: **c1 FIX** (5 sustained findings — vacuous S3 acceptance on
a zero baseline; charge-not-self-contained; 2 wrong figures; false parallelism; overstated
sizing; all independently re-measured by the orchestrator) → fix round → **c2 SHIP** (every fix
re-run; residual P3 wording note: "accounting is self-contained" is true of the planned slice,
not the whole module — 10 accounting impls live in provider; P4: dead-JdbcTemplate tasklet).
Selection evidence (T1) never moved under refutation. Board: 11127 knut-VERIFIED; 11124/11125
`reviewed` on SHIP sha b03b1592…

## 6. Owner decisions requested (phase 2)

1. **Fund the staged study S1–S5** (~5–8 seat-days on our schedule) — yes/no.
2. **R1 toolchain**: option (i) docker build (recommended) / (ii) host JDK-25 / (iii) resolver edit.
3. Repo disposition when artifacts exist: keep private bryn1, or publish the study write-up
   (Apache-2.0 attribution obligations apply once refactored code is public).
