PREMISE: VM350 / qwen3.8-flash-next / Lane E as of 2026-10-23

# 05 — charge-slice decision ledger: S4 hand-off rows + S4a suspect map + S4-B peel mechanism (what S4 remaining passes read first)

PROVENANCE: split from `04-s2-suspect-decisions.md` at its 138-L mark per the S4 gate cycle-1 ARCH
ruling (research/04 "split named for the next append"; this file IS that append — fork `470896d`
+ this commit). `04` keeps the S2 accounting-slice + S3 peel rows as history; every S4 row lives
HERE. Files 01-03 unchanged. S4 remaining passes (S4c/S4d) read this file first; S2/S3 history
reads `04`.

### S4 hand-off rows (from gate cycle-1, DA FIX F-A + riding P3/P4 — recorded here, NO code in S3)

| # | decision | rationale (one line) | proof |
|---|---|---|---|
| S4-T | **DEFERRED DECISION (placeholder, S4 must decide)** | The S3 peel leaves a two-mechanism state (module-local `AccountingNativeSqlExecutor` beside the live `JdbcTemplate` at the 10 non-slice sites — DA F-A P2: replace, not coexist); its TERMINUS is one of: **converge** — peel the remaining 10 sites (provisioning x6, closure x2, financialactivity x2) onto the executor/repos so the seam is the module's only SQL path — or **relax the probe** — let repository classes hold wrapped `JdbcTemplate` instances and RETIRE `AccountingNativeSqlExecutor`. S3 could not converge (its DoD probe pins the remainder at exactly 10); the red-capability the F-A ask demanded landed with `a932d91` (6 unit pins, planted-mutation capture). | fork `a932d91` + `f5-executor-red-capture.log` |
| S4-D | **P3/P4 debts ride (rows per DA F-B..F-E + brief handback)** | (1) 6.6 PIN-SPLIT (F-C P3): add children-only and transactions-only demotion-reject pins — the present pin stubs both disjuncts true together, so either single-disjunct removal stays green (R-4 observation as a row); (2) DEBIT same-shape flag (F-B P3): the debit tag branch carries the pre-fix credit failure mode — widening F-2 to it is an owner-scope call, R-3 is its explicit half-invariant record; (3) FALSE-VALUED FLAGS (P4, brief-named): `if (allowMultipleXEntries)` drops an explicit false on compound PUTs in BOTH tag branches — neither applied nor reported; (4) F-D P4: IncorrectResultSize size-report deviation (unreachable at slice call sites); (5) F-E P4: readOnly-proxy transaction-metadata nuance of the delegation swap (data identical). | DA verdict `202610230859-5a975a66/out/DA-verdict.md` (JUDGED b86a6bb512dc494b3dcd3cac9351ab7d1907676fc5ec6b2731f978c3dc7f61f2 — live pin as of the 2026-10-23 rename re-pin, read from out/DA-verdict.md this session) |


## S4 additions — S4a CHARGE SLICE CHARACTERIZATION BASELINE (MC 11183, 2026-10-23)

Scope pinned AS-IS (NO fixes this pass): `fineract-charge` module public service/DTO surface +
validation + query building, and provider-resident `portfolio/charge/` (ChargeReadPlatformServiceImpl
502 l / 14 jdbcTemplate + 3 namedTemplate sites, ChargeWritePlatformServiceJpaRepositoryImpl 291 l,
ConvertChargeDataToSpecificChargeData, ChargeConfiguration). DB-free seams: mocked JdbcTemplate/
NamedParameterJdbcTemplate capture the SQL strings; the captured RowMapper is driven against a
dynamic-proxy ResultSet (label/index reads in lockstep incl. wasNull — Spring 7 JdbcUtils dispatches
typed index getters). Fork `study` commits: `5cf5598` (deserializer S-C1..S-C6, 35 tests), `a39cf6e`
(enums/dropdown/API surface S-C7..S-C9, 25 tests), `6c92ff2` (Charge entity S-C10..S-C15, 12 tests),
`4df027c` (read-service, 14 tests), `292fcae` (write-service, 12 tests), `d4b1250` (util+starter,
5 tests), `068c8f4` (label renumber only). Boundary counts (:fineract-charge:test / :fineract-
accounting:test guard): baseline 11/2 + 84/12 -> 46/3 -> 71/7 -> 83/8 at `6c92ff2`, 84/12 at every
boundary; provider targeted `org.apache.fineract.portfolio.charge.*` 31 tests / 4 classes green at
`068c8f4`. Planted-red (build copy): `if (!chargeForUpdate.isActive())` -> `if (true)` at write-impl
147 -> exactly `reactivationOfAUsedChargeRunsNoUsageGuardAtAll` RED (ChargeCannotBeUpdatedException),
all other 11 pins of the class GREEN; restored byte-verified. Evidence:
`.audits/20261023-1333-s4a-charge-baseline/S4a-evidence.txt`.

