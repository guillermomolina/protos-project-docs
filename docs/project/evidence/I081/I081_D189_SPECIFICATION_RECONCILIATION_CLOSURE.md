# I081 — D189 normative specification reconciliation closure

Date: 2026-10-07

## Work identity

```text
WORK_ITEM=I081
PROTOS_ISSUE=guillermomolina/protos#829
DECISION_AUTHORITY=D189/guillermomolina/protos#820
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
```

This record is durable non-normative project evidence. It does not replace the
normative Protos specification or the live GitHub Issue state.

## Published product revision

I081 was published as one specification-only commit:

```text
STARTING_PROTOS_REVISION=5e2e459b952583a48b870c84321a9796bf81794f
ENDING_PROTOS_REVISION=44688445543a2cef79b89c67b7145e4d382da6d0
COMMIT_SUBJECT=I081: reconcile D189 foreign callback semantics into normative specification (spec 0.1.446)
SPECIFICATION_REVISION=0.1.446
```

The published commit changes exactly:

```text
spec/PROTOS_SPEC_CHANGELOG.md
spec/concurrency/ACTORS.md
spec/concurrency/FUTURES_AND_TASKS.md
spec/semantics/VALUES_AND_COLLECTIONS.md
```

No runtime, Standard Library, bundled-tool, Java implementation, or test file is
changed by I081.

## D189 contract reconciliation

The published normative owners were re-read at the exact ending revision.

`spec/semantics/VALUES_AND_COLLECTIONS.md` now owns the baseline
**Synchronous foreign callbacks** contract and records that:

- the callback denotes the exact Protos semantic value;
- a Closure callback uses ordinary Closure activation semantics, preserving
  lexical environment, receiver, `methodHome`, return home, and non-local
  return;
- callback arguments reuse the existing D188 foreign-value admission;
- callback results are projected only through the bounded lossless outbound
  projection admitted by D189, without automatic export of arbitrary
  identity-bearing Protos objects;
- a Protos Error/control outcome that returns recognizably unchanged through the
  same foreign operation continues as that exact Protos outcome;
- failures produced or transformed by the entered foreign operation remain fresh
  D188 `ForeignError` occurrences;
- synchronous nested/reentrant callback invocation is permitted inside the live
  originating foreign dynamic extent;
- the callback capability is live only during that originating dynamic extent;
- late, expired, or post-termination entry is rejected before guest execution;
- physical callback-wrapper retention does not create semantic lifetime or
  execution resurrection; and
- retained/asynchronous callbacks remain deliberately deferred.

`spec/concurrency/ACTORS.md` §24J now records that synchronous foreign
callbacks:

- execute in the originating Actor and same synchronous execution segment;
- create no Actor turn, message, or implicit interleaving boundary;
- do not permit concurrent Protos entry against the Actor-local execution;
- reject unrelated foreign-created-thread entry before guest execution; and
- do not make physical carrier/thread identity semantic Actor/Task authority.

`spec/concurrency/FUTURES_AND_TASKS.md` now records that synchronous foreign
callbacks:

- run in the current Task and same structured execution scope;
- create no Task, Future, structured scope, or callback-specific cancellation
  checkpoint;
- may continue through operations that complete without an actual suspension;
- reject an actual Task suspension across the arbitrary foreign dynamic extent
  before suspension commits;
- use an ordinary fresh `Error` occurrence for that forbidden-suspension
  failure rather than introducing a new public Error category; and
- preserve existing cooperative/cancellation-first semantics at already-defined
  cancellation boundaries.

The global specification changelog advances to `0.1.446` and explicitly
records that D188 foreign-value semantics remain unchanged and that I081 adds no
syntax, Error category, or runtime interoperability API.

## Acceptance result

```text
D189_SELECTED_CANDIDATE=A_DYNAMIC_SYNCHRONOUS_CALLBACK_BASELINE
D189_CONTRACT_COVERAGE=PASS
SPECIFICATION_RECONCILIATION=PASS

CALLBACK_OWNER_ACTOR_SPEC=PASS
CALLBACK_OWNER_TASK_SPEC=PASS
NO_NEW_TASK_SPEC=PASS
EXACT_CALLBACK_VALUE_SPEC=PASS
D188_ARGUMENT_ADMISSION_SPEC=PASS
CALLBACK_RESULT_PROJECTION_SPEC=PASS
CALLBACK_ERROR_CONTROL_SPEC=PASS
SYNCHRONOUS_REENTRANCY_SPEC=PASS
FOREIGN_THREAD_REJECTION_SPEC=PASS
SUSPENSION_BOUNDARY_SPEC=PASS
CANCELLATION_SPEC=PASS
DYNAMIC_LIFETIME_SPEC=PASS
LATE_CALLBACK_NO_RESURRECTION_SPEC=PASS
RETAINED_ASYNC_DEFERRED_SPEC=PASS

D188_PRESERVED=PASS
PLAT052_NOT_PRESELECTED=PASS
PLAT053_NOT_PRESELECTED=PASS

RUNTIME_IMPLEMENTATION_CHANGE=NO
STANDARD_LIBRARY_IMPLEMENTATION_CHANGE=NO
TEST_IMPLEMENTATION_CHANGE=NO
```

## Validation provenance

The maintainer reported after publication:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

This record therefore preserves the execution provenance as:

```text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
```

No test count or command list is inferred beyond the maintainer report.

The GitHub publication was independently re-read at the exact ending revision.
At closure verification time `guillermomolina/protos:main` points exactly to
`44688445543a2cef79b89c67b7145e4d382da6d0`, and the commit contains only the
four specification files listed above.

## Closure and follow-up

I081 has fulfilled its single implementation scope. There is no additional I081
slice.

D189 is now both durably ratified and normatively reconciled. Foreign
provider/runtime implementation remains outside I081.

The next AUD019 decision-chain work is PLAT052 / `guillermomolina/protos#821`,
which is a separate implementation-independent **platform-security
investigation**, not an I081 continuation. PLAT053 remains blocked on PLAT052.

```text
I081_STATUS=CLOSED_COMPLETED
I081_NEXT_SLICE=NONE
D189_NORMATIVE_RECONCILIATION=COMPLETE
PLAT052_STATUS=READY
PLAT052_TYPE=INVESTIGATION
PLAT053_STATUS=BLOCKED_BY_PLAT052
FOREIGN_RUNTIME_IMPLEMENTATION_AUTHORIZED_BY_I081=NO
```

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published I081 commit, the live normative specification owners,
the live GitHub work-item state, and the maintainer-reported local validation.
No independent human review is claimed by this record.
