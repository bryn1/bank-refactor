PREMISE: VM350 / qwen3.8-flash-next / Lane E / VM350 as of 2026-10-22

# 01 — Target selection: public OSS legacy-style banking backends

MC 11126 (display 11121.1.3, [type:research]). Scope: target selection only; the feasibility plan is a later child.
Method: shallow clones (`--depth=1`) under `.tmp/clones/`, LOC counted with `find | wc -l` on the clone
(no cloc on this host — VERIFIED absent via `which cloc tokei`), repo metadata via GitHub/GitLab REST APIs and
raw.githubusercontent fetches, all measured this session 2026-10-22. Commits pinned: Fineract `355287c8`,
mainframe-bank `d3123224`. Evidence tags: VERIFIED (measured/fetched here, path or URL given) / INFERRED (reasoned).

## 1. Comparison matrix

| Candidate | Lang + era | Size (method) | Activity | License (from file/API) | Tests | Build (sandbox-plausible?) | Staged-refactor fit |
|---|---|---|---|---|---|---|---|
| **apache/fineract** | Java 25, true legacy lineage (Mifos mFinance era code still in tree; INFERRED), active | 919,408 LOC / 7,047 java files; provider main 278,878 (find\|wc on clone @355287c8) | pushed 2026-10-22 (VERIFIED API + clone `git log -1`) | Apache-2.0 (`LICENSE_SOURCE` in clone, VERIFIED head quote) | 1,002 `*Test*.java`, 99 `*IntegrationTest*` (find, VERIFIED) | Gradle wrapper + Java 25 + Spring Boot 4.1.1 (VERIFIED build.gradle); README docker-compose build plausible but NOT run here (INFERRED) | **Strong**: modular (per-module Gradle), module slices sized 3.9k–14.3k LOC ≈ 1–3 seat-days |
| **seyhunak/mainframe-bank** | GnuCOBOL 3.2 free-format, written 2026 as a *re-enactment* of a 1980s/90s shop (single commit, README quote) | 5,799 LOC in 18 .cbl + 619 lines in 24 .cpy copybooks (find\|wc @d3123224) | pushed 2026-10-10 (VERIFIED clone `git log -1`) | **NONE — no LICENSE file** (instrument: `git ls-tree -r HEAD \| grep -ci licen` = 0; GitHub API license=null) | 7 test files `test/T*.cbl` (VERIFIED ls-tree) | shell `bin/BUILD` + GnuCOBOL; plausible if gnucobol installed, not installed here (INFERRED) | Size/scope ideal, but **no license ⇒ no derivative permission — disqualified** |
| **OpenCBS/OpenCBS-Cloud** | TypeScript (NestJS) + Java + PLpgSQL, 2022-era rewrite, NOT legacy-style | 3,834 KB git size; languages-by-bytes API: TS 2.95 MB, Java 2.73 MB | dormant: created 2022-12-01, last push 2023-01-27 (VERIFIED API) | GPL-3.0 (VERIFIED API license endpoint, full GPLv3 text returned) | not measured (shallow-clone skipped, time-box) | npm/gradle mixed (INFERRED) | Modern stack, young code — wrong corpus for a *legacy* study; strong runner-up |
| **apache/ofbiz-framework** (accounting module) | Java + Groovy/FTL, legacy ERP, active | Java 17,445,969 bytes by languages API (≈ >500k LOC, no clone — too big to shallow-clone in budget) | pushed 2026-10-22 (VERIFIED API) | Apache-2.0 (VERIFIED, fetched `raw.githubusercontent.com/apache/ofbiz-framework/trunk/LICENSE`, quotes "Apache License Version 2.0") | not measured (whole ERP) | Gradle (VERIFIED via Makefile/gradle in languages list, INFERRED otherwise) | ERP/GL not bank-core; monolith entanglement; rejected as too big + wrong domain |

## 2. RECOMMENDED TARGET

RECOMMENDED TARGET: https://github.com/apache/fineract — staged study scoped to per-module slices, first slice `fineract-charge` (3,946 LOC / 39 files, VERIFIED find|wc) then `fineract-accounting` (14,314 LOC / 174 files; only 3 module test files — VERIFIED).

Justification (all VERIFIED on clone `355287c8` @ /home/claudecode/bank-refactor/.tmp/clones/fineract unless tagged):
- **Real legacy production core banking, not a toy**: 919,408 Java LOC; active upstream (HEAD commit dated 2026-10-22); Mifos-mFinance lineage (INFERRED — widely documented, not re-fetched here).
- **Slices sized to the seat-day rule**: top-level Gradle modules are pre-cut — `fineract-charge` 3,946 LOC ≈ one seat-day; `fineract-accounting` 14,314 LOC ≈ 2–3; `fineract-savings` 26,482. A later slice needs no artificial boundary-drawing.
- **Measured architectural smells, named files** (god classes / transaction script / JDBC-in-service mix):
  `fineract-core/src/main/java/org/apache/fineract/commands/service/CommandWrapperBuilder.java` — 4,221 lines;
  `fineract-provider/src/main/java/org/apache/fineract/portfolio/loanaccount/service/LoanWritePlatformServiceJpaRepositoryImpl.java` — 3,703 lines;
  `fineract-provider/src/main/java/org/apache/fineract/portfolio/loanproduct/serialization/LoanProductDataValidator.java` — 3,024 lines;
  94 `*ServiceImpl` files under `fineract-provider/src/main` call `jdbcTemplate.query/execute` directly (VERIFIED grep count) — the classic raw-JDBC-beside-JPA smell layer to peel.
