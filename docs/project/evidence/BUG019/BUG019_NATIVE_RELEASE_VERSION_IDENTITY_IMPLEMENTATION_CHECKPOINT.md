# BUG019 — Native release version identity implementation checkpoint

Date: 2026-10-06

This snapshot records the published BUG019 implementation and validation evidence
for `guillermomolina/protos#811`. It is durable non-normative project evidence;
the product implementation and live Issue remain authoritative in
`guillermomolina/protos`.

## Stable identities

```text
FORMAL_WORK_ITEM=BUG019
GITHUB_ISSUE=guillermomolina/protos#811
PROTOS_REVISION=7b9e629bddce9d501a2773a9f3a3973c8de37885
PROTOS_VERSION=0.3.250-SNAPSHOT
PRODUCT_COMMIT=BUG019: report exact Native distribution version identity
SEMANTIC_CHANGE=NO
```

The defect was discovered from the published Native 0.3.237 distribution, whose
public launcher selected the correct artifact but whose Native payload reported:

```text
protos --version
Protos development
```

The Native Image does not inherit the shaded-JAR
`Implementation-Version`, so the JVM-oriented package metadata path was absent
in the Native executable.

## Published repair

The exact product revision
`7b9e629bddce9d501a2773a9f3a3973c8de37885` publishes the repair across:

```text
bin/protos
dist/test_native_launcher.py
dist/validate_native.py
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
```

The Native distribution launcher now reads the checksum-covered
`implementation_version` from `SOURCE.txt` using shell builtins only, publishes
that value to the Native payload through
`PROTOS_IMPLEMENTATION_VERSION`, and overwrites any forged caller-provided value.

The JVM path explicitly clears that Native-only transport and therefore continues
to obtain its packaged version from the JAR manifest.

`ProtosCli` resolves the public version identity in this order:

```text
1. packaged JVM Implementation-Version
2. Native distribution PROTOS_IMPLEMENTATION_VERSION
3. development
```

The development fallback is therefore preserved when neither packaged identity
exists.

## Strengthened admission

The Native launcher regression covers:

- Native selection before Java;
- exact `PROTOS_HOME`;
- exact propagation of the distribution version;
- preservation of literal arguments;
- rejection of caller-forged Native version identity; and
- unchanged portable JVM routing.

The CLI regression covers package-version precedence, Native-distribution
fallback, and the final `development` fallback.

`dist/validate_native.py` now reads the extracted archive's
`SOURCE.txt`, executes the public launcher with `--version`, and requires exact
stdout:

```text
Protos <implementation_version>\n
```

It publishes the explicit marker:

```text
NATIVE_DIST_EXACT_VERSION_IDENTITY_CHECK: PASS
```

rather than accepting only the old `Protos ` prefix.

## Validation provenance

After publication, the maintainer reported:

```text
PUSH_TO_MAIN=PASS
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

No raw local test log or exact test count was supplied with the handoff, so this
record does not fabricate either.

## Finalization discrepancy

The exact product commit increments `pom.xml` from
`0.3.249-SNAPSHOT` to `0.3.250-SNAPSHOT`, but the same commit does not add the
required `0.3.250-SNAPSHOT` section to the root `CHANGELOG.md`.

`AGENTS.work/IMPLEMENTATION.md` requires an executable implementation-source
commit under `src/` to carry both the implementation-version increment and its
matching root changelog entry in the final commit.

Therefore this evidence records the implementation as technically published and
validated, but does **not** claim formal BUG019 closure yet.

## Checkpoint result

```text
NATIVE_RELEASE_VERSION_IDENTITY_IMPLEMENTED=PASS
JVM_VERSION_IDENTITY_PRESERVED=PASS
DEVELOPMENT_FALLBACK_PRESERVED=PASS
CALLER_FORGED_NATIVE_IDENTITY_REJECTED=PASS
NATIVE_EXACT_VERSION_ADMISSION=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
CHANGELOG_0_3_250_ENTRY=MISSING
BUG019_TECHNICAL_IMPLEMENTATION=COMPLETE
BUG019_FORMAL_CLOSE_READY=NO
REMAINING_STEP=publish matching root CHANGELOG.md entry
```

AI assistance: this evidence record was drafted with ChatGPT from the live
BUG019 Issue, the maintainer's validation/publication report, and exact
inspection of Protos revision
`7b9e629bddce9d501a2773a9f3a3973c8de37885`.
