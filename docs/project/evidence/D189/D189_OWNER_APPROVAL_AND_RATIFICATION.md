# D189 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — CANDIDATE A SELECTED; D189 READY TO CLOSE AFTER PUBLICATION**

Formal decision: `guillermomolina/protos#820` — D189

Parent audit: `guillermomolina/protos#818` — AUD019

Normative reconciliation owner: `guillermomolina/protos#829` — I081

Approval date: **2026-10-07**

## Approval chain

D189 completed the required GITHUB010 investigation for foreign callback
execution and lifetime semantics.

The investigation compared at minimum:

```text
A — dynamic synchronous callback baseline
B — new Actor-local Task per callback
C — retained/general asynchronous callback baseline
```

The completed owner-review packet recommended:

```text
RECOMMENDED_CANDIDATE=A_DYNAMIC_SYNCHRONOUS_CALLBACK_BASELINE
```

The project owner then explicitly answered:

> apruebo A

This is direct approval of the exact Candidate A recommendation immediately
presented for D189.

```text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=EXPLICIT
SELECTED_CANDIDATE=A_DYNAMIC_SYNCHRONOUS_CALLBACK_BASELINE
```

## Selected boundary

The approved contract preserves these core rules:

```text
same originating Actor
same current Task and structured execution scope
no new Task on callback entry
exact Protos callback semantic value
ordinary Closure/call/receiver/methodHome/return-home semantics
D188 argument admission/conversion
minimal lossless outbound result projection
unchanged Protos Error/control round-trip preserves exact semantic identity
foreign-produced/transformed failures remain D188 ForeignError
nested/sequential/recursive synchronous re-entry allowed
concurrent re-entry rejected
foreign-created/thread-pool callback entry rejected before guest execution
actual suspension across arbitrary foreign dynamic extent rejected before commit
no arbitrary Java/native/foreign stack capture
no new callback cancellation checkpoint
callback valid only during originating foreign operation dynamic extent
physical wrapper retention does not extend semantic lifetime/rooting
late/terminal/closed callbacks never resurrect Task/Actor/Process/provider/Context
retained/asynchronous callback ingress is a future explicit facility
```

The full selected contract and rationale are preserved in:

```text
docs/project/decisions/language/D189_FOREIGN_CALLBACK_EXECUTION_AND_LIFETIME_SEMANTICS.md
```

## Revision revalidation

The D189 prompt recorded this starting authority:

```text
PROMPT_REVALIDATED_PROTOS_REVISION=a89249ab41933c899017f6fc6e58b7206cfc5935
PROMPT_SPECIFICATION_REVISION=0.1.445
```

Before producing the owner recommendation and before this durable publication,
live `main` was re-read at:

```text
RATIFICATION_PROTOS_REVISION=84ff312ca6c29d2bfa9e6a4d69861c1dfdd07c47
RATIFICATION_PROTOS_VERSION=0.3.265-SNAPSHOT
RATIFICATION_SPECIFICATION_REVISION=0.1.445
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

The intervening product commits are:

```text
b3d85c1deee91455c2777d0025a19ec3970530ac
  PLAT051-B: opt std:regex/Regex Pattern and Match into semantic transfer

84ff312ca6c29d2bfa9e6a4d69861c1dfdd07c47
  TEST009-AI: keep missing Array binding failure out of PE
```

Neither changes the D189-governing normative foreign callback contract; the
specification remains `0.1.445`, and TEST009-AI explicitly records no
specification or semantic change.

```text
D188_INVARIANT_DELTA=NONE
D189_DECISION_DELTA=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Existing authority preserved

The selected result is consistent with:

- D188/#819 and normative specification revision `0.1.445`;
- ordinary Closure activation, lexical capture, receiver, `methodHome`,
  return-home and NLR semantics;
- Actor §24J synchronous foreign-call execution-segment semantics;
- task-scoped structured ownership and cooperative cancellation;
- PLAT014 C′ continuation ownership;
- PLAT019 B′ native suspension bridge boundary;
- PLAT028 Candidate C: C′ owns suspendible guest/post-callback control and
  arbitrary host/native frames are not Protos continuations.

No D188 semantic rule is reopened.

## Project-record revision coupling

This publication was prepared from live
`guillermomolina/protos-project-docs` default-branch base:

```text
PROJECT_RECORD_BASE_REVISION=dc9cb350547eaf9a306c5e082901877448176ecb
```

The final project-record revision containing this evidence and the D189 decision
record is recorded on the authoritative GitHub Issue after publication.

## Decision closure and implementation routing

D189 is a decision owner. Exact owner approval plus durable ratification
publication are sufficient to close D189 as completed.

Because the selected contract changes observable language semantics, normative
specification reconciliation is separate implementation work and has been
allocated as:

```text
I081 / #829
  -> normative D189 specification reconciliation
  -> implementation repository: guillermomolina/protos
```

I081 does not implement foreign providers/runtime interop. PLAT052, PLAT053 and
later implementation allocation remain authoritative for those later layers.

```text
DECISION_SELECTION=RATIFIED
DURABLE_DECISION_PUBLICATION=PASS_AFTER_COMMIT_VERIFICATION
D189_CLOSE_AFTER_PUBLICATION=YES
D189_STATUS=COMPLETED

SPECIFICATION_RECONCILIATION=ROUTED_TO_I081/#829
I081_IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY

FOREIGN_RUNTIME_IMPLEMENTATION_AUTHORIZED=NO
```

## Validation class

This is governance/documentation-only publication. No Protos source,
specification, implementation version, runtime code, or product test is changed
by this project-record commit.

Changed documentation is checked for whitespace/diff hygiene before publication;
the repository publication itself is then re-read from `main`.

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the completed D189 investigation, current live repository state, and the
project owner's explicit approval interaction. No independent human review is
claimed.