- **Safety net for refactoring**: 1,002 test files incl. 99 integration tests (VERIFIED find) — characterization tests already exist for behavior-preservation gates.
- **License permits exactly our use**: Apache-2.0 allows private derivative refactoring + a public process write-up; if refactored code is ever published, keep LICENSE/NOTICE + attribution (plain Apache-2.0 terms, VERIFIED `LICENSE_SOURCE` head).

## 3. Rejected alternatives (one line each)

- **ATG Teller / "Teller38" COBOL estate — lead FAILS, finding**: `github.com/Atgeit/teller` does not exist (GitHub API `repos/Atgeit/teller` → 404, VERIFIED). No public Teller/38 repo found (GitHub+GitLab search APIs, firecrawl, HN Algolia — all empty, VERIFIED as searched). `gitlab.com/atg38/teller38` API → 404 "Project Not Found"; web returns 302→sign-in 403 identically for a control bogus path, so private-vs-nonexistent is indistinguishable anonymously — either way it is not publicly readable today.
- **seyhunak/mainframe-bank** — no license at all ⇒ default all-rights-reserved ⇒ private derivative + public write-up not lawfully granted; also it is a 2026 re-enactment (single commit), not true legacy. Keep on watchlist only if the author adds a license.
- **OpenCBS/OpenCBS-Cloud** — modern NestJS rewrite (created 2022-12), dormant 2+ years; not a legacy corpus; GPL-3.0 fine but wrong material for the stated study.
- **apache/ofbiz-framework (accounting)** — Apache-2.0 and huge, but it is ERP general-ledger entangled with the whole OFBiz framework, no deposit/lending core; too big to clone in budget (775 MB repo, VERIFIED API `size`).
- **Mifos mFinance / mPoint / mifos-platform era code** — converged: `openmf/mifos-platform` → 404 (VERIFIED API); `mifos/mifos` is an empty gitter-bridge test repo (VERIFIED API description "Empty github repo for testing gitter IRC bridge"); the lineage's living code IS Fineract.
- **GNU Finance / gnoferrari** — `pipacs/gnoferrari` → API 404 and `git clone` 404 (VERIFIED); project defunct on GitHub.
- **Java EE / JBoss bank reference apps (dukesbank et al.)** — official `javaee-samples/dukesbank` → 404 (VERIFIED API); surviving forks (e.g. `brada-vmware/dukesbank`, 2014) carry no license and are tutorial-scale demos — toy + unlicensed.

## 4. License per candidate — private derivative refactoring + public process write-up?

| Candidate | License (evidence) | Private derivative refactor? | Public write-up of the process? |
|---|---|---|---|
| apache/fineract | Apache-2.0 — `LICENSE_SOURCE` in clone (VERIFIED) | Yes (VERIFIED by license terms) | Yes; publishing refactored code keeps Apache-2.0 attribution/NOTICE obligations |
| seyhunak/mainframe-bank | none — `git ls-tree` license grep = 0 (VERIFIED) | **No** (unlicensed = all rights reserved) | Write-up about it is fine, but no code-derivative study |
| OpenCBS/OpenCBS-Cloud | GPL-3.0 — API license endpoint, full text (VERIFIED) | Yes (private study = no copyleft trigger) | Yes; distributing the refactored code would trigger GPL source-provision |
| apache/ofbiz-framework | Apache-2.0 — fetched trunk/LICENSE (VERIFIED) | Yes | Yes |
| ATG Teller/38 | unknown/unreachable (see §3) | cannot grant — repo not publicly readable | n/a today |

## Caveats / limitations
- LOC figures are `find|wc -l` on depth-1 clones — includes comments/blank lines, excludes .git; method stated per row. OFBiz/OpenCBS were NOT cloned (byte-sizes from GitHub languages API only) — their LOC cells are byte-counts, not lines.
- Build feasibility was NOT executed anywhere (constraint: no toolchain installs) — every "plausible" is INFERRED from build files read this session.
- The mFinance→Fineract lineage statement is INFERRED (documented publicly, not re-fetched in this session's budget).
- Clones retained for parent verification: `.tmp/clones/fineract` (138 MB), `.tmp/clones/mainframe-bank`. Artifact left uncommitted for the parent to integrate.
