# M2 — The fake-pin lesson (S-C15) and the discriminating-follow-up pattern

**The law:** a characterization pin must be satisfiable ONLY by the behavior it
names. An assertion that stays green while the bug is present AND while it is
fixed proves nothing — it is a FAKE PIN, and it silently removes the regression
signal exactly where the future fix will step. This study grew one, caught it
with an independent adversarial gate, and turned the catch into a reusable
pattern.

## 1. The suspect: mutation before validation (S-C15)

`Charge.update()` mutates entity fields and only then runs trailing
`notNull`/period validators, which throw (`research/05-s4-charge-decisions.md`,
suspect-map row S-C15): a rejected update leaves the managed JPA entity
half-updated — JPA may flush what the API already refused. S4a pinned it AS-IS at
fork `6c92ff2` with
`ChargeUpdateCharacterizationTest::aRejectedUpdateHasAlreadyMutatedTheEntityInMemory`.

## 2. The catch: green under bug AND fix

The S4 gate cycle-1 devils-advocate leg re-traced that pin's assertions against
BOTH code states and found all three hold identically in the BUG state and in
the FIXED (validate-before-mutate) state (GW/`DA-verdict.md` — GW =
`/home/claudecode/Kassan/.audits/202610231204-54dc1352/out/`, per README —
finding **F-A, P2**). The discriminating-looking assertion
`assertFalse(followUp.containsKey("feeFrequency"))` was **vacuous**: the follow-up
command it sent never CARRIED `feeFrequency`, so the diff-based changes map could
not report it under either state. Consequences the verdict names: the map row
"pinned" was false, 23/24 suspects were pinned — S-C15 rode UNPINNED — and the
later cleanup pass touching `Charge.update()` would get zero regression signal.

Why this escaped the producer: the pin was green, its planted-mutation sibling
(S-W3) was red-proven, and nothing in a suite run can reveal an assertion that
cannot fail. Detection needed someone re-deriving, per assertion, what code state
makes it false — the gate's whole job.

## 3. The fix pattern: discriminating follow-up (fork `470896d`)

The DA verdict's bounded ask, executed as test-only commit `470896d`: after the
REJECTED update, the test re-SENDS the rejected value — follow-up command
`{"feeFrequency":2,"feeInterval":3}` — and asserts the follow-up changes map
LACKS `feeFrequency`. The logic (from `DA-verdict.md` F-A + the commit message):

- **Bug state (mutate-before-validate):** the rejected first update already set
  `feeFrequency=2` in memory; the follow-up diff (command vs entity) therefore
  reports NO change for it — assertion (map LACKS it) is GREEN.
- **Fixed state (validate-before-mutate):** the entity never took the value; the
  now-complete follow-up command is accepted and DOES report `feeFrequency` —
  assertion is RED.

So the assertion is satisfiable only under pre-throw mutation: it pins the
defect's presence and flips the moment validators move ahead of mutation. That
is what "discriminating" means — the test names the code state it demands.
(Full bug-state logic in `research/05` S-C15 proof column; the follow-up
parameters per the `470896d` commit message.)

## 4. Both-ways red-captures — the pin must have FAILED for real

A discriminator claim only counts when it has been observed failing on the other
state. Both directions were captured in build copies and archived:

- **GREEN at fork, RED under planted fix:** full suites green at `470896d`
  (charge 83/8, accounting-guard 84/12, provider 31/4); in the build copy,
  reordering the feeFrequency throw BEFORE its assignment (same exception type)
  produced `12 tests completed, 1 failed` — exactly
  `aRejectedUpdateHasAlreadyMutatedTheEntityInMemory`, failure message the
  discriminating assertion itself ("the rejected feeFrequency=2 was applied
  BEFORE the throw... expected: <false> but was: <true>"); plant removed,
  `grep PLANT-FIX = 0`, re-run green
  (`.audits/20261023-1435-s4-gate-cycle1-fixes/S4-cycle1-fixes-evidence.txt`).
- **RED first, then GREEN:** the S4-D(2) debit mirror (`b416474`) captured its
  new pin RED pre-fix — `expected: <true> but was: <null>` (flag applied,
  absent from the changes map) — then green post-fix, the whole accounting suite
  100/15 (`.audits/20261023-1720-s4-cd-converge-debts/S4-cd-evidence.txt`
  BOUNDARIES block). The gate then re-reproduced the RED independently: TEST-c4
  mutation MUT(iii) reverted the one-liner and flipped EXACTLY that named pin,
  same failure message, restore byte-equal (GW/TEST-verdict-c4.md GATE 2d).

An honest wrinkle worth teaching: the `-1720-` run log records an intermediate
run on a STALE build copy that still showed the pin RED after the fix; the true
post-fix tree was re-copied and re-run green, and the file says the commit claim
rests on the second run. Claims name the run that proves them.

## 5. Same family, second lesson: per-disjunct pinning (fork `8e1de51`)

The S3 planted-red against the accounting demotion guard made the sibling
weakness visible: the reject-pin stubbed children=true AND transactions=true
together, so deleting EITHER guard disjunct alone would have kept it green —
the pin proved the disjunction only in combination
(`.audits/20261023-0936-s3-jdbc-peel/S3-evidence.txt` F-3 block, recorded as an
S4 candidate). S4-D(1) executed the pin-split at `8e1de51`: children-only and
transactions-only reject pins, main code untouched. Gate proof (GW/TEST-verdict-c4.md
GATE 2c): deleting the children disjunct from the guard flipped EXACTLY
`demotingHeaderToDetailWithChildrenOnlyIsRejected`; the transactions-only pin
and the combined pin stayed green BY DESIGN (the combined case still rejects via
the surviving disjunct — the F-C P3 note).

**Rule:** for a guard of shape `A || B`, a pin that stubs A AND B rejects proves
only `A||B` at one point. Pin each disjunct alone, and pin the positive control
(neither). One disjunct, one discriminating case.

## 6. Carry-away rules

1. For every pin, ask: what code state makes this assertion FALSE? No answer =
   vacuous = fake, no matter how green it is.
2. Pin a defect so its assertion is satisfiable ONLY by the defect state
   (discriminating follow-up: re-send the rejected value **completed** —
   `470896d` pairs `feeFrequency=2` with `feeInterval=3` — so the follow-up is
   an ACCEPTED command once the defect is fixed, **and assert that acceptance
   in the same test** (as `470896d` does on `feeInterval`), then read the LACK
   of the key from the diff-based observable; skip either guard and the fixed
   state can merely throw, and a swallowed throw poses as the LACK you assert —
   the recipe would regenerate the vacuous pin).
   A fix then flips it RED and the fix commit must carry the flip knowingly.
3. Capture the red side ON A PLANTED STATE and archive the transcript; a
   discriminator nobody has seen fail is a rumor (see the two red-captures, and
   fleet law: every gate must prove it can go RED).
4. Disjuncts get one pin each.
5. Independent re-tracing catches what green suites cannot — budget the
   adversarial pass as part of characterization, not after it.

Evidence: `research/05` rows S-C15 / S4-D; fork `6c92ff2`, `470896d`, `8e1de51`,
`b416474`; `.audits/20261023-1435-s4-gate-cycle1-fixes/`,
`.audits/20261023-1720-s4-cd-converge-debts/`, GW `DA-verdict.md` (c1) and
`TEST-verdict-c4.md`. All TESTED; the S-C15 defect itself remains PINNED,
NOT fixed (M5).
