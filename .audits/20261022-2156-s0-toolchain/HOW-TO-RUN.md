# S0 toolchain — how to run the pinned Fineract build (reusable for S1–S5)

Card MC 11147 (parent 11146). Owner-approved approach: docker-based JDK-25 build
(option R1-i, ruling 2026-10-22 "go with recomended then start"). Host JDK is 21.0.12
and CANNOT build this repo (requires Java 25 at `build.gradle` ~line 418:
`JavaLanguageVersion.of(25)`). All Java execution happens inside `eclipse-temurin:25-jdk`.

## Layout / reading used (recorded per brief)

- **PIN (contract, never written):** `.tmp/clones/fineract` @
  `355287c8feca747b3cd936a5b1cef1bd7804ffea` — byte-identical, pristine (zero new files).
- **BUILD DIR (disposable):** `.tmp/fineract-s0-build` — a `cp -a` copy of the pinned clone,
  built there instead of the clone because Gradle insists on writing `.gradle/` + per-module
  `build/` INTO the project tree. Brief reading: "the PIN is the contract, the dir is disposable".
- **CACHES (host, gitignored):** `.tmp/gradle-home` — Gradle 9.7.1 distribution + dependency
  cache, mounted as `GRADLE_USER_HOME`; nothing written outside `bank-refactor/.tmp/`.

## One-time setup (done in S0, idempotent)

```bash
cd /home/claudecode/Kassan/bank-refactor
docker pull eclipse-temurin:25-jdk
mkdir -p .tmp/gradle-home
cp -a .tmp/clones/fineract .tmp/fineract-s0-build     # disposable build dir
```

Re-sync the build dir to the pristine pin (discards any S1+ edits in it):

```bash
rm -rf .tmp/fineract-s0-build && cp -a .tmp/clones/fineract .tmp/fineract-s0-build
```

## The build command (the S0-S5 harness)

```bash
cd /home/claudecode/Kassan/bank-refactor
docker run --rm --user 1000:1000 \
  -e GRADLE_USER_HOME=/work/gradle-home -e HOME=/work/gradle-home \
  -e VERSIONING_DISABLE=true \
  -v "$PWD/.tmp/fineract-s0-build:/work/fineract" \
  -v "$PWD/.tmp/gradle-home:/work/gradle-home" \
  -w /work/fineract \
  eclipse-temurin:25-jdk \
  ./gradlew --no-daemon --console=plain <TASKS>
```

- `--user 1000:1000` — container writes files as the host `claudecode` uid, so git stays
  operable and no root-owned files leak into the tree.
- `-e HOME=/work/gradle-home` — temurin image default HOME (`/`) is not writable for uid 1000;
  point HOME into the mounted cache dir.
- `-e VERSIONING_DISABLE=true` — REQUIRED. The pinned clone is a **shallow clone with no
  tags**, and the repo's `GitVersionValueSource` (build.gradle ~147) hard-fails configuration
  with "couldn't find matching tag in shallow git repository" (observed S0 run 1,
  BUILD FAILED in 2m 20s). This env is the repo's own escape hatch (`commandOption` reads
  `VERSIONING_DISABLE` / `-Dversioning.disable`), falls back to version `0.0.0-SNAPSHOT`,
  harmless for test/compile tasks, no clone edit and no `git fetch --unshallow` needed.
  Only if a later phase needs the real derived version, run
  `git -C .tmp/fineract-s0-build fetch --unshallow origin && git fetch --tags` IN THE COPY.
- `GRADLE_USER_HOME` on the mounted volume keeps the Gradle distribution + all dependency
  downloads off the repo tree and off the host's `~`.
- `--no-daemon` per brief; use long `run_in_background` jobs for anything past a few minutes.
  Measured S0 on VM350: Gradle 9.7.1 distribution + dependency graph downloaded in ~2 min;
  cold configuration-to-green of both test tasks took 2m10s (warm runs re-use
  `.tmp/gradle-home` and are similar or faster).
- Any Gradle task works in place of `<TASKS>`, e.g.
  `:fineract-accounting:test :fineract-charge:test` (the S0 baseline),
  `:fineract-provider:compileJava`, `check`, etc.

## Dev DB: NOT needed for the S0 baseline

`:fineract-accounting:test` and `:fineract-charge:test` at this pin are **DB-free unit tests**:
5 test classes total, pure JUnit5/Mockito (grep for `SpringBootTest|ContextConfiguration|
Testcontainers|DataSource` returns nothing), and each module's `build.gradle` excludes
`**/core/boot/tests/**` unless `-Penv=dev`. `test { useJUnitPlatform() }` only. Recorded per
brief ("if the chosen test tasks are DB-free, record that instead").

A DB is needed only for `:fineract-provider` boot/integration/e2e work. At this pin the repo
ships `docker-compose-*.yml` at the root for that; README standardizes on **PostgreSQL**
(MariaDB/MySQL are deprecated per FSIP-9) — `./gradlew createPGDB -PdbName=fineract_tenants`
etc., or `docker compose -f docker-compose-development.yml up`. Deal with it when an S-phase
actually boots the provider.

## Proof the pinned clone stayed untouched

```bash
git -C .tmp/clones/fineract rev-parse HEAD    # 355287c8feca747b3cd936a5b1cef1bd7804ffea
git -C .tmp/clones/fineract status --porcelain | wc -l   # 0 (before AND after any build)
```

## Rollback

The toolchain is additive: `rm -rf .tmp/fineract-s0-build .tmp/gradle-home` and delete this
run dir's commits revert the only host-visible footprint. The pinned clone and the repo's
tracked files are never touched by the build itself.
