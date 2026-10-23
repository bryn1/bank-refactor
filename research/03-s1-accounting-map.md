PREMISE: VM350 / qwen3.8-flash-next / Lane E as of 2026-10-23

# 03 — S1 map: fineract-accounting glaccount+rule slice (dependency / API surface)

MC 11150 ([type:code], child of 11146, phase S1). **rev 2 — DA/ARCH gate cycle-1 fix round
(2026-10-23).** Target pinned apache/fineract @
`355287c8feca747b3cd936a5b1cef1bd7804ffea` (pin clone `.tmp/clones/fineract`; working copy
`../fineract-fork` branch `study`). **Method law (from 02 §method): LOC recounted with
`find <dir> -name '*.java' -exec cat {} + | wc -l`**; dependency edges are measured
`import`/constructor-injection reads, not inferred. Tags: VERIFIED (measured this session,
path cited) / INFERRED (reasoned, not executed). No product code changed here; the S1 test
suite lives in `../fineract-fork` under `fineract-accounting/src/test/java` (see §7).
Length (over the ~150 soft mark) is deliberate: the edge tables (§4–§5) and the suspect
inventory (§6) are one measured unit — splitting them would separate evidence from reading.

## 1. Slice definition + measurement

Slice = package subtrees `org/apache/fineract/accounting/glaccount/` and `…/rule/` inside
module **fineract-accounting** (main sources only):

| subtree | files | LOC (cat-law) |
|---|---|---|
| `fineract-accounting/…/glaccount/` | 25 | 2,185 |
| `fineract-accounting/…/rule/` | 24 | 2,076 |
| **slice total** | **49** | **4,261** |

VERIFIED: reproduces the 02-plan figure (4,261 / 49) exactly.

The slice's Java package namespace `org.apache.fineract.accounting.glaccount.*` is **split
across two Gradle modules** — this is the map's most load-bearing fact (§5, §6).

## 2. Files grouped by sub-concern (49)

### glaccount (25 files, 2,185 LOC) — module fineract-accounting
| sub-concern | files (LOC) |
|---|---|
| api | `GLAccountsApiResource` (286), `GLAccountsApiResourceSwagger` (225) |
| command | `GLAccountCommand` (110) |
| data | `GLAccountDataForLookup` (38) |
| domain (jobs-support) | `TrialBalance` (82), `TrialBalanceRepository` (62), `TrialBalanceRepositoryWrapper` (48) |
| exception | `GLAccountDisable` (32), `GLAccountDuplicate` (32), `GLAccountInvalidClassification` (31), `GLAccountInvalidDelete` (61), `GLAccountInvalidParent` (32), `GLAccountInvalidUpdate` (51), `GLAccountInvalidUsage` (31 — **dead-but-present**: `new GLAccountInvalidUsageException` appears nowhere at this pin, grep 0 — the usage-change guard never throws it; see §6.6), `InvalidParentGLAccountHead` (30) |
| handler | `CreateGLAccountCommandHandler` (43), `DeleteGLAccountCommandHandler` (42), `UpdateGLAccountCommandHandler` (42) |
| jobs | `updatetrialbalancedetails/UpdateTrialBalanceDetailsConfig` (63), `…Tasklet` (151) |
| serialization | `GLAccountCommandFromApiJsonDeserializer` (71) |
| service | `GLAccountReadPlatformService` (43) + `…Impl` (274), `GLAccountWritePlatformService` (32) + `…JpaRepositoryImpl` (273) |

### rule (24 files, 2,076 LOC) — module fineract-accounting
| sub-concern | files (LOC) |
|---|---|
| api | `AccountingRuleApiResource` (255), `AccountingRuleApiResourceSwagger` (75), `AccountingRuleJsonInputParams` (67) |
| data | `AccountingRuleData` (65), `AccountingTagRuleData` (35), `request/AccountRuleRequest` (32) |
| domain | `AccountingRule` (204), `AccountingRuleRepository` (30), `AccountingRuleRepositoryWrapper` (44), `AccountingTagRule` (58) |
| exception | `AccountingRuleData` (30), `AccountingRuleDuplicate` (36), `AccountingRuleInvalidDelete` (33), `AccountingRuleInvalid` (57), `AccountingRuleNotFound` (36) |
| handler | `Create/Delete/UpdateAccountingRuleCommandHandler` (42 each) |
| serialization | `AccountingRuleCommandFromApiJsonDeserializer` (224) |
| service | `AccountingRuleReadPlatformService` (30) + `…Impl` (204), `AccountingRuleWritePlatformService` (32) + `…JpaRepositoryImpl` (346) |
| starter | `AccountingRuleConfiguration` (57) |

