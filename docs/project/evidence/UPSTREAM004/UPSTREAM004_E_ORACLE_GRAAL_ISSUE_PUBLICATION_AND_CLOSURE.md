# UPSTREAM004-E — Oracle/Graal issue publication and closure evidence

Status: **EXTERNAL ISSUE OPENED; UPSTREAM004 CLOSURE READY**

This durable, non-normative record supports
`guillermomolina/protos#753`.

## External publication

The project owner reviewed the prepared Truffle issue draft and independently
submitted the report upstream.

```text
UPSTREAM_REPOSITORY=oracle/graal
UPSTREAM_ISSUE=oracle/graal#14579
UPSTREAM_URL=https://github.com/oracle/graal/issues/14579
UPSTREAM_TITLE=Bytecode DSL root fails Tier-2 runtime compilation in Native Image with FrameWithoutBoxing materialization
UPSTREAM_STATE=OPEN
UPSTREAM_LABEL=truffle
UPSTREAM_CREATED_AT=2026-10-01T05:47:14Z
SUBMITTER=guillermomolina
```

The published issue was re-read from GitHub after submission.

## Published technical claim

The upstream issue reports the independently reproduced minimum boundary:

```text
BYTECODE_DSL_ROOT=MiniPlainRoot
ENABLE_YIELD=false
CUSTOM_OPERATIONS=NONE
LOCALS=NONE
LOOPS=NONE
GUEST_RESULT=1

SAME_25_5_5_SNAPSHOT_JVM_TIER2=PASS
SAME_25_5_5_SNAPSHOT_NATIVE_TIER2=FAIL_FRAMEWITHOUTBOXING
```

The Native failure remains:

```text
Object of type Lcom/oracle/truffle/api/impl/FrameWithoutBoxing;
should not be materialized
(must not pass virtual object into an invoke that cannot be inlined)
```

The report does not claim a specific compiler fix or internal root cause beyond
the retained evidence.

## Version authority retained upstream

The published report includes both the easy released reproduction and the
current-snapshot verification:

```text
RELEASE_CONTROL=GraalVM CE 25.3.4.1+1.1
LATEST_TESTED_DEVELOPER_BUILD=25.5.5-dev-20260930_0129
LATEST_TESTED_GRAALVM=25.5.5-dev+1.1
TRUFFLE_GRAAL_MAVEN_PLANE=25.5.5-SNAPSHOT
ORACLE_GRAAL_REVISION=11b21fb2e5691e49d46525dac7b87ef33aeafe82
```

## Reproducer attachment

The upstream issue contains the reviewed standalone reproducer attachment:

```text
ATTACHMENT=upstream004-oracle-graal-reproducer.tar.gz
ATTACHMENT_URL=https://github.com/user-attachments/files/32888939/upstream004-oracle-graal-reproducer.tar.gz
ATTACHMENT_SHA256=d02b26d8c90a095d4e0d41718889d7e7f654493862c57369adcb38b177289e5e
ATTACHMENT_CONTAINS_PROTOS_SOURCE=NO
```

The package identity matches UPSTREAM004-D.

## AI-assistance disclosure

The submitted issue explicitly contains:

> Disclosure: This report was prepared with AI assistance. I personally ran and
> verified the reproducer, commands, logs, and conclusions, and I take
> responsibility for the submission.

This is consistent with the current `oracle/graal/CODING_ASSISTANTS.md`
policy retained during UPSTREAM004 preparation.

## Relationship to Protos

The external issue completes the upstream-reporting scope only.

It does **not** repair the Protos product state:

```text
BUG013=OPEN
DIST009=BLOCKED
NATIVE_FORCED_GUEST_TIER2=FAIL
HARDENED_NATIVE_GATE_REVISION=7770aa135bb65c7219de5dbe9e2b3ac22cdaf31a
```

BUG013 remains responsible for restoring a Protos configuration/implementation
that satisfies the maintained Native Tier-2 admission gate. That may be done by
a bounded rollback or narrowing that avoids exposing the upstream defect while
`oracle/graal#14579` remains open.

DIST009 must remain blocked until BUG013 satisfies its Native admission
acceptance criteria. The external report does not justify weakening or skipping
the gate.

## UPSTREAM004 completion

All UPSTREAM004 deliverables are now satisfied:

```text
EPHEMERAL_REPRODUCER_PRESERVED=YES
REPRODUCER_FILE_HASHES_RETAINED=YES
ENVIRONMENT_IDENTITY_RETAINED=YES
LATEST_SNAPSHOT_NATIVE_FAILURE_RETAINED=YES
SAME_SNAPSHOT_JVM_CONTROL_RETAINED=YES
MINIMAL_PLAIN_ROOT_REPRODUCES=YES
FULL_REPRO_COMMANDS_VERIFIED=YES
UPSTREAM_TEMPLATE_SELECTED=TRUFFLE_ISSUE_REPORT
DRAFT_REVIEWED_BY_HUMAN=YES
REPRODUCER_ATTACHMENT_RETAINED=YES
EXTERNAL_ISSUE_OPENED=YES
EXTERNAL_ISSUE=oracle/graal#14579
```

Formal closure evidence:

```text
ISSUE_CLOSURE_COMMENT=READY
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS_AFTER_THIS_RECORD_IS_PUBLISHED_AND_REREAD
```

No further UPSTREAM004 work is required unless upstream requests additional
reproducer information. Such follow-up may be logged against the closed item or
allocated separately if it becomes independently meaningful project work.
