# BUG019 — Native release version identity repair and closure evidence

Date: 2026-10-06

This snapshot records the published BUG019 repair, finalization reconciliation,
validation, and closure evidence for `guillermomolina/protos#811`. It is durable
non-normative project evidence; the product implementation and live Issue remain
authoritative in `guillermomolina/protos`.

## Stable identities

```text
FORMAL_WORK_ITEM=BUG019
GITHUB_ISSUE=guillermomolina/protos#811

IMPLEMENTATION_REVISION=7b9e629bddce9d501a2773a9f3a3973c8de37885
IMPLEMENTATION_COMMIT=BUG019: report exact Native distribution version identity
IMPLEMENTATION_VERSION=0.3.250-SNAPSHOT

CHANGELOG_RECONCILIATION_REVISION=83f4319132eb38d0e528475293d095a8bca9a60e
CHANGELOG_RECONCILIATION_COMMIT=BUG019: reconcile missing changelog entry

SEMANTIC_CHANGE=NO
```

## Original defect

The defect was discovered from the published Native 0.3.237 distribution, whose
public launcher selected the correct artifact but whose Native payload reported:

```text
protos --version
Protos development
```

The Native Image does not inherit the shaded-JAR
`Implementation-Version`, so the JVM-oriented package metadata path was absent
in the Native executable.

The already-published 0.3.237 artifact remains immutable and unchanged. BUG019
repairs distributions built from the repaired product source onward.

## Published repair

Exact Protos revision
`7b9e629bddce9d501a2773a9f3a3973c8de37885` publishes the product repair across:

```text
bin/protos
dist/test_native_launcher.py
dist/validate_native.py
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
```

The Native distribution launcher reads the checksum-covered
`implementation_version` from `SOURCE.txt` using shell builtins only, publishes
that value to the Native payload through
`PROTOS_IMPLEMENTATION_VERSION`, and overwrites any forged caller-provided value.

The JVM path explicitly clears that Native-only transport and therefore continues
to obtain its packaged version from the JAR manifest.

`ProtosCli` resolves public version identity in this order:

```text
1. packaged JVM Implementation-Version
2. Native distribution PROTOS_IMPLEMENTATION_VERSION
3. development
```

The genuine development fallback is therefore preserved when neither packaged
identity exists.

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

`dist/validate_native.py` reads the extracted archive's `SOURCE.txt`, executes
the public launcher with `--version`, and requires exact stdout:

```text
Protos <implementation_version>\n
```

The admission path publishes:

```text
NATIVE_DIST_EXACT_VERSION_IDENTITY_CHECK: PASS
```

rather than accepting only the historical `Protos ` prefix.

## Validation provenance

For the published implementation, the maintainer reported:

```text
PUSH_TO_MAIN=PASS
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

For the subsequent changelog reconciliation, the maintainer again reported:

```text
PUSH_TO_MAIN=PASS
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

No raw local test logs or exact test counts were supplied with either handoff,
so this record does not fabricate them.

## Finalization reconciliation

The implementation commit correctly incremented `pom.xml` from
`0.3.249-SNAPSHOT` to `0.3.250-SNAPSHOT`, but omitted the matching root
`CHANGELOG.md` section required by `AGENTS.work/IMPLEMENTATION.md`.

Because the implementation commit was already published, history was not
rewritten. A later bounded reconciliation commit,
`83f4319132eb38d0e528475293d095a8bca9a60e`, changes only
`CHANGELOG.md` and adds the missing `0.3.250-SNAPSHOT` BUG019 entry in
historical version order.

The reconciliation records that:

- Native distributions built from the repaired source report their exact
  packaged Protos version;
- the Native launcher takes the version from distribution `SOURCE.txt` and
  overrides caller-provided identity;
- JVM distributions retain JAR-manifest identity;
- genuine development execution retains `Protos development`;
- exact Native version-output admission is required;
- already-published artifacts are unchanged; and
- no specification change occurred.

This preserves the historical fact that the original implementation commit
omitted the changelog while closing the repository-policy discrepancy with a
non-destructive follow-up commit.

## Closure result

```text
BUG019=COMPLETE
NATIVE_RELEASE_VERSION_IDENTITY=PASS
JVM_VERSION_IDENTITY_PRESERVED=PASS
DEVELOPMENT_FALLBACK_PRESERVED=PASS
CALLER_FORGED_NATIVE_IDENTITY_REJECTED=PASS
NATIVE_EXACT_VERSION_ADMISSION=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
CHANGELOG_RECONCILIATION=PASS
BUG019_TECHNICAL_IMPLEMENTATION=COMPLETE
BUG019_FORMAL_CLOSE_READY=YES
NEXT_TECHNICAL_SLICE=NONE
```

AI assistance: this evidence record was drafted and reconciled with ChatGPT from
the live BUG019 Issue, the maintainer's publication/validation reports, and exact
inspection of Protos revisions
`7b9e629bddce9d501a2773a9f3a3973c8de37885` and
`83f4319132eb38d0e528475293d095a8bca9a60e`.