## 3. Entry classes (what an S2 refactor must keep callable)

External contract surface, all VERIFIED by reading the files:

- **REST**: `GLAccountsApiResource` `@Path("/v1/glaccounts")` (JAX-RS; POST/PUT/DELETE go through
  `PortfolioCommandSourceWritePlatformService` (fineract-core) with `CommandWrapperBuilder`
  resource names GL_ACCOUNT / ACCOUNTING_RULE); `AccountingRuleApiResource` `@Path("/v1/accountingrules")`.
  Reads call `GLAccountReadPlatformService` / `AccountingRuleReadPlatformService` directly.
- **Command handlers** (`@CommandType`-annotated, Spring component-scanned):
  `Create/Update/DeleteGLAccountCommandHandler` → `GLAccountWritePlatformService`;
  `Create/Update/DeleteAccountingRuleCommandHandler` → `AccountingRuleWritePlatformService`.
- **DI wiring**: glaccount write impl + deserializer are `@Service`/`@Component`
  (component-scan); rule services are wired explicitly in
  `rule/starter/AccountingRuleConfiguration` (`@Bean @ConditionalOnMissingBean`). Two idioms
  in one slice — S2 must pick one deliberately.
- **Batch job**: `UpdateTrialBalanceDetailsConfig`/`Tasklet` (Spring Batch, `acc_gl_*` SQL) —
  reaches the glaccount data via `TrialBalanceRepository*`, scheduled by infrastructure-jobs
  (`JobName.UPDATE_TRIAL_BALANCE_DETAILS`, core).

Constructor-dependency edges (measured `private final` sets):

| class | injected deps |
|---|---|
| `GLAccountWritePlatformServiceJpaRepositoryImpl` | `GLAccountRepository`(core), `JournalEntryRepository`, `ProductToGLAccountMappingRepository`, `GLAccountCommandFromApiJsonDeserializer`, `CodeValueRepositoryWrapper`(core), `JdbcTemplate` |
| `GLAccountReadPlatformServiceImpl` | `JdbcTemplate` |
| `AccountingRuleWritePlatformServiceJpaRepositoryImpl` | `AccountingRuleRepositoryWrapper`, `AccountingRuleRepository`, `GLAccountRepositoryWrapper`(core), `OfficeRepositoryWrapper`(core), `AccountingRuleCommandFromApiJsonDeserializer`, `CodeValueRepository`(core) |
| `AccountingRuleReadPlatformServiceImpl` | `JdbcTemplate`, `GLAccountReadPlatformService` |

## 4. Outbound edges (what the 49 files reach) — measured

All `import org.apache.fineract.*` statements of the 49 slice files resolve to exactly two
modules (VERIFIED by resolving each class to its `src/main/java` home):

1. **fineract-accounting, other sub-packages** (module-internal): `accounting.closure.api`
   (1 import: `GLClosureJsonInputParams`, used by the rule write service),
   `accounting.journalentry.domain/data`, `accounting.producttoaccountmapping.domain`,
   `accounting.common`, `accounting.trialbalance.exception`, and the slice's own halves.
2. **fineract-core**: everything else — `infrastructure.core.*` (JsonCommand, DataValidatorBuilder,
   CommandProcessingResult*, serialization, exceptions, JdbcSupport…), `infrastructure.codes.*`,
   `infrastructure.bulkimport/jobs/security` (API-resource + job plumbing), `organisation.office.*`
   (Office, OfficeRepositoryWrapper, OfficeReadPlatformService), `useradministration.domain.AppUser`,
   `commands.*` (handler interfaces, CommandWrapper, CommandWrapperBuilder).
3. **Zero imports of fineract-provider or any other module.** The plan's fact "module-internal
   today for the planned slice" is VERIFIED in the outbound direction.

**The core half of the slice's own package** (same `org.apache.fineract.accounting.glaccount.*`
namespace, but module **fineract-core**, so OUTSIDE the refactor slice as defined):
`domain.GLAccount`, `domain.GLAccountRepository`, `domain.GLAccountRepositoryWrapper`,
`domain.GLAccountType`, `domain.GLAccountUsage`, `api.GLAccountJsonInputParams`,
`data.GLAccountData`, `exception.GLAccountNotFoundException`,
`mapper.GlAccountMapper` (+`GlAccountTypeMapper`), and `accounting.common.AccountingConstants`
/`AccountingEnumerations`, `accounting.journalentry.domain.JournalEntryType` (core).
A "lift the slice into the bank core" move drags or splits these first — they are the seam.

