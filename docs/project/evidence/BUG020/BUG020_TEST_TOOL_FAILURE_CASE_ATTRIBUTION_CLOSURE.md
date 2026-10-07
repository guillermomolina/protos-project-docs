# BUG020 — Test Tool failing logical Case attribution repair and closure evidence

Date: 2026-10-06

This snapshot records the published repair and closure evidence for
`guillermomolina/protos#812` (BUG020). It is durable non-normative project
evidence; observable Protos and Test Tool contracts remain owned by
`guillermomolina/protos`.

## Stable identities

```text
FORMAL_WORK_ITEM=BUG020
GITHUB_ISSUE=guillermomolina/protos#812
PROTOS_REVISION=0e3ad479c65a382b71f74723b0a170a5ad742fa2
PROTOS_VERSION=0.3.248-SNAPSHOT
PRODUCT_COMMIT=BUG020: name the failing logical Case in Test Tool errors

CONTRACT_AUTHORITY=D120/#455
RESULT_AUTHORITY=D155/#615
SEMANTIC_CHANGE=NO
```

## Original symptom

A directory-scoped Test Tool run could advance through bounded aggregate
progress and then terminate with only a generic Tool-level diagnostic:

```text
[library/datetime] 42/56
Test tool error: Object {}
  at /workspaces/protos/protos/tools/test/Main.protos:684:13
  at /workspaces/protos/protos/tools/test/Main.protos:682:5
  at /workspaces/protos/protos/tools/test/Main.protos:525:1
```

That output did not identify which logical Case triggered the failure.

The behavior contradicted the already-ratified D120 presentation boundary,
which requires immediate failure visibility with exact Case identity while
keeping normal passing output bounded. BUG020 therefore required no new Tool
presentation decision.

## Root cause boundary

The suite-native `LogicalCaseRunner` normally obtains a Case completion and
then invokes the terminal lifecycle and completion observers that feed the
existing D120 progress/reporting path.

The failing path is different: a logical Case execution Future can signal while
`.value()` is being obtained, before a D155 completion object exists. In that
path the runner never reaches its ordinary completion observer, so the Case
entry is still known locally but the existing D120 Case-attribution machinery
is bypassed. The Error then propagates outward and is eventually presented by
the generic Test Tool error boundary without a Case name.

## Published repair

Exact Protos revision
`0e3ad479c65a382b71f74723b0a170a5ad742fa2`
(`0.3.248-SNAPSHOT`) publishes the bounded repair.

`LogicalCaseRunner.run` now accepts an optional `failureObserver`. The
runner wraps only the execution-Future `.value()` boundary. If that boundary
signals:

1. the observer receives the stable logical `index`, the authoritative Case
   `entry`, and the original `error`;
2. `Main.protos` renders the already-existing `CaseRef.display(entry)`;
3. stderr receives:

   ```text
   Test case error: <corpus>:<source>::<selector>
   ```

4. the exact same Error is re-signalled unchanged.

The repair therefore adds attribution without converting the Error, changing
its terminal policy, manufacturing a D155 completion, or hiding the existing
outer `Test tool error` diagnostic.

## Preserved behavior

The published change deliberately preserves:

- D120 bounded aggregate progress;
- compact passing output;
- D155 Case-result classification for paths that do produce completions;
- existing Tool-level failure and exit behavior for the no-completion path;
- logical result ordering;
- `--jobs` and bounded scheduler behavior;
- per-Case isolation and captured output policy;
- the existing CaseRef display format;
- the original Error object and propagation path; and
- the Protos language and Standard Library specifications.

No logging architecture, terminal UI, retry/timeout policy, assertion semantics,
or scheduler redesign is introduced.

## Published product files

The exact BUG020 product commit changes:

```text
CHANGELOG.md
pom.xml
protos/tools/test/LogicalCaseRunner.protos
protos/tools/test/Main.protos
```

The root changelog records the new observable diagnostic prefix and explicitly
states that Error, exit status, progress output, scheduling, and specification
semantics remain unchanged.

## Validation provenance

After the product commit was pushed, the maintainer reported:

```text
PUSH_TO_MAIN=PASS
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

No raw local test logs, individual test names, or exact test counts were supplied
with the closure handoff, so this record does not fabricate them.

The published commit itself is the stable product identity for the repaired
behavior.

## Closure result

```text
FAILING_CASE_IDENTITY_VISIBLE=PASS
ORIGINAL_ERROR_RE_SIGNALLED_UNCHANGED=PASS
UNATTRIBUTED_OBJECT_ONLY_FAILURE=REMOVED
D120_BOUNDED_PROGRESS_PRESERVED=PASS
D155_EXISTING_COMPLETION_CLASSIFICATION_PRESERVED=PASS
EXIT_POLICY_CHANGED=NO
SCHEDULING_SEMANTICS_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
SPECIFICATION_CHANGE=NO
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
NEXT_TECHNICAL_SLICE=NONE
BUG020_CLOSE_READY=YES
```

AI assistance: this evidence record was drafted with ChatGPT from the live
BUG020 Issue, the maintainer's publication/validation report, and exact
inspection of Protos revision
`0e3ad479c65a382b71f74723b0a170a5ad742fa2`.
