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
- T2 card 11124 [type:plan] feasibility plan → research/02-feasibility-plan.md (after T1)
- T3 card 11125 [type:review] DA gate over T1+T2 → .audits/<run>/out/DA-verdict.md (after T2).
  Ruling vs leaf request for a DA pass over T1 alone: ONE gate (T3) over BOTH artifacts —
  one mechanism per concern; leaf's seed claims (Teller38-404, mainframe-bank license, Fineract
  counts, seat-day sizing) are folded into the T3 brief.

## Cycles used
- phase-1 DA loop: 0/3

## STATUS: PHASE 1 RUNNING 2026-10-22 (hermes-standby, parent 11122)
