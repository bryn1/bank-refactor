# LEDGER — bank-refactor (OSS legacy bank backend refactor feasibility)

MC: anchor #11121, parent 11122 (display 11121.1). Workspace: /home/claudecode/bank-refactor (git).
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
- phase-1 DA loop: 0/3

## STATUS: PHASE 1 RUNNING 2026-10-22 (hermes-standby, parent 11122)
