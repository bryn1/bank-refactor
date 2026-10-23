# S1 phase-input brief — for the dod-loop phase children (test / devils-advocate / arch)

Producer (code, MC 11150) handoff for gate run `/home/claudecode/Kassan/.audits/202610222333-37c5619e/out/`.
The phase children write `TEST-verdict.md`, `DA-verdict.md`, `ARCH-verdict.md` (+ `# VERDICT:` line) there;
DONE.md is the orchestrator's. Precedent shape: `/home/claudecode/Kassan/.audits/202610222146-3f778626/out/`.
A FIX verdict routes back to the producer (code 11150) for the fix cycle.

## Artifacts under review
- Tests: fineract-fork `study` @ `8444614` (7 files, `fineract-accounting/src/test/java/…CharacterizationTest.java`), porcelain empty.
- Map: bank-refactor `research/03-s1-accounting-map.md`; evidence: bank-refactor `.audits/20261023-0045-s1-accounting-baseline/` (S1-evidence.txt VERIFY_EXIT=0; bank-refactor commit `bb62c44`).

## TEST child — independent reproduction (DB-free, docker JDK-25 per .audits/20261022-2156-s0-toolchain/HOW-TO-RUN.md)
```bash
cd /home/claudecode/Kassan/bank-refactor
rm -rf .tmp/fineract-s1-gate && cp -a ../fineract-fork .tmp/fineract-s1-gate   # fresh copy, do NOT touch fineract-s1-build or the pin clone
docker run --rm --user 1000:1000 \
  -e GRADLE_USER_HOME=/work/gradle-home -e HOME=/work/gradle-home \
  -e VERSIONING_DISABLE=true \
  -v "$PWD/.tmp/fineract-s1-gate:/work/fineract" \
  -v "$PWD/.tmp/gradle-home:/work/gradle-home" \
  -w /work/fineract eclipse-temurin:25-jdk \
  ./gradlew --no-daemon --console=plain :fineract-accounting:test
# expect BUILD SUCCESSFUL; XMLs under .tmp/fineract-s1-gate/fineract-accounting/build/test-results/test/:
#   10 classes / 69 tests / 0 failures (baseline at pin: 3 classes / 5 tests — stash -u to reproduce OLD)
```
RED-reproduction (one mutation, build-copy only): `sed -i 's/notExceedingLengthOf(200)/notExceedingLengthOf(300)/'`
in `fineract-accounting/src/main/java/org/apache/fineract/accounting/glaccount/command/GLAccountCommand.java`
matches TWO lines (create AND update) — the producer's red-proof mutated only the create line
(`.value(this.name).notBlank().notExceedingLengthOf(200);`); mutating both additionally reddens
`updateAllowsPartialFields`. Then rerun with
`--tests org.apache.fineract.accounting.glaccount.command.GLAccountCommandValidationCharacterizationTest`
→ expect `createRejectsNameLongerThan200() FAILED` (producer capture: red-proof-full.log). Restore = re-copy.
End-state to re-verify: `git -C ../fineract-fork status --porcelain` empty;
pin clone rev-parse `355287c8feca747b3cd936a5b1cef1bd7804ffea` + porcelain 0.

## DA child — adversarial targets (all in map §6 + the 7 test files)
Pin-vs-reality honesty: are any pins asserting should-be instead of is? Suspect labels right?
Known-weak spots to attack: tagId-swallow pin (does it prove silence or just absence of throw?),
null-usage create acceptance (DB column nullable=false — is the pin honest about the layer boundary?),
changes-map NPE pin (JPA weaving present at test runtime? it reproduced with _persistence_get_systemDefined),
rule delete guardless pin, blank-string collapse pin (T7) vs GLAccountCommand direct-construction pin (T1) —
different layers, is that documented where S2 will read it (map §6 vs test comments)?

## ARCH child — placement targets
Map correctness: §4 outbound (re-derive: every `import org.apache.fineract.` of the 49 files resolves to
fineract-accounting or fineract-core — re-run the import resolution); §5 inbound counts (re-grep);
§3 DI tables vs constructor reality. Plus: is the map the right placement (research/03, numbered convention)
and does it duplicate anything research/02 already owns?