### S4 suspect map (S4a rows; all PINNED AS-IS — decision belongs to the S4 cleanup pass)

| # | file:line | current behavior (pinned) | risk if left | pin — fork commit |
|---|---|---|---|---|
| S-C1 | fineract-charge .../serialization/ChargeDefinitionCommandFromApiJsonDeserializer.java:121-126,145 | explicit JSON `null` for penalty/active/enableFreeWithdrawalCharge/enablePaymentType passes parameterExists, extractBooleanNamed returns null, unboxing NPEs before any validator error | API caller sees a 500/NPE instead of a validation error for a legal-looking payload | create/update validation pins — `5cf5598` |
| S-C2 | same file:116,119,130,134,150 + DataValidatorBuilder (core, frozen) | integerGreaterThanZero / inMinMaxRange SILENTLY SKIP null: feeInterval/feeFrequency/free-withdrawal freq/restart freq/paymentTypeId explicit-null passes validation | half-configured charges accepted (flag true without its required partner — see S-W1) | null-skip pins — `5cf5598` |
| S-C3 | same file (create/update branch switch on chargeAppliesTo) | unknown chargeAppliesTo (99) reports one error but STILL skips every branch validator (loan/savings guards never run) | payload with bogus appliesTo exercises no field rules | unknown-appliesTo pin — `5cf5598` |
| S-C4 | same file (validateForUpdate) | `{}` (empty object) passes validateForUpdate with zero errors | updates with no fields accepted silently at the API layer (entity later no-ops) | empty-update pin — `5cf5598` |
| S-C5 | same file:357/395/401 + 347/454 | identical chargeAppliesTo branch blocks duplicated (and minCap/maxCap validated in TWO identical blocks): each bad value yields TWO duplicate errors | error payloads carry duplicate keys; fix drift between copies | dup-block error-count pins — `5cf5598` |
| S-C6 | same file (feeOnMonthDay extraction) + JsonParserHelper.extractMonthDayNamed | monthDay payload without `locale` NPEs inside DateTimeFormatterBuilder.toFormatter (core helper) | 500 instead of validation error for locale-less payloads | monthDay-locale pin — `5cf5598` |
| S-C7 | fineract-charge .../domain/ChargeCalculationType.java (fromInt) | CalculationType.fromInt(null) NPEs while sibling enums' fromInt(null) return INVALID | null from a DB converter path explodes instead of landing on INVALID | fromInt-tables pins — `a39cf6e` |
| S-C8 | ChargePaymentMode.java + ChargeEnumerations.java | PaymentMode has NO INVALID member: every unknown id (and the DB converter's default) displays as "Regular" | corrupted/out-of-range payment modes masquerade as REGULAR (real money semantics) | display + fromInt pins — `a39cf6e` |
| S-C9 | fineract-charge .../service/ChargeDropdownReadPlatformServiceImpl.java | savings collection-time dropdown omits SAVINGS_CLOSURE(4) while the validator's validSavingsValues() accepts it | UI cannot select a time-type the API accepts | ordered-dropdown pins — `a39cf6e` |
| S-C10 | fineract-charge .../domain/Charge.java (update(), countFrequencyType block) | update() re-decodes countFrequencyType via **ChargeTimeType**.fromInt: input 99 stores 0 on update but create stored 5 (PeriodFrequencyType) | update silently re-types frequency values through the wrong enum table | update-path pin — `6c92ff2` |
| S-C11 | Charge.java:676 (toData) | toData builds CurrencyData(code, null, 0, 0, null, null) — name/decimal-places lost (P4, rides S4-D-style debt) | DTO consumers see decimalPlaces 0 for 2-decimal currencies | toData-shape pin — `6c92ff2` |
| S-C12 | Charge.java:552 (update feeOnMonthDay branch; ctor 234-235 mirror) | adding feeOnMonthDay when this.feeOnDay is null NPEs (`!this.feeOnDay.equals(...)`) | 500 on a legal sequence for charges created without day/month | feeOnMonthDay-NPE pin — `6c92ff2` |
| S-C13 | Charge.java:476-480 (update paymentTypeId) | `{"paymentTypeId": null}` is REPORTED in the changes map (9->null) but never applied — entity keeps the type | applied != reported invariant broken at the entity | reported-never-applied pin — `6c92ff2` |
| S-C14 | Charge.java (toData chargePaymentMode unbox) | toData() unboxes chargePaymentMode — savings/client/share charges with null mode NPE the DTO build | toData crashes for charge classes that never set the field | toData-NPE pin — `6c92ff2` |
| S-C15 | Charge.java (update(): trailing validators) | update() MUTATES entity fields before the trailing notNull/period validators throw | a rejected update leaves the managed entity half-updated (JPA may flush it) | mutation-before-throw pin — `6c92ff2`; made DISCRIMINATING at `470896d` (S4 gate F-A/DA: follow-up re-sends feeFrequency=2 with feeInterval=3 so the LACK-of-feeFrequency assertion holds only under mutate-before-validate; planted validate-first reorder flips it RED — captured, others green, restored) |
| S-R1 | fineract-provider .../charge/service/ChargeReadPlatformServiceImpl.java:76-88 + addInClause helper | office-scoping fragment from entity-access is string-CONCATENATED into the WHERE clause (`" and c.id in ( " + inClause + " ) "`) — trusted SQL re-embedded unparameterized | SQL-injection blast radius if the upstream fragment ever becomes user-influenced | in-clause wrapping pin — `4df027c` |
| S-R2 | same file (processChargeExclusions + named-template SQLs) | excluded charge-times are INLINED as raw integers (`not in(2,5)`) beside `:named` params in the same statement | binding-discipline split; enum-sourced today, footgun on refactor | named-param capture pin — `4df027c` |
| S-R3 | ChargeReadPlatformServiceImpl.java:331-334 (ChargeMapper.mapRow) | JdbcSupport.getInteger may return null; CurrencyData construction UNBOXES decimalPlaces -> NULL column NPEs the whole row mapping | one NULL column kills the entire list query | fake-ResultSet NPE pins — `4df027c` |
| S-R4 | ChargeReadPlatformServiceImpl.java:382-384 | free-withdrawal frequency/restart fields read with primitive getInt: a NULL column arrives on the DTO as **0** (a legal-looking value), never null | NULL vs 0 indistinguishable downstream (0 = "every withdrawal free" semantics) | NULL->0 collapse pin — `4df027c` |
| S-W1 | Charge.java:269-270 + write-service create | enablePaymentType=true WITHOUT paymentTypeId: validator passes (S-C2), service skips lookup, Charge.fromJson stores flag FALSE | user thinks payment type is enabled; stored row says disabled — silent config loss | create-silent-drop pin — `292fcae` |
| S-W2 | ChargeWritePlatformServiceJpaRepositoryImpl.java:246-251 (handleDataIntegrityIssues) | ANY data-integrity violation whose root-cause message merely CONTAINS "name" is remapped to error.msg.charge.duplicate.name | unrelated constraint violations (NOT NULL on office_name etc.) reported as duplicate-name | substring-misclassification pin — `292fcae` |
| S-W3 | ChargeWritePlatformServiceJpaRepositoryImpl.java:143-161 | active-branch guard tests the state AFTER update() (`if (changes.containsKey("active")) { if (!chargeForUpdate.isActive()) ... }`) and the fee guard sits in the ELSE-IF: reactivating a USED charge + changing feeInterval applies EVERYTHING with ZERO usage-guard queries | usage guards bypassable by combining fields; deactivation-only protection | reactivation-zero-guards pin — `292fcae` |
| S-W4 | ChargeWritePlatformServiceJpaRepositoryImpl.java:174-185 | service applies only NON-NULL paymentTypeId: changes-report says paymentTypeId changed to null while the association stays attached (service-side of S-C13) | applied != reported at the API response | reported-never-detached pin — `292fcae` |
| S-U1 | fineract-provider .../charge/util/ConvertChargeDataToSpecificChargeData.java:33-60 | toSavingsAccountChargeData hardcodes isFreeWithdrawal/frequency/restart to NULL and id/accountId to null even when the source ChargeData carries them | savings views built from definitions lose free-withdrawal config | field-carrying pins — `d4b1250` |

### Named DB-gaps (what the pins CANNOT prove without a live database — INFERRED level per brief §ACCEPTANCE)

| # | gap | would-prove |
|---|---|---|
| DB-1 | actual SQL execution vs PostgreSQL (all 14 jdbcTemplate + 3 namedTemplate sites: joins, column names, alias correctness) | the captured SQL strings are RUNNABLE and return the expected shape |
| DB-2 | @Cacheable/@CacheEvict "charges" cache behavior (tenant-key proxy on retrieveAllCharges; evict on update/delete) | caching layer actually serves/evicts per tenant key |
| DB-3 | EXISTS-guard truthiness vs real schema (m_product_loan_charge, m_savings_product_charge, m_loan_charge + is_active=1, m_savings_account_charge + is_active=1, WCL repo) | usage guards fire against real linking tables |
| DB-4 | real unique-constraint end-to-end feeding handleDataIntegrityIssues | duplicate-name insert actually reaches the mapped error (vs the mocked message path) |
| DB-5 | repo ITs / API integration (verify-api-*/verify-liquibase-* tripwires from S0) | full-stack charge CRUD on a migrated DB |

### S4-B row — PEEL MECHANISM (charge read-side SQL seam, MC 11183, 2026-10-23)

| # | decision | rationale (one line) | pins — commits |
|---|---|---|---|
| S4-B | **MECHANISM (route: RELAX-in-provider)** | The charge read side's 14 JdbcTemplate + 3 NamedParameterJdbcTemplate sites move BEHIND a new provider-internal `portfolio/charge/domain/ChargeSqlReadRepository` that WRAPS the same Spring JdbcTemplate/NamedParameterJdbcTemplate/FineractEntityAccessUtil instances the service already received — NO new executor and NO charge->accounting edge: reusing the S3 `AccountingNativeSqlExecutor` was ruled out because the pin-honesty law (parent ruling: move ZERO pins, change ZERO assertions) makes the peeled code pass UNCHANGED only if the same collaborator instances the S4a pins mock stay behind the seam, and an executor would replace those calls; charge also must not couple to an accounting-module mechanism while S4-T (converge-vs-relax for accounting) is still undecided. This is the two-mechanism state named by S4-T: after this pass the CHARGE slice reads via wrapped JdbcTemplate and the ACCOUNTING slice via AccountingNativeSqlExecutor — recorded honestly; S4-T's accounting-side terminus remains open (this pass neither converges nor retires it, and charge does not pre-adopt either answer). Seam-naming rider (ARCH cycle-1): the "Sql" insertion in `ChargeSqlReadRepository` is a DELIBERATE naming drift from the fork's S3 `<X>ReadRepository` idiom — named, not punished (no collision: Spring Data `ChargeRepository` untouched). Constructor signature + public interface + DTOs FROZEN; repository is a plain provider class (no bean, no interface), built inside the service constructor; SQL + schemas + ChargeMapper + office in-clause + loan-exclusion clause carried BYTE-IDENTICAL (literal-set probe: every pre-peel string literal of the service is present in service-union-repo, zero drift; added literals only from the new javadoc). | — `7fd7b92` (core m_charge + ChargeMapper + in-clause), `d666cee` (loan cluster + exclusion clause), `83dc366` (savings/share/WCL/client; service jdbc probe 0, repo probe 17); S-R1..S-R4 pins pass UNCHANGED at every boundary (31/4 + 84/12 + 83/8; test-tree diff over 068c8f4..tip is EMPTY) |
