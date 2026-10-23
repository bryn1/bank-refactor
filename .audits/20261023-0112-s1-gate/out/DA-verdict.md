PREMISE: VM350 / qwen3.8-flash-next / Lane E as of 2026-10-23T01:50Z (DA gate, S1 cycle 1, MC 11150)

# DA-verdict — S1 gate cycle 1: characterization baseline + dep/API map

Read-only adversarial gate over: fork `study@8444614` (7 test files, +1144 lines),
`research/03-s1-accounting-map.md` (study@bb62c44), evidence
`.audits/20261023-0045-s1-accounting-baseline/`. All checks below were re-executed or
re-read this session; nothing was built or re-run (concurrent tester respected).

## 1. Characterization integrity — the pinned behavior IS current behavior (VERIFIED)

`git diff --name-status 355287c 8444614` → **7 × A, zero M** (all under
`src/test/java`). No production hunk exists between pin and study: no silent "fix"
was possible (VERIFIED). Every suspect claim I spot-checked against production source:

| Suspect claim | Production reality (read this session) | Class |
|---|---|---|
| §6.1 tagId swallowed on update | `validateForUpdate` throws on the errors list BEFORE the trailing TAGID check; the check is dead on the path (`GLAccountCommand.java`, tail of method) | VERIFIED |
| §6.2 null usage accepted on create | USAGE has no `notNull()`/`ignoreIfNull()`; `DataValidatorBuilder.inMinMaxRange` (lines 264–269) passes null through when `ignoreNullValue` is false; `account_usage` is `nullable=false` (`GLAccount.java:76`) | VERIFIED |
| §6.3 changes-map vs applied | `AccountingRule.handlePropertyUpdate(boolean)` puts into `actualChanges` unconditionally; property write guarded by `accountToCredit == null` | VERIFIED |
| §6.4 closure-enum office read | `AccountingRuleWritePlatformServiceJpaRepositoryImpl.java:31,93` reads office via `GLClosureJsonInputParams.OFFICE_ID`; both enums carry the literal `"officeId"` — "harmless today" accurate | VERIFIED |
| §6.5 guardless rule delete | `deleteAccountingRule` = findOne + delete, nothing else | VERIFIED |
| §6.6 XOR asymmetry | validator create requires one side per side-pair; service rejects "both" on create AND update; "neither" unchecked on update — matches pins | VERIFIED |
| NPE on fresh-entity update | `update()` calls the **boolean** overload with `this.systemDefined` → unboxing NPE; behavior real. Comment attributes it to "the JPA-woven getter" — mechanism imprecise (call-site unboxing of the field); P3 nit, behavior pinned correct | VERIFIED |
| blank JSON string ≡ absent | `JsonParserHelper.extractStringNamed` (115–129) assigns only `isNotBlank` values — blank collapses to null | VERIFIED |
| usage-only update never reaches guards | `validateForUpdate` `anyOfNotNull(name, glCode, parentId, type, description, disabled)` — usage NOT listed | VERIFIED |
| delete happy path already pinned at pin | `testDeleteGLAccountSuccessWithNoDependencies` exists at 355287c | VERIFIED |

## 2. Map honesty — every quantitative claim I recounted came out exact (VERIFIED)

- Slice: 25/2,185 + 24/2,076 = **49 / 4,261** — exact.
- Spot LOC (9 files): GLAccountsApiResource 286, GLAccountCommand 110, write impl 273,
  AccountingRule 204, rule deserializer 224, rule write impl 346, glaccount deserializer 71,
  read impl 274 — all exact. `@Path("/v1/glaccounts")` ✓.
- Split-package claim: `GLAccount`, `GLAccountRepository`, `GLAccountType`, `GLAccountUsage`,
  `GLAccountData` all resolve to **fineract-core** — exact.
- Outbound closure: all 96 distinct `org.apache.fineract.*` imports of the 49 files resolve
  inside fineract-core or fineract-accounting; **zero** other modules (§4.3 VERIFIED).
- Inbound: provider **36** ✓, wcl 7 ✓, loan 3 ✓ savings 3 ✓ tax 3 ✓ investor 2 ✓ branch 1 ✓
  charge 1 ✓; per-class counts (28/24/11/7/7/6/3/3/1/1/1) reproduce exactly under the
  map's metric "importers outside the class's own module" (e.g. GLAccountData 36 gross − 8
  core-internal = 28 ✓).

## 3. Red-proof fidelity — real, named, discriminating (VERIFIED)

`red-proof-full.log`: `createRejectsNameLongerThan200() FAILED` at line 68 under the
name-boundary 200→300 mutation, 15/1 class run, `BUILD FAILED`, `EXIT=1`; the other 14
class tests PASSED (correct discrimination). Build copy end-state re-measured: HEAD
= study tip, `status --porcelain` empty, `diff` empty — the mutant lived only in the
disposable copy. Fork production untouched (§1).

## 4. Counts & end-state (VERIFIED)

- green-run.log: **69 tests / 10 classes**, BUILD SUCCESSFUL EXIT=0; my own `@Test` counts:
  pin = 5 (git grep at 355287c: 1 + 1 + 3), new files = 15+5+6+11+9+10+8 = **64**; delta
  +64 ✓.
