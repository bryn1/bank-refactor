# M1 — Characterization before refactor

**The law:** you may not change behavior you cannot first pin down. Every stage of
this study put a green characterization suite on the UNMODIFIED pin before it
touched a line — and proved the suite can go RED before trusting it. The plan
made this acceptance, not folklore: `research/02-feasibility-plan.md` §3 S1 row
("characterization green on unmodified code AND proven RED (one planted-bad run
flips it)") and §4 R7 ("a characterization suite counts only after one
planted-bad run proves it goes RED").

## 1. Why the safety net had to be built, not inherited

Measured at the pin (`research/02` §1, "Test safety net" row, VERIFIED there):
the repo has 1,002 `*Test*.java` files, but the slices in play are bare —
accounting had 3 test classes (one Mockito test pinning service-vs-mocked-
repository, NOT DB behavior), charge had 2 domain-only JUnit tests with no DB.
"Characterization is the expensive part of every slice" — the plan's own words —
and the study confirmed it: the S1 slice (49 files / 4,261 LOC) grew the
accounting suite from **3 classes / 5 tests at the pin** to **10 classes / 69
tests** at fork `8444614` (`.audits/20261023-0045-s1-accounting-baseline/S1-evidence.txt` §3).

The slice baselines as they grew, one commit per sub-concern:

| suite | pin baseline | S1/S4a tip | commits |
|---|---|---|---|
| `:fineract-accounting:test` | 3/5 | 11 classes / 74 tests | `8444614` (69/10) → `3150b9d` (+5 gate-fix pins) |
| `:fineract-charge:test` | 11/2 | 83 tests / 8 classes | `5cf5598` (→46/3) → `a39cf6e` (→71/7) → `6c92ff2` (→83/8) |
| provider `portfolio.charge.*` (targeted) | — | 31 tests / 4 classes | `4df027c` 14 + `292fcae` 12 + `d4b1250` 3+2 |

Boundary counts and per-class sums: `.audits/20261023-0045-s1-accounting-baseline/S1-evidence.txt`,
`.audits/20261023-1333-s4a-charge-baseline/S4a-evidence.txt` (TEST COUNTS block).
(Honesty rider worth teaching with: two fork commit SUBJECTS at `4df027c`/`292fcae`
say "11 tests"; the classes carry 14 and 12 `@Test` — the evidence XML sums are the
truth and the LEDGER carries a correction line. Evidence beats commit messages.)

## 2. The three DB-free capture idioms

No dev database existed (S0 recorded boot tests excluded). The whole net is
DB-free JUnit5 + Mockito, and it stands on three idioms, each measured out of
`S4a-evidence.txt` (MECHANISM block) and `research/05` S4a intro:

1. **Mocked template = SQL capture.** `JdbcTemplate` /
   `NamedParameterJdbcTemplate` are mocked; every call captures the SQL string
   and the `RowMapper`/`ResultSetExtractor` argument. The pins then assert the
   captured SQL BYTE-EXACT (schema text, in-clause wrapping, EXISTS guards) and
   the binds positionally. This is what later makes the peel provable: the same
   captured strings must re-appear unchanged from behind the new seam (M3).
2. **Dynamic-proxy ResultSet drives the captured mapper.** A plain stub answering
   one method at a time cannot keep `JdbcUtils.getResultSetValue`'s dispatch
   consistent, so the captured `ChargeMapper` is driven against a
   `java.lang.reflect.Proxy` whose label-based and index-based reads answer
   **in lockstep, including `wasNull()`** — required because Spring 7's
   `JdbcUtils` dispatches INT/LONG reads to the typed index getters and then
   consults zero-arg `wasNull()` (established from spring-jdbc-7.0.9 bytecode;
   recorded in `S4a-evidence.txt`).
   The lesson: mock the CONTRACT the code under test actually calls, not your
   guess of it — read the dependency's dispatch path when a stub goes green for
   the wrong reason.
3. **Business-date ThreadLocal.** Date paths (`DateUtils.safeMonthDay`) read a
   ThreadLocal business date; tests seed it with the repo's own
   `EnricherTest` idiom instead of freezing clocks (same evidence block).
   Legacy code's ambient state must be established, not worked around.

