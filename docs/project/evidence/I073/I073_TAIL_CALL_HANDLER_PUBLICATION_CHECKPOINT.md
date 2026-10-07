# I073 — Bytecode DSL tail-call-handler adoption publication checkpoint

Status: **COMPLETE**

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
I073_STATUS=COMPLETE
I074_STATUS=READY

NEXT_SLICE=I074
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

## Repair publication and closure

The forward-only metadata repair was subsequently published:

```text
REPAIR_REVISION=41a7a6e0bc08ef3d29f8f591c48af5e45821900e
REPAIR_PARENT=f206eddf91fade472acb0f058a4729c9a4dca27d
REPAIR_FILES=CHANGELOG.md,pom.xml
REPAIR_SOURCE_CHANGE=NO
MAVEN_VERSION=0.3.123-SNAPSHOT
CHANGELOG_I073_ENTRY=PASS
```

The repair commit changes no executable source. It increments the Maven
implementation version from `0.3.122-SNAPSHOT` to `0.3.123-SNAPSHOT` and
adds the required I073 entry to the root `CHANGELOG.md`.

The I073 implementation is therefore represented by the two consecutive
published revisions:

```text
SUBSTANTIVE_REVISION=f206eddf91fade472acb0f058a4729c9a4dca27d
FINALIZATION_REVISION=41a7a6e0bc08ef3d29f8f591c48af5e45821900e
```

Together with the project-owner-reported `make test=PASS`, this satisfies the
I073 closure evidence.

```text
TAIL_CALL_HANDLER_GENERATION=ENABLED
OBSERVABLE_PROTOS_SEMANTICS=UNCHANGED
MAKE_TEST=PASS
CLEAR_PERFORMANCE_REGRESSION=NOT_REPORTED
PUBLICATION=PASS
PUBLICATION_METADATA=PASS
I073_CLOSURE=PASS
```

I074 may now advance to READY.
