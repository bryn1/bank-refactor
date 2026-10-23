# M5 — The suspect catalog as a negative-testing reading list

**The asset:** S4a mapped the charge slice's wrong-behavior candidates as 24
suspects — S-C1..S-C15 (module), S-R1..S-R4 (read service), S-W1..S-W4 (write
service), S-U1 (util) — every one pinned AS-IS, zero fixes, full rows in
`research/05-s4-charge-decisions.md` ("S4 suspect map"). That catalog is a
negative-testing course: 24 real cases of "what could go wrong" that nobody
inventoried, each with a pin that makes the behavior observable. The five below
are the study's chosen reading set — the LEDGER's S4-close carry names them as
S5 teaching material "pinned, not fixed, per funded scope": **S-W3, S-C15,
S-R3, S-W1, S-R4**.

Read each row in `research/05` first; this module adds only the lesson each case
carries and why leaving it was the disciplined move, not an oversight.

## The five, read as lessons

### S-W3 — guards ordered on post-mutation state (`292fcae`)
`ChargeWritePlatformServiceJpaRepositoryImpl.java:143-161`: the active-branch
usage guard tests the entity state AFTER `update()` already ran, and the fee
guard sits in the ELSE-IF — so REACTIVATING a used charge while also changing
`feeInterval` applies everything with ZERO usage-guard queries. The pin
`reactivationOfAUsedChargeRunsNoUsageGuardAtAll` holds that end state (it was
also the suite's planted-red witness, M1 §4 / M3 §4).
*Lesson:* a guard chain keyed on mutated state + `else if` is a bypass surface —
the negative test is the FIELD COMBINATION, which per-layer unit review never
thinks of. *Why left:* fixing it changes money-affecting accept/reject behavior;
the funded scope was characterize+peel, and a fix without the compat reasoning
(S2's KEEP-vs-FIX rationale discipline, `research/04`) is exactly the scope creep
`research/02` R3 fences off.

### S-C15 — mutation before validation, pinned by its own symptom (`6c92ff2`)
`Charge.update()` writes entity fields, then the trailing validators throw; the
rejected update leaves the managed entity half-updated (JPA may flush). The
pinning story is M2: the first pin was FAKE, the fix made it discriminating
(`470896d`) — today the pin is satisfiable only by the defect state, so the
defect stands pinned-but-alive at fork tip 618896c.
*Lesson:* mutate-then-validate is invisible to response-level tests (the API
still returns its error) and visible only to a test that reads the entity AFTER
the throw — plus M2's rule that the assertion must name the state it demands.
*Why left:* the fix reorders validation vs mutation across `update()`; that
changes which side effects survive rejection for EVERY field — an owner/cleanup-
pass decision, not a peel.

### S-R3 — one NULL column kills the whole list query (`4df027c`)
`ChargeReadPlatformServiceImpl.java:331-334`: `JdbcSupport.getInteger` may
return null and the `CurrencyData` construction UNBOXES `decimalPlaces` — a NULL
column NPEs the mapper for every row: one bad column kills the entire list.
Pinned via the fake-ResultSet NPE pins (the dynamic-proxy idiom, M1 §2).
*Lesson:* negative tests at the row-mapping layer are cheap and lethal — feed
NULL where the DTO unboxes; blast radius is the query, not the row.
*Why left:* the fix changes DTO nullability semantics (a public shape);
`research/02` R4 freezes DTO shapes for this study.

### S-W1 — the silent flag drop, everyone's and no one's (`292fcae`)
`Charge.java:269-270` + write-service create: `enablePaymentType=true` WITHOUT
`paymentTypeId` — the validator skips the null partner (family: S-C2's silent
null-skip), the service skips the lookup, `Charge.fromJson` stores the flag
FALSE. The user thinks payment type is enabled; the row says disabled. Pinned by
the create-silent-drop pin.
*Lesson:* cross-layer responsibility gaps need END-STATE negative tests — each
layer "handled" the payload, and the stored state is wrong; a per-layer test
suite passes over it.
*Why left:* rejecting it at the validator would newly refuse payloads accepted
since the pin date (public contract); making it succeed would invent behavior.
Classic KEEP-candidate reasoning per `research/04`'s rationale style — and that
ledger exists only for accounting; the charge cleanup pass (with the same FIX/
KEEP row discipline) is unfunded.

### S-R4 — NULL collapsed into a legal-looking 0 (`4df027c`, label `068c8f4`)
`ChargeReadPlatformServiceImpl.java:382-384`: free-withdrawal frequency/restart
read with primitive `getInt` — a NULL column arrives on the DTO as **0**, and 0
is a meaningful value (0 = "every withdrawal free"); downstream cannot
distinguish unset from zero. Pinned by the NULL→0 collapse pins; the label was
renumbered S-R2→S-R4 in `068c8f4` to avoid a collision with the other S-R2 —
the map's own audit trail.
*Lesson:* primitive reads on nullable columns are untypeable at the API — the
negative test is a NULL column, and the failure is silent (wrong value, not
error), the worst class.
*Why left:* the fix changes DTO values clients observe for NULL rows — contract
again, owner territory.

## Why "pinned, not fixed" was the disciplined outcome

The funded S4 scope ran characterization (S4a) and peels (S4b/S4c) plus the
mapped debts (S4-D) — no charge cleanup round. The S2 precedent shows what a fix
round costs even when funded: 12 suspects each needing FIX/KEEP with a one-line
banking-safety/compat rationale, 7 commits, pins renamed WITH their decision rows
(`research/04`; LEDGER S2 row). Fixing charge suspects without that per-row
reasoning would repeat the mistake `research/02` §4 R3/R4 fence against (scope
creep + contract drift). Pinning, by contrast, is the SAFE holding state: any
future change to these behaviors now goes RED loudly, so leaving them costs
nothing daily and buys the decision time.

The owner decisions riding on this catalog (from `research/05` + LEDGER carry
rows):

1. **Charge cleanup pass** — decide FIX/KEEP per suspect, as S2 did for
   accounting. The five above are its headline rows.
2. **False-valued flags (S4-D(3), record-only)** — on compound PUTs an explicit
   `false` for the multiple-credit/debit flags is NEITHER applied NOR reported,
   in both tag branches; the F-2-mirror fix (`b416474`) deliberately kept false
   symmetric with its prior state. A fix = "report explicit false" = a public
   reporting-semantics change = owner ruling. Recorded as-is, per dispatch.
3. **The `PaginationHelper` residual** (M4 §3) — retiring the surviving
   JdbcTemplate instance needs a `fineract-core` mechanism change; carried.

## The INFERRED boundary — what the catalog's pins cannot prove

Every pin is DB-free (M1 §2), so the catalog pins SQL SHAPE, not execution. The
study named five gaps rather than let silence imply them (`research/05`
"Named DB-gaps", recorded at INFERRED level): **DB-1** the captured SQL is
RUNNABLE against PostgreSQL (14+3 sites: joins, aliases, column names);
**DB-2** `@Cacheable`/`@CacheEvict` "charges" caching actually serves/evicts per
tenant; **DB-3** the EXISTS usage guards fire against the real linking tables
(m_product_loan_charge, m_savings_product_charge, m_loan_charge,
m_savings_account_charge, WCL); **DB-4** a real unique-constraint violation
actually reaches the mapped duplicate-name error; **DB-5** full-stack charge CRUD
on a migrated DB. The repo's own `verify-api-*` / `verify-liquibase-*` workflows
are the named tripwires (`research/02` §1/§4 R4).
Teaching label: S-W3/S-W1's *bypass logic* is TESTED (guards observably skipped
against mocks); their behavior *against real linking tables* is DB-3 — INFERRED.
State the level when you cite these cases; a negative-testing course that hides
its boundary teaches false confidence.

## How to run this as a course

For each of the 24 rows: (1) read the map row — behavior + file:line + risk;
(2) open the pin at the named fork commit and find the one assertion that makes
the case observable (ask M2's question: what state makes it FALSE?); (3) write
the negative test yourself from the risk column BEFORE reading the pin, and
compare what you missed; (4) classify per `poc-vs-production` thinking — which
rows are bugs vs design limitations vs frozen-contract artifacts; (5) note which
rows would flip RED under your proposed fix — that list is your change's true
blast radius. The pattern families to hunt across the full catalog: null
handling (S-C1, S-C7, S-R3, S-R4), silently-skipped validation (S-C2..S-C4),
guard ordering (S-W3, S-W1), applied≠reported (S-C13, S-W4 — and the CLOSED R-3
credit/debit pair in `research/04`/`research/05` S4-D as the fixed contrast),
duplication drift (S-C5), enum-table misuse (S-C8, S-C10).
