# TEST-verdict — S1 gate: independent verification of the accounting glaccount+rule characterization baseline (MC 11150)

Tester: test profile (independent of the producer). Date 2026-10-23. Host VM350.
Harness: docker `eclipse-temurin:25-jdk` per `.audits/20261022-2156-s0-toolchain/HOW-TO-RUN.md`,
`-e VERSIONING_DISABLE=true`, `GRADLE_USER_HOME=$PWD/.tmp/gradle-home` (reused cache), `--no-daemon --console=plain`.
Build copy (mine, fresh): `.tmp/fineract-s1-testgate-build` = `cp -a ../fineract-fork` (study @ 8444614, porcelain 0 at copy time; producer's build dir never reused). Full logs: `out/.tmp/*.log`.

## GATE 1 — GREEN reproduces (fresh copy) — PASS

```
rm -rf .tmp/fineract-s1-testgate-build
cp -a /home/claudecode/Kassan/fineract-fork .tmp/fineract-s1-testgate-build   # HEAD 8444614, porcelain 0
docker run ... eclipse-temurin:25-jdk ./gradlew --no-daemon --console=plain :fineract-accounting:test
```
gate1-green.log: `BUILD SUCCESSFUL in 33s`, `GATE1_EXIT=0` (test task FROM-CACHE — same input hash as the cached producer run).
To prove the tests actually EXECUTE in this container, forced rerun:
```
./gradlew --no-daemon --console=plain :fineract-accounting:test --rerun
```
gate1b-rerun.log: `> Task :fineract-accounting:test` (executed, not FROM-CACHE), `BUILD SUCCESSFUL in 37s`, `GATE1B_EXIT=0`.
Counts parsed from `fineract-accounting/build/test-results/test/TEST-*.xml`:
**TOTAL classes=10 tests=69 failures=0 errors=0 skipped=0** — matches the S1-evidence claim (10/69/0) exactly.

## GATE 2 — RED ×2 (mutations planted by ME, neither the producer's 200→300 GLAccountCommand edit) — PASS (2/2 caught)

### 2a — rule-domain behavior flip (§6 suspect #3 area: changes-map vs applied-property guard)
```
sed -i 's/if (this.accountToCredit == null) {/if (this.accountToCredit != null) {/' \
  .../accounting/rule/domain/AccountingRule.java        # handlePropertyUpdate(boolean) guard flip, 1 line
```
gate2a-red.log (scoped `--tests ...AccountingRuleUpdateCharacterizationTest`):
```
Test allowMultipleCreditEntriesIsReportedAsChangedEvenWhenDeliberatelyNotApplied() FAILED
org.opentest4j.AssertionFailedError: property must stay unchanged while an account-to-credit is set ==> expected: <false> but was: <true>
> Task :fineract-accounting:test FAILED
BUILD FAILED in 35s ; GATE2A_EXIT=1
```
Named test: `AccountingRuleUpdateCharacterizationTest > allowMultipleCreditEntriesIsReportedAsChangedEvenWhenDeliberatelyNotApplied` (the §6-suspect-3 pin). Restored via `git checkout`; build-copy porcelain 0.

### 2b — deserializer boundary flip (rule deserializer create-side name length)
```
sed -i '79s/notExceedingLengthOf(100)/notExceedingLengthOf(200)/' \
  .../rule/serialization/AccountingRuleCommandFromApiJsonDeserializer.java   # validateForCreate NAME bound
```
gate2b-red.log (scoped `--tests ...AccountingRuleCommandFromApiJsonDeserializerCharacterizationTest`):
```
Test createRequiresNameAndRejectsNamesOver100Chars() FAILED
org.opentest4j.AssertionFailedError: Expected org.apache.fineract.infrastructure.core.exception.PlatformApiDataValidationException to be thrown, but nothing was thrown.
> Task :fineract-accounting:test FAILED
BUILD FAILED in 36s ; GATE2B_EXIT=1
```
Named test: `AccountingRuleCommandFromApiJsonDeserializerCharacterizationTest > createRequiresNameAndRejectsNamesOver100Chars`. Distinct from the producer's red-proof (different file, class, method, numbers). Restored; build-copy porcelain 0.

## GATE 3 — suite-quality sample (4 of the 7 new files read in full) — ACCEPTABLE

Files read: `AccountingRuleUpdateCharacterizationTest`, `rule/serialization/AccountingRuleCommandFromApiJsonDeserializerCharacterizationTest`, `GLAccountWritePlatformServiceUpdateDeleteCharacterizationTest`, `GLAccountCommandValidationCharacterizationTest`.
- Assertions are MEANINGFUL state, not "no exception": exact globalisation message codes (`error.msg.glaccount.glcode.invalid.delete.has.children` etc.), per-parameter error-name checks inside `PlatformApiDataValidationException.getErrors()`, entity state after update (applied vs reported divergence), Mockito `verify(..., never())` interaction facts (e.g. parentId==id fails BEFORE any findById), and boundary pairs (name 200 accepted / 201 rejected).
- "Would a test stay green if the pinned method returned null?": NO in every sampled case — result tests dereference the return (`changes.get(...)`, `result.getResourceId()`) so null → NPE-fail, and assertThrows pins fail on null instead of the expected throw.
- FINDING P3 (inherent, not a defect): acceptance-direction pins (`assertDoesNotThrow`) would stay green if the validator were stripped of ALL checks in the accepted direction; the reject-direction of the same boundary is pinned in the same files, so the two directions together constrain the implementation. Characterization idiom, acceptable.
- FINDING P4 (§6 coverage note): suspect #4 (rule create reads officeId via `GLClosureJsonInputParams.OFFICE_ID`) has no test pin — correct call, nothing observable to pin (both enums carry the literal `"officeId"`, §6 itself says harmless); the map documents it. Not a suite gap.
- No §6-suspect pin found uncaught by my two independent mutations' classes of failure; my 2a directly demonstrates the §6-suspect-3 pin bites.

## End-state proofs (executed at gate end, after all mutation restores)
```
git -C ../fineract-fork branch --show-current          # study
git -C ../fineract-fork rev-parse --short HEAD         # 8444614
git -C ../fineract-fork status --porcelain | wc -l     # 0
git -C .tmp/clones/fineract rev-parse HEAD             # 355287c8feca747b3cd936a5b1cef1bf1bd7804ffea
git -C .tmp/clones/fineract status --porcelain | wc -l # 0
```
Cleanup: `rm -rf .tmp/fineract-s1-testgate-build` executed by me at end (rollback clean; mutants existed ONLY in that disposable copy).

## Verdict rationale
GATE 1 green reproduces independently (fresh copy + forced non-cache execution + XML counts 10/69/0). BOTH of my independently-planted mutations (rule-domain guard flip in the §6 suspect area; deserializer boundary flip) drove NAMED new-suite tests RED with meaningful assertion messages, and were restored. GATE 3 sample shows meaningful, state-level assertions with no null-transparent tests. PASS criteria met.

# JUDGED: 849d74368f01457bf1b6ebeb84ed2e2b3c7b9880538cfb0397da402068f7cbd4
# VERDICT: PASS
