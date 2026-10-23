# M4 — Two mechanisms & the terminus (and why "probe zero" must name its measurement)

**The law:** a refactor may leave two mechanisms side by side only if the state
is NAMED, the choice to unify is DATED, and the unification NAMES what it
replaces. This study did exactly that — the two-mechanism state was recorded as
a debt row, the terminus was decided at the next stage, and the end-state number
("probe 10→0") learned the hard way that a green count is a claim about ONE
quantity, not about the property it stands for.

## 1. The two mechanisms

**Mechanism A — accounting: converge onto `AccountingNativeSqlExecutor` (S3).**
Module-local execution class mirroring `JdbcTemplate`'s own collaborators and
documented contracts: connection via `DataSourceUtils.getConnection` (same
tx-exposure contract), positional binds via `setNull(Types.NULL)`/`setObject`
(the `StatementCreatorUtils` behaviour for the scalar types used), the SAME
Spring `RowMapper`/`ResultSetExtractor` iteration, the SAME
`SQLErrorCodeSQLExceptionTranslator` on the same DataSource, and mirrored
`queryForObject` size contracts (`EmptyResultDataAccessException` /
`IncorrectResultSizeDataAccessException`) — exception semantics unchanged
(`research/04` row S3-M; `.audits/20261023-0936-s3-jdbc-peel/S3-evidence.txt`
READING/MECHANISM block). WHY not just keep a `JdbcTemplate`: two measured
constraints — (i) the S3 DoD required the module-wide `jdbcTemplate.` probe to
stay at exactly the 10 non-slice sites, so a new repository holding a
JdbcTemplate instance would have defeated the probe; (ii) the module is
provider-agnostic at compile time (`dependencies.gradle` excludes Hibernate,
EclipseLink jakarta-JPA), ruling out Hibernate work APIs (`S3-evidence.txt`).
The executor was born ZERO-TESTED — the S3 gate DA caught that (F-A P2:
"two-mechanism state + zero-tested executor"); the fix gave it 6 unit pins at
fork `a932d91` with a red-capability proof (bind-index mutation flips exactly
the two bind pins, `f5-executor-red-capture.log`).

**Mechanism B — charge: RELAX-in-provider, `ChargeSqlReadRepository` wrapping
the SAME templates (S4b).** The charge read side's 14 `JdbcTemplate` + 3
`NamedParameterJdbcTemplate` sites moved behind a new provider-internal
repository that WRAPS the same Spring template instances the service already
received (`7fd7b92` `d666cee` `83dc366`; mechanism row `research/05` S4-B). No
new executor, no charge→accounting dependency edge. WHY not adopt mechanism A —
the row's own one-liner: the pin-honesty law (move ZERO pins, change ZERO
assertions) means the peeled code passes UNCHANGED only if the SAME collaborator
instances the S4a pins mock stay behind the seam — an executor would REPLACE
those calls and force every stub re-wired (the M1/M3 pin-capture point in
reverse). Plus: charge must not couple to an accounting-module mechanism while
the terminus question was undecided, and the dispatch ruling FORBADE the
charge→accounting edge (`S4b-evidence.txt` carries the verbatim ruling excerpt).
"RELAX" = the probe target was relaxed for the charge slice (0 dotted sites in
the service, 17 in the repository — `S4b-evidence.txt` service/repo probes),
because correctness there is carried by byte-identity, not by mechanism change.

