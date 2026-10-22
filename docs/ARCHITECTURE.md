# ARCHITECTURE — bank-refactor

## 1. What this project is

A feasibility + staged clean-code-refactor **STUDY** on public OSS — owner choice quoted
in `LEDGER.md` ("Public/OSS legacy code as a practice target"); not shipping software;
R2 (reverse-engineering) guardrail satisfied by construction (public sources only).
**Status: Phase 1 CLOSED — FEASIBLE, Strategy B spine (+C format), stages S0–S6**; see
`LEDGER.md` STATUS line and `docs/FEASIBILITY.md`.

## 2. The study-repo tree (this directory)

```
bank-refactor/
├── LEDGER.md            single project state — the one ledger, moves with every commit
├── docs/
│   ├── FEASIBILITY.md   the Phase 1 deliverable (cited feasibility report)
│   └── ARCHITECTURE.md  this doc
├── research/            closed cited artifacts: 01-target-selection.md, 02-feasibility-plan.md
├── .audits/<run>/       THE audit home — every DA gate / verify / fix run lands here,
│                        fresh <YYYYMMDD-HHMM>-<slug>/ dir per run, never reused
└── .tmp/                scratch, gitignored, never a deliverable (holds the pinned clones, §4)
```

## 3. Target corpus map (source: research/01 + research/02, cited — not re-measured here)

- Target: **apache/fineract** pinned @ `355287c8feca747b3cd936a5b1cef1bd7804ffea`,
  36 Gradle modules (research/02, DA-c2 re-count SUSTAINED).
- Study slices: `fineract-charge` 3,946 LOC / 39 files (module = interfaces/domain/api
  only, 0 jdbcTemplate — F-1); `fineract-accounting` 14,314 LOC / 174 files (self-contained,
  first slice); provider-resident `portfolio/charge/` 949 LOC / 4 files — the charge
  behavior lives there (F-2), so the S4 slice names BOTH locations.
- God-file exemplars: `CommandWrapperBuilder.java` 4,221 l (S6 capstone),
  `LoanWritePlatformServiceJpaRepositoryImpl.java` 3,703 l,
  `LoanProductDataValidator.java` 3,024 l.
- Stages S0–S6 per `docs/FEASIBILITY.md` §3 (S0 docker dev-DB baseline → S1–S3 accounting
  slice → S4 charge slice → S5–S6; S4 NOT parallel-safe: `:fineract-accounting`
  compile-depends on `:fineract-charge`).

## 4. Pinned clones + roles (under `.tmp/`, gitignored, DISPOSABLE)

- `.tmp/clones/fineract` @ `355287c8` — the working clone the S-stages run against.
  The **pin SHA is the contract, not the dir**: re-clonable anytime (`--depth=1`).
- `.tmp/clones/mainframe-bank` @ `d3123224` — rejected-candidate evidence cited by
  research/01 (no LICENSE ⇒ no derivative permission, disqualified). Same disposable rule.

## 5. Deps / toolchain

- Java **25** required (build.gradle:418) vs host JDK 21 — owner decision **R1 open**;
  docker build recommended (FEASIBILITY §6). No ports, no services, no data stores
  beyond the S0 dev DB (docker-compose).

## 6. Location & history

Moved 2026-10-22 (MC 11144) from `/home/claudecode/bank-refactor` to
`/home/claudecode/Kassan/bank-refactor` (whole-dir mv, history intact); compat
symlink at the old path for immutable board probes — per
`agents/_shared/skills/workspace-convention/SKILL.md` §Session-workspace placement.