- Pin clone: `rev-parse HEAD` = `355287c8…` ✓, `status --porcelain` = 0 lines (re-measured
  without index-lock side effects failing — a first attempt errored on an unsupported
  flag and its empty stdout is NOT counted; the clean re-run is).
- Fork study porcelain 0 ✓; commit author `code (11150) <code@agent-town.local>` ✓.

## 5. Findings

**F-1 (P2, CONTRADICTED — map §6 vs suite): the suspect inventory mis-states its own pinning.**
§6's intro: "each pinned in the S1 suite (comment `CHARACTERIZED (suspect…)` at the pin
site)" — FALSE two ways: (a) §6.4 (closure-enum office read) has NO CHARACTERIZED pin
anywhere in the 7 files (behaviorally untestable — it is a documentation-only suspect and
should be labelled as such); (b) **4 of the 11** `CHARACTERIZED (suspect` pins in the
suite are ABSENT from §6: fresh-entity update NPE, blank-string JSON collapse,
usage-only-update-never-reaches-guards, header-without-children delete-skip. §6 lists one
set of 6, the commit message lists a different set of 7, the comments mark 11. §6 is
the S2 decision list — the three views must be reconciled to one union.

**F-2 (P2, coverage hole — mutation-blind surface): the glaccount tag-resolution path is
unpinned.** `retrieveTagId` (per-account-type `CodeValueRepositoryWrapper.findOneByCodeNameAndIdWithNotFoundDetection`, silent null for an unmatched type) runs on BOTH create (tagId present)
and update (TAGID in changes); zero tests stub or verify `codeValueRepositoryWrapper` in
either glaccount service test (grep: 0 when/verify). The rule-side analog
(CodeValueNotFoundException) IS pinned — asymmetric. A mutation re-routing the ASSETS/LIABILITIES/…
code names, or dropping a branch, keeps the whole suite green. This is also where suspect
§6.1's swallowed validation resurfaces: `tagId=-1` update passes validation, lands in
changes, and hits `retrieveTagId(-1,…)` at service level — the command-level pin pins
the swallow but not its service-level consequence (`tagId=0` silently no-ops instead:
`GLAccount.update` treats tagId vs 0L, so no change is detected).

**F-3 (P2, uncovered §6-adjacent behavior + dead landmine): the usage-change guard is
one-directional and the downgrade path is unpinned.** The journal-entry guard fires only
when the result state `isHeaderAccount()` (update tail: `containsKey(USAGE) && isHeaderAccount()`);
a HEADER→DETAIL change with existing children/transactions is unguarded and no test pins
what actually happens. Related dead landmine: `GLAccountInvalidUsageException` is
constructed **nowhere** in fineract-accounting or fineract-core (`new GLAccountInvalidUsageException`
→ 0 hits) while the map's §2 lists it among live slice files — a tool-assisted S2 reading
would assume a usage guard exists where none does. Pin it or inventory it.

**F-4 (P3): `final-parity-green.log` overstates.** `:fineract-accounting:test` is
`FROM-CACHE` in it (line 70) — the test task did not execute; the evidence §5 "FINAL CHECK
… 10 classes / 69 tests, 0 failures" reads as an execution. The claim survives (green-run.log
executed the same inputs; a cache hit is input-identity evidence), and the producer DID
honestly disclose FROM-CACHE for the baseline — the final-check line just needs the same
label.

**F-5 (P3): NPE comment mechanism** ("via the JPA-woven getter") — see §1; behavior pinned
correctly, explanation imprecise.

**F-6 (P3): §5 "Importer modules" list omits fineract-core** (16 core files import the
split package's core-resident classes). Defensible under §4's framing (they import core's
own classes, not the slice) but stated without the qualifier the per-class table uses.

**F-7 (P3, recorded limit, not a defect): one mutation was planted (brief-legal).** By
construction the dead trailing tagId check cannot go red; the red-proof proves boundary
logic is caught, not that dead code is watched.

## 6. DoD back-read (brief line by line)

| Brief clause | State |
|---|---|
| map committed to study repo (research/03) | ✓ bb62c44 |
| tests committed to fork study | ✓ 8444614, test-only diff |
| :fineract-accounting:test green incl. new; old vs new count reported | ✓ 5→69, 3→10 classes, delta +64; baseline via disclosed FROM-CACHE + my independent git-grep |
| red-proof captured, named mutant | ✓ §3 |
| fork study porcelain empty | ✓ re-measured |
| pin clone untouched (rev-parse + porcelain 0) | ✓ re-measured |
| suspect pins commented, not fixed | ✓ zero production hunks |
| unit idiom, ≤600 lines, one concern/file | ✓ max 231 lines |
| DB pins optional | not done — allowed (optional) |

## 7. Reading

The baseline is SOUND where it counts: nothing was fixed under the cover of pinning, the
red-proof is real and discriminating, every measured number I re-took is exact. What
fails the bar is the steering layer S2 will drive by: the suspect inventory contradicts
the suite it claims to summarize (F-1), and two reachable write/validation behaviors the
brief's own remit covers (tag resolution; usage-change matrix) are mutation-blind (F-2,
F-3). All three are cheap, single-concern cycle-2 fixes for the producer (map §6
reconciliation + 3–4 added pins), not a rebuild.

# JUDGED: 2b397c4b3a520f0c0208d6d4c20e0d4cee663ed66d071864b4e934040586d3e5
# VERDICT: FIX