The two-mechanism state was recorded, not hidden: S4-B names it ("after this
pass the CHARGE slice reads via wrapped JdbcTemplate and the ACCOUNTING slice via
AccountingNativeSqlExecutor — recorded honestly; S4-T's accounting-side terminus
remains open").

## 2. The terminus: what CONVERGED means

The S4-T row (written as the S3 hand-off decision, executed at S4c —
`research/05`): all ten remaining dotted `jdbcTemplate.` sites in accounting —
ProvisioningEntries x6, GLClosure x2, FinancialActivityAccount x2 — peeled
behind module repositories ON the executor, pins-first per cluster (M1 §3), and
this is what the row's "replace, not coexist" requirement bought:

- **WHAT IT REPLACED (named, per the row):** the coexisting dotted-JdbcTemplate
  SQL path inside accounting. The three services now hold no SQL and no JDBC
  collaborator; the module probe `grep -r "jdbcTemplate\."
  fineract-accounting/src/main` went **10 → 8 → 6 → 0** at peels C2/C4/C6
  (`S4-cd-evidence.txt` BOUNDARIES; re-measured at the final gate: 0 dotted
  `.query/.update/.execute` anywhere in accounting main, GW TEST-c4 GATE 3).
- **Why converge was cheap HERE and relax held for charge:** mechanism A's pins
  had been captured at the DataSource-level JDBC seam (below the mechanism), so
  swapping the mechanism moved zero assertions — the peel cost only the
  `newService()` construction helper and its import lines per test class; the
  charge pins had been captured at the template mocks (above the mechanism),
  where a swap means re-stubbing —
  exactly what the pin-honesty law forbids. The capture point chose the
  mechanism's future; M1's ordering law is what made CONVERGE the dispatchable
  leg while RELAX remained the honest charge answer.

## 3. The honest residual — what probe 0 does NOT mean

One `JdbcTemplate` **instance** survives inside
`ProvisioningEntriesReadRepository`: passed solely as the argument to the shared
`PaginationHelper.fetchPage` calls (core-infra executes the count+page there).
Retiring it needs a `PaginationHelper` mechanism change in `fineract-core` —
frozen, out of slice scope — carried for S5/owner (named in `research/05` S4-T
"NAMED RESIDUAL"; site list re-verified at the final gate: the instance appears
only as fetchPage arg at `ProvisioningEntriesReadRepository:294, :337`, GW
TEST-c4 GATE 3). A second, unrelated instance the gate hunt surfaced:
pre-existing `UpdateTrialBalanceDetailsTasklet` news a JdbcTemplate and passes it
as an UNUSED parameter — zero SQL through it, zero dotted calls (GW
TEST-c4 FINDINGS 2). Both are invisible to the probe by construction.

## 4. Why "probe zero" must name its measurement

The probe counts **dotted call sites**. It says nothing about instances — and the
moment the number got into prose without its measurement, prose upgraded the
proxy into a property: the executor's own class javadoc claimed the module
"keeps no JdbcTemplate instance". The final gate's DA-c5 leg caught that as a
LIE in a docstring; the fix is fork `618896c` (javadoc-only, proven 0
non-comment diff lines) restating exactly two things — the converged rule (no
accounting-owned dotted JdbcTemplate SQL path, probe 0) and the ONE named
residual. Companion `8f768f0` had already corrected a sibling javadoc that still
claimed the fetchPage hand-offs "stay in the service" when the peel had MOVED
them into the repository — its commit SUBJECT records the proof (javadoc-only,
zero executable change); the paired F-DA-2 rider on the executor javadoc closed
in the same wave (GW `DONE.md` cycles 4–5 addendum: "Riders closed at close:
F-DA-2 javadoc (618896c)").

**The rule this installs:** every green number is a claim about one measured
quantity — print the quantity next to the number ("dotted call sites: 0"), and
list what the measurement cannot see (instances-as-arguments, unused instances)
in the same breath. A residual named at the probe is a known boundary; an
unnamed one is the next lie waiting to be written into a docstring.

## 5. Carry-away rules

1. Prefer one mechanism per concern; when two must coexist mid-refactor, NAME
   the state, DATE the decision, and record the deciding options (rows S4-T /
   S4-B are three sentences of prose that saved the next stage from re-litigating).
2. Mechanism choice follows the pin-capture point: pins below the mechanism →
   converge cheaply; pins above it → relax honestly, or pay re-stubbing and
   admit it as a behavior-touching change.
3. "Replace, not coexist": a terminus row must name what the new path RETIRED
   and show the probe moving to zero site by site (10→8→6→0), not just at tip.
4. Name the measurement beside every probe count, and state the probe's blind
   spots as first-class residual rows — with owner or stage carried, never by
   implication.
5. Docstrings live in the blast radius of refactors: gate them like code
   (DA-c5's catch was a docstring; its fix was proven comment-only, mechanically).

Evidence: `research/04` S3-M, `research/05` rows S4-B/S4-T/S4-D; fork `416a4ae`
… `a932d91` (mechanism A + its pins), `7fd7b92`…`83dc366` (mechanism B),
`1131677`…`89f6f37` (converge), `8f768f0` `618896c` (docstring truthing);
`S3-evidence.txt`, `S4b-evidence.txt`, `S4-cd-evidence.txt`,
GW `TEST-verdict-c4.md` + `DA-verdict-c5.md` (F-DA-2). All TESTED; the
`PaginationHelper` retirement and the false-valued-flag disposition ride as
owner decisions (M5).
