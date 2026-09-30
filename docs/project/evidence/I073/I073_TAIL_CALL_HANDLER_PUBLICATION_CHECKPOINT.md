# I073 — Bytecode DSL tail-call-handler adoption publication checkpoint

Status: **SUBSTANTIVE IMPLEMENTATION PUBLISHED — PUBLICATION METADATA REPAIR REQUIRED**

This durable, non-normative evidence record preserves the exact state after the
first I073 product publication.

## Identity

```text
DATE=2026-09-30

WORK_ITEM=I073
ISSUE=guillermomolina/protos#744
PARENT=PERF011 / guillermomolina/protos#693

PROTOS_REVISION=f206eddf91fade472acb0f058a4729c9a4dca27d
PARENT_REVISION=738e2b9f5d8101f4229542afbaf8f8689785f680
```

## Published substantive delta

The exact commit changes one file and one line:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java

+ enableTailCallHandlers = true
```

GitHub commit statistics:

```text
FILES_CHANGED=1
ADDITIONS=1
DELETIONS=0
```

No Protos language or Standard Library semantics were changed.

## Validation

The project owner reported in the active coordination session that the required
`make test` gate was run before allowing publication and passed:

```text
MAKE_TEST=PASS
```

No separate GitHub Actions/status check is associated with the commit.

## Publication-metadata discrepancy

At exact product revision
`f206eddf91fade472acb0f058a4729c9a4dca27d`:

```text
MAVEN_VERSION=0.3.122-SNAPSHOT
CHANGELOG_TOP=0.3.122-SNAPSHOT / I062
```

The substantive I073 source publication did not include a Maven implementation
version increment or a matching root `CHANGELOG.md` entry.

Current `AGENTS.work/IMPLEMENTATION.md` requires every committed executable
implementation change under `src/` to include exactly one implementation
version increment and a matching changelog section in the same final commit.

The published I073 source change is therefore functionally validated but does
not yet satisfy the repository publication contract.

Repository policy also forbids rewriting published history or force-pushing.
The remaining reconciliation is a bounded forward repair rather than history
rewriting.

## Required repair

At the observed repair baseline, `main` is still the I073 commit and
`pom.xml` remains `0.3.122-SNAPSHOT`.

The smallest repair is:

```text
pom.xml:
  0.3.122-SNAPSHOT -> 0.3.123-SNAPSHOT

CHANGELOG.md:
  add 0.3.123-SNAPSHOT entry describing I073

SOURCE_CHANGE:
  NONE

SPEC_CHANGE:
  NONE
```

The already-reported behavioral `make test` PASS need not be repeated merely
for metadata-only finalization unless repository movement or another change
invalidates that evidence.

## Routing

```text
I073_STATUS=IN_PROGRESS
I074_STATUS=PAUSED

NEXT_SLICE=I073 publication-metadata repair
NEXT_SLICE_TYPE=IMPLEMENTATION_FINALIZATION_REPAIR
NEXT_REPOSITORY=guillermomolina/protos
```

I074 must not be advanced until the forward repair is published, the exact
result is re-read, and I073 satisfies its closure gate.