Non-import runtime exits (measured in code, flagged INFERRED where only grep-observable):
- Raw SQL through injected `JdbcTemplate`: `GLAccountWritePlatformServiceJpaRepositoryImpl.validateForAttachedProduct`
  (`select count(*) from acc_product_mapping …` — reaches the *product* table directly, bypassing
  the mapping repository), and both read-service impls (`acc_gl_account`, `acc_accounting_rule` SQL).
- `@CommandType` strings + `CommandWrapperBuilder` resource enums registered core-side — the
  provider command framework discovers handlers by component scan (no compile edge).
- Spring Batch job config registers a step into the infrastructure-jobs scheduler (core).

## 5. Inbound edges (who consumes the slice from outside) — measured, S2's real coupling

Other modules importing `accounting.glaccount.*` / `accounting.rule.*` (import counts by class):

| imported class | own module | external importers |
|---|---|---|
| `glaccount.data.GLAccountData` | core | 28 |
| `glaccount.domain.GLAccount` | core | 24 |
| `glaccount.domain.GLAccountType` | core | 11 |
| `glaccount.domain.GLAccountRepositoryWrapper` | core | 7 |
| `glaccount.domain.GLAccountRepository` | core | 7 |
| `glaccount.service.GLAccountReadPlatformService` | **accounting** | 6 |
| `glaccount.exception.GLAccountNotFoundException` | core | 3 |
| `glaccount.domain.GLAccountUsage` | core | 3 |
| `rule.domain.AccountingRule` / `AccountingRuleRepository` | accounting | 1 each (provider provisioning) |
| `rule.exception.AccountingRuleNotFoundException` | accounting | 1 |
| `glaccount.data.GLAccountDataForLookup`, `glaccount.mapper.*` | accounting / core | 1–2 |

Importer modules (file counts): provider 36, loan 3, savings 3, tax 3, working-capital-loan 7,
investor 2, branch 1, charge 1 (bulk-import populators, provisioning, teller, journal-entry
processors). Counted under the discipline stated directly below (VERIFIED by producer recount
at fork `3150b9d` + independent DA c2 re-gate).

**Counting discipline for this section (both tables):** importers = `src/main` files in modules
other than **both** `fineract-accounting` and `fineract-core` (both excluded regardless of which
module owns the imported class); test sources excluded everywhere. Under this rule
`git grep -l "import org.apache.fineract.accounting.<pkg>.<Class>;"` (semicolon-anchored —
unanchored, `GLAccountData` also swallows `GLAccountDataForLookup`) reproduces the table:
28 / 24 / 11 per class and the read-service 6, module row 36/7/3/3/3/2/1/1 — recounted by the
producer at fork `3150b9d` (cycle-3 fix) and independently reproduced at the DA c2 re-gate. A
naive all-source grep (tests + the two excluded modules included) inflates to 51/50/30
(read-service 12) — the reason the exclusion is spelled out.

Reading for S2: the slice's *write* half is genuinely local (no external importer of the write
services); the coupling sits entirely in the **read model + domain data types**
(`GLAccountData`, `GLAccount`, `GLAccountType`, read service), most of which already live in
fineract-core. The write/read seam (§3 deps table) is where a bank-core lift cuts cleanly.

## 6. Suspect inventory — ONE union of every `CHARACTERIZED (suspect — candidate for S2 decision)` pin in the suite

Rev 2 reconciled this section to the suite itself (DA cycle-1 F-1): it lists every suspect pin
(fork `study`, `fineract-accounting/src/test/java/org/apache/fineract/accounting/`), no pin is
omitted and no entry lacks a pin. Suspects WITHOUT observable behavior are labeled
documentation-only below instead of pretending to be pinnable.

