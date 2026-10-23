# Teaching artifacts — the bank-refactor study (MC 11200, phase-2 S5)

This directory makes the legacy-refactor study of **apache/fineract @ pin
`355287c8feca747b3cd936a5b1cef1bd7804ffea`** learnable: five modules, each one
measured lesson distilled from what the study actually did and actually measured.
No new facts are introduced here — every claim carries a pointer to an artifact
that was measured in the study (a research file, a fork commit, or an `.audits/`
evidence file). Where an artifact itself labels a claim INFERRED, that label is
reproduced.

## What the study is

- Goal and staged plan: `research/02-feasibility-plan.md` (S0–S6, DA-gated rev 2).
- History of every decision row: `LEDGER.md` (rows S0–S4, GATES CLOSED markers).
- Slice maps + decision ledgers: `research/03-s1-accounting-map.md` (S1),
  `research/04-s2-suspect-decisions.md` (S2+S3 history),
  `research/05-s4-charge-decisions.md` (S4: suspect map, peel mechanism, terminus).
- The code lineage: `../fineract-fork` branch `study` — characterization-test and
  refactor commits; the pin clone stays byte-identical and was never built in.
- Harness: docker `eclipse-temurin:25-jdk`, `-e VERSIONING_DISABLE=true`
  (`S0`: `.audits/20261022-2156-s0-toolchain/`).

## The modules

| Module | Lesson | Anchors (fork `study`) |
|---|---|---|
| [M1](M1-characterization-before-refactor.md) | Characterize before you refactor: DB-free capture seams, and pin FIRST — then peel | `8444614` `3150b9d` `5cf5598`…`068c8f4`; `8ab4135`→`89f6f37` |
| [M2](M2-the-fake-pin-lesson.md) | A pin green under BOTH bug and fix is not a pin — the S-C15 fake pin, caught by the adversarial gate, fixed into the discriminating-follow-up pattern | `6c92ff2` → `470896d`; `8e1de51` `b416474` |
| [M3](M3-byte-identity-peel-discipline.md) | Peel = plumbing only: byte-identity probes, zero-test-edit law, planted-mutation red-proofs that each flip exactly the named pin | `416a4ae`…`a932d91`; `7fd7b92`…`83dc366`; `1131677`…`89f6f37` |
| [M4](M4-two-mechanisms-and-the-terminus.md) | Two mechanisms, honestly named, then one CONVERGED — and why "probe zero" must name its measurement | `a932d91`; `7fd7b92`…`83dc366`; `618896c` |
| [M5](M5-suspect-catalog-reading-list.md) | The pinned-NOT-fixed suspects as a negative-testing reading list: what each pins, why scope discipline left them, owner decisions riding | `292fcae` `4df027c` `6c92ff2` `068c8f4` |

Suggested order: M1 → M3 → M2 → M4 → M5 (M2 after M3 shows why a green peel suite
is still not proof). Each module stands alone.

## Evidence conventions used in every module

- A bare sha (e.g. `89f6f37`) resolves in **`/home/claudecode/Kassan/fineract-fork`**
  (`git log --oneline -1 <sha>`); the pin is `355287c8`. No sha cited here means a
  commit of any other repo.
- Paths starting `.audits/` are in this repo (bank-refactor), one run dir per
  stage, each evidence file ending `VERIFY_EXIT=0`.
- The final S4 gate wave ran at session-workspace level:
  `/home/claudecode/Kassan/.audits/202610231204-54dc1352/out/` — abbreviated
  **GW/** here (TEST/DA/ARCH verdicts per cycle + `DONE.md`).
- Suite shorthand: `83/8` = 83 tests in 8 classes of one Gradle test task,
  0 failures 0 errors, run in the docker JDK-25 harness from a `cp -a` build copy
  (the fork itself is never built in; the pin clone is never touched).
- Evidence levels: claims from evidence files are TESTED/VERIFIED as stated there;
  the five DB-gap rows (DB-1..DB-5, `research/05`) are **INFERRED** — what the
  DB-free pins cannot prove. Every module that leans near that boundary repeats
  the label.

## What these modules are NOT

Not a fineract tutorial, not a design proposal, and not a status report — the
LEDGER and research/05 hold current state and open owner decisions. These are the
transferable lessons, each with the artifact that proves the lesson was learned
the hard way, on measured ground.