## 3. Pin FIRST, then peel — the order that makes peels cheap

The law in operational form: **capture characterization pins BELOW the execution
mechanism, commit them alone, and only then move code.** If the capture sits at
the DataSource/Connection/PreparedStatement/ResultSet seam, a peel to a new
mechanism cannot touch a single assertion — only the test's construction helper
may change. The S4c converge wave is the clean example (`research/05` S4-T row;
`.audits/20261023-1720-s4-cd-converge-debts/S4-cd-evidence.txt` PIN-MOVEMENT
AUDIT block):

| cluster | pins commit (test files ONLY) | peel commit (main files + ONE helper line per test class) |
|---|---|---|
| GLClosure (4 pins) | `1131677` | `6d6a471` — probe 10→8 |
| FinancialActivityAccount (3 pins) | `1900691` | `f5d3beb` — probe 8→6 |
| ProvisioningEntries (6 pins) | `8ab4135` | `89f6f37` — probe 6→0 |

`8ab4135` → `89f6f37` read side-by-side is the teaching pair: the pins-commit
characterizes six dotted-`jdbcTemplate.` call shapes at the JDBC seam (dateDiff
expanded via the real `DatabaseSpecificSQLGenerator`, grouped-sum schema,
like-wildcard, two-query chain binds); the peel-commit moves the SQL + mappers
byte-identical behind a repository on the module executor, and **every one of the
13 S4c pins passes unchanged** — the suite count only ever went UP at
pins-commits, never at peel-commits (`S4-cd-evidence.txt` BOUNDARIES; M3 owns the
count discipline).

Had the pins been captured at the service's own `jdbcTemplate.` mock instead
(above the mechanism), the same peel would have forced every stub to be
re-stubbed — each re-stub a place a real behavior change could hide. That is the
whole reason for the order.

## 4. A baseline suite must have gone RED once

Ghost completion (plan R7) is killed by a planted-bad run against the build copy,
captured, restored. Three captures show the range:

- **Boundary mutation** (S1): `notExceedingLengthOf(200)`→`(300)` flips exactly
  `createRejectsNameLongerThan200` — captured in `red-proof-full.log`
  (`S1-evidence.txt` §4, restore = fresh byte-parity copy).
- **Suspicious-suspect mutation** (S4a): `if (!chargeForUpdate.isActive())`→
  `if (true)` at write-impl :147 flips EXACTLY the S-W3 suspect pin
  `reactivationOfAUsedChargeRunsNoUsageGuardAtAll`, all 11 siblings green
  (`S4a-evidence.txt` RED-PROOF block, restore byte-verified).
- **Construction mutation** (S1 gate fix): swapping one dispatch-array constant
  goes red on the tag-resolution pin (sanity-red, `fix-cycle-sanity-red.log`,
  `S1-evidence.txt` §6).

Mutations lived ONLY in disposable `cp -a` build copies; the fork porcelain and
the pin clone `355287c8` porcelain stayed 0 at every one of these runs
(every evidence file's END-STATE block re-proves this).

## 5. Carry-away rules

1. Measure the slice and the safety net BEFORE writing either — the plan's S3
   near-miss (an acceptance check that was unmeetable-by-construction because the
   chosen slice held ZERO `jdbcTemplate.` calls, `research/02` §1 F-1 lesson) is
   what happens when nobody measures first.
2. Characterize DB-free by capturing at the seam the code actually calls; where
   the seam is a library's dispatch, read the library.
3. Pins commit alone, below the mechanism, before any peel. Then peels cost zero
   test edits — and that zero is checkable (M3).
4. A suite that has never gone RED is a decoration. Plant the bad thing, capture
   the named failure, restore byte-verified.

Evidence level: everything above is TESTED/VERIFIED per the cited evidence files
(each ends `VERIFY_EXIT=0`). What these pins do NOT prove is the DB-gap list
(DB-1..DB-5, `research/05`), INFERRED by design.
