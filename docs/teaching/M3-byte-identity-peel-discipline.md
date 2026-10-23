# M3 — Byte-identity peel discipline

**The law:** a peel moves plumbing, never behavior. SQL text, RowMappers,
extractors and clause-builders cross the seam BYTE-IDENTICAL, no test file is
edited at a peel commit, and the suite counts stay flat. The claim is not a
promise — it is a set of probes and red-proofs run at every commit boundary.

## 1. What the discipline protects

Characterization pins are the contract the peel must preserve. A test edited
during a peel destroys exactly the signal that distinguishes "the behavior moved"
from "the behavior changed" — so the study made test-editing during peels not
just discouraged but *checkable*: `git diff --name-only <base>..tip | grep -v
src/test` (scope) and its mirror (zero-test-touch) printed at every stage
(`.audits/20261023-1355-s4b-charge-peel/S4b-evidence.txt`, battery block
"test files untouched in B (068c8f4..83dc366) → 0" — range pinned from the line
as printed (the live doc names its shas; the evidence artifact stays as run).
Pins may move — but only at
FIX commits, each moved pin renamed WITH its decision row in the same work-unit
(the S2 seven-pin rename list, `.audits/20261023-0400-s2-cleanup1/S2-evidence.txt`
"PINS MOVED" block; the cross-repo reading of "same commit" is stated in
`research/04` and `research/05`).

## 2. The literal-set probe ([] missing)

Method: extract every SQL string literal from the pre-peel service, then check
each is still present in the post-peel union (service + repository); and the
reverse, to catch retype drift disguised as a move. Measured results, all
`[]`:

| peel | probe result | evidence |
|---|---|---|
| charge read-side (`7fd7b92`→`83dc366`) | service literals missing = `[]`; repo extras = `['"relax"']` — javadoc text, not SQL, recorded honestly | `S4b-evidence.txt` BYTE-IDENTITY PROBE |
| GLClosure (`6d6a471`) | missing literals: 0 | `S4-cd-evidence.txt` |
| FinancialActivityAccount (`f5d3beb`) | missing literals: 0 | same |
| ProvisioningEntries (`89f6f37`) | missing literals: 0 | same |

The gate then ran the same audit independently, both directions: literal sets
EQUAL with missing=[] AND extra=[] — GLClosure 7/7, FinancialActivityAccount 5/5,
ProvisioningEntries 28/28 (GW = `/home/claudecode/Kassan/.audits/202610231204-54dc1352/out/TEST-verdict-c4.md`
GATE 2a). Teaching point: run the probe BOTH ways — one direction proves nothing
was lost, the other proves nothing was quietly rewritten.

## 3. Mapper byte equality, in bytes

Mappers are the other half of the moving mass. Gate GATE 2a (same file)
re-extracted all six mappers from before/after the peel and found them
byte-equal with sizes printed: `GLClosureMapper` 2211 B,
`FinancialActivityAccountMapper` 1745 B, `LoanProductProvisioningEntryMapper`
3266 B, `LoanProductProvisioningEntryRowMapper` 3260 B,
`ProvisioningEntryDataMapper` 1748 B, `ProvisioningEntryIdDateRowMapper` 1395 B.
Planted-mutation restores are likewise byte-verified (sha256 of the restored file
printed against the committed copy — TEST-c4 GATE 2b–2d). Byte equality, not
"looks the same".

## 4. Planted-mutation red-proofs — each flips exactly the named pin

Green-after-a-move is only trusted after the move has been shown capable of
going red. Every peel stage planted ONE mutation in a disposable build copy,
captured which tests flipped, and restored. The discipline signature is
"N named pins, nothing else":

| stage | planted mutation | flipped — EXACTLY | evidence |
|---|---|---|---|
| S2 (fix round, not a peel) | revert the C1+C2 validator hunks | the 5 fix-proving pins, 40 others green | `S2-evidence.txt` RED DISCIPLINE (`red-check.log`) |
| S3 F-3 | invert demotion-guard disjunct `!isEmpty() \|\| tx` → `isEmpty() && tx` | `demotingHeaderToDetailWithChildrenAndTransactionsIsRejected` RED; positive control GREEN | `S3-evidence.txt` F-3 (`f3-red-capture.log`) |
| S3 executor gate-fix | `setObject(i+1)` → `setObject(i)` bind-index off-by-one | EXACTLY the two bind pins (`aNullParameterBindsAsNull…` + `positionalParametersBindInOrderOneBased…`), 4 others green — one behavior, its two named pins | `S3-evidence.txt` B5 (`f5-executor-red-capture.log`) |
| S4a | write-impl :147 `isActive()` → `true` | exactly the S-W3 suspect pin | `S4a-evidence.txt` RED-PROOF |
| S4b | `where c.is_deleted=false ` → `...=TRUE ` in `ChargeSqlReadRepository` | exactly `retrieveAllChargesSendsTheSoftDeleteFilteredNameOrderedChargeSchema` (14/1) | `S4b-evidence.txt` PEEL RED-PROOF |
| S4c | `" and glClosure.id = ?"` → `" and glClosure.id=?"` (spaces removed) | exactly `retrieveByIdUsesTheIdPredicateAndMapsEveryFieldOfTheRow` (4/1) | `S4-cd-evidence.txt` PEEL RED-PROOF |
| S4 gate TEST-c4 | `like` → `ilike` in a provisioning literal | exactly `provisioningEntryDataByDateAppendsPercentWildcard…`, other 99 green | GW TEST-verdict-c4 GATE 2b |

Byte-exact SQL pins are why a one-space mutation is detectable: the pins compare
the captured string, so *any* drift — alias, case, spacing — names itself.
Byte-identical GREEN output across DIFFERENT mutations would be the tell that the
probe is dead; here each mutation flipped a different, pre-named test.

## 5. Zero-test-edit + steady counts, boundary by boundary

S4b (charge read-side, route wrapping the SAME template instances, M4):
**zero test files touched in the entire peel range**; every boundary green at the
same numbers — charge 83/8, accounting guard 84/12, provider 31/4
(`S4b-evidence.txt` BOUNDARIES; scope battery: the whole delta = 2 provider main
files). S4c (accounting converge, mechanism swap): a peel may touch exactly ONE
test line per test class — the construction helper that wires the new repository
(`newService()`); assertions byte-identical, captured by the PIN-MOVEMENT AUDIT
("every C pins-commit contains ONLY test files; every peel-commit changes exactly
{3 main repos + 3 services + the ONE newService() helper line per test class}",
`S4-cd-evidence.txt`).

The count law falls out of the pin-first order (M1): counts rise ONLY at
pins-commits and hold flat across peels —

```
pre-C            accounting 84/12 | charge 83/8 | provider 31/4
pins C1 (+4)  → 88/13 after peel C2 (flat)
pins C3 (+3)  → 91/14 after peel C4 (flat)
pins C5 (+6)  → 97/15 after peel C6 (flat)
S4-D (+3)     → 100/15 at tip 618896c   charge 83/8 + provider 31/4 NEVER moved
```

(`S4-cd-evidence.txt` BOUNDARIES; final gate re-measured on a fresh copy:
100/15 + 83/8 + 31/4, GW TEST-verdict-c4 GATE 1.) A count that drifts at a peel
commit is the suite telling you behavior crossed the seam.

## 6. Carry-away rules

1. Peel = literal move. Prove it with a two-way literal-set probe ([]/[]) and
   byte-equal mapper extraction, not with "tests pass".
2. Freeze the tests at peel commits — literally, via `git diff --name-only`;
   the ONE allowed exception is construction wiring when pins were captured
   below the mechanism (M1 §3).
3. Each stage plants one mutation and demands an exactly-N failure list naming
   the expected pins; restore byte-verified. An un-flippable pin is not a
   guard (M2).
4. Watch counts at boundaries: up at pins-commits, flat at peels, and the
   guard suites of untouched modules steady — the accounting guard 84/12 was the
   charge peels' compile/regression witness the whole way (S4b boundaries).
5. Run the probes at every boundary AND at the tip; the final gate re-runs them
   from a fresh copy, because "worked in the producer's copy" ≠ "reproducible"
   (GW TEST-c4 ran on a fresh `cp -a` at `b416474`, porcelain 0 at copy).

Evidence level: all rows TESTED per the cited runs (each evidence file ends
`VERIFY_EXIT=0`; the GW verdicts carry their own `# VERDICT:` lines). What
byte-identity does NOT prove: that the SQL RUNS against PostgreSQL — DB-1..DB-5
(`research/05`) are the named INFERRED boundary, owned by M5's reading list.