| # | suspect (current behavior, pinned AS-IS) | pin site |
|---|---|---|
| 6.1 | `GLAccountCommand.validateForUpdate` runs its `tagId > 0` check AFTER the errors-list throw — dead on the path; tagId violations pass validation | `glaccount.command.GLAccountCommandValidationCharacterizationTest::updateSilentlyIgnoresTagIdViolations` |
| 6.2 | `validateForCreate` accepts `usage=null` (no `notNull()`/`ignoreIfNull`; `inMinMaxRange` lets null pass) while `acc_gl_account.account_usage` is `nullable=false` | `…ValidationCharacterizationTest::createAcceptsNullUsageEvenThoughTheColumnIsNotNullable` |
| 6.3 | tagId chain of 6.1: update `{"tagId":-1}` passes the command layer, the service resolves it and the wrapper's not-found detection surfaces `CodeValueNotFoundException(-1)` — a code-value error where a validation error was owed | `glaccount.service.GLAccountWritePlatformServiceTagResolutionCharacterizationTest::updateWithNegativeTagIdPassesCommandValidationThenFailsAtTheServiceTagLookup` |
| 6.4 | tagId=0 update is indistinguishable from "no tag change" (`GLAccount.update` base-collapses absent tag to 0L): no lookup, no reported change — silently dropped | `…TagResolutionCharacterizationTest::updateWithTagIdZeroIsSilentlyDroppedAsNoChange` |
| 6.5 | usage-only update rejected by `validateForUpdate` (`anyOfNotNull` omits `usage`) — the service-level usage guards are unreachable via usage-only payloads | `glaccount.service.GLAccountWritePlatformServiceUpdateDeleteCharacterizationTest::usageOnlyUpdateNeverReachesTheServiceGuards` |
| 6.6 | the usage-change guard is one-directional: fires only when the RESULT is HEADER (promotion); HEADER→DETAIL demotion with children/transactions is unguarded, journal repo never queried | `…UpdateDeleteCharacterizationTest::demotingHeaderToDetailIsNotGuardedAtAllEvenWithChildrenAndTransactions` |
| 6.7 | delete guard skips header accounts whose children collection is empty (guard needs HEADER + non-empty children) | `…UpdateDeleteCharacterizationTest::deleteOfHeaderAccountWithoutChildrenSkipsTheChildrenGuardAndSucceeds` |
| 6.8 | `AccountingRule.update()` reports `allowMultipleCreditEntries` in the changes map while deliberately not applying it (`accountToCredit != null` guard) — result claims a change that did not happen | `rule.domain.AccountingRuleUpdateCharacterizationTest::allowMultipleCreditEntriesIsReportedAsChangedEvenWhenDeliberatelyNotApplied` |
| 6.9 | `AccountingRule.update()` on a never-persisted entity throws NPE (call-site unboxing of the null `Boolean systemDefined` into the primitive overload) | `…AccountingRuleUpdateCharacterizationTest::updateOnFreshlyConstructedEntityWithoutSystemDefinedThrowsNpe` |
| 6.10 | rule deserializer: an explicit blank JSON string for name extracts as null ≡ absent (`JsonParserHelper.extractStringNamed`) — a blanking update is a silent no-op | `rule.serialization.AccountingRuleCommandFromApiJsonDeserializerCharacterizationTest::updateSilentlyTreatsAnExplicitlyBlankNameAsNotSent` |
| 6.11 | account-XOR-tags enforced asymmetrically: create-validator lets account AND tags for one side pass; the service, not the validator, rejects "both"; "neither" unchecked on update | `…DeserializerCharacterizationTest::createAcceptsAccountAndTagsForTheSameSide`, `::updateDoesNotEnforceTheAccountOrTagsPresenceThatCreateEnforces`, `rule.service.AccountingRuleWritePlatformServiceCharacterizationTest::createWithBothDebitAccountAndDebitTagsIsRejectedByTheServiceNotTheValidator` |
| 6.12 | `deleteAccountingRule` deletes with no journal-entry/usage guard at all (contrast `deleteGLAccount`'s three) | `…AccountingRuleWritePlatformServiceCharacterizationTest::deleteRemovesTheRuleWithNoUsageOrTransactionGuardsAtAll` |

Documentation-only findings (no observable behavior to pin — kept because S2 must not be
surprised; nothing in the suite can or should assert them):

- **D-1** `createAccountingRule` reads the office param via `GLClosureJsonInputParams.OFFICE_ID`
  (closure enum) instead of the rule enum — both carry the literal `"officeId"` (VERIFIED both
  files), so behavior is identical today; a wrong-package dependency that will mislead a
  tool-assisted refactor. Unpinnable: an enum choice with no observable difference has no test.
- **D-2** `GLAccountInvalidUsageException` (module file, §2) is constructed NOWHERE at this pin
  (grep `new GLAccountInvalidUsageException` = 0) — dead-but-present; the usage guard behavior it
  would express is pinned at 6.6.

## 7. Test surface (S1 baseline)

Existing at pin: 3 test classes / 5 `@Test` methods (S0 record):
`glaccount/service/GLAccountWritePlatformServiceJpaRepositoryImplTest` (1),
`rule/data/request/AccountRuleRequestSerializationTest` (3),
`provisioning/…DeserializerTest` (1, outside slice).
S1 adds `fineract-accounting/src/test/java/…` characterization files (glaccount command
validation, glaccount deserializer, write-service create/update/delete branches, write-service
tag resolution (§6.3–6.4), rule write service, rule domain update, rule deserializer boundaries)
— run counts OLD vs NEW are in the S1 evidence dir
`.audits/20261023-0045-s1-accounting-baseline/S1-evidence.txt`.
DB-free unit idiom throughout (Mockito/JUnit5), matching the module's existing style;
`ThreadLocalContextUtil` only where `DateUtils`-dependent paths run (none in the pinned set).
