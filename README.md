# bank-refactor — legacy bank backend refactor study (OSS practice target)

A staged, evidence-gated study of refactoring a legacy bank backend into clean, idiomatic
code, run against the public practice target [apache/fineract](https://github.com/apache/fineract)
(Apache-2.0). Phase 2 (MC 11145/11146, closed 2026-10-23/24) executed stages S0–S5 on a study
fork (`../fineract-fork`, branch `study`): characterization-first baselines with no DB, byte-
identity SQL peels, one JDBC terminus decision executed, and teaching artifacts distilling the
lessons. Every stage closed on mechanical acceptance through adversarial gate loops.

Where things live:
- `LEDGER.md` — the project state carrier (one dated row per landed decision).
- `docs/FEASIBILITY.md`, `docs/ARCHITECTURE.md` — the phase-1 verdict and the doc kept true.
- `research/01..05` — cited decisions and the suspect/decision ledgers.
- `docs/teaching/` — README + M1–M5: the study as a course, claims pointer-backed.

The companion `fineract-fork` repo carries upstream `apache/fineract` in full, including its
Apache-2.0 `LICENSE` and `NOTICE`; study commits are attributed per profile, and no upstream
history was rewritten. Private study repos; not for distribution.
