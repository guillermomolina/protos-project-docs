# I083 — D192 specification-reconciliation allocation evidence

Status: **ALLOCATED — READY**

Formal implementation work: `guillermomolina/protos#834` — I083

Decision authority: `guillermomolina/protos#833` — D192

Parent implementation: `guillermomolina/protos#830` — I082

Allocation date: **2026-10-07**

## Allocation reason

D192 ratified Candidate A — exact receiver — after explicit project-owner
approval and durable publication.

The selected D192 contract defines observable Protos semantics and therefore
must be reconciled into the normative `guillermomolina/protos:spec/` authority.
That mutation is implementation work, not decision work.

The allocation follows the existing D188 -> I080 and D189 -> I081 pattern:

~~~text
Dxxx
  owns investigation + owner-approved decision + durable ratification

Ixxx
  owns normative specification reconciliation of the ratified decision
~~~

Therefore:

~~~text
FORMAL_IDENTIFIER=I083
ISSUE=guillermomolina/protos#834
TITLE=Reconcile D192 foreign each normal-result semantics into normative specification
FAMILY=I
STATUS=READY
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
DECISION_AUTHORITY=D192/#833
~~~

## Ratified semantic input

~~~text
SELECTED_CANDIDATE=A_EXACT_RECEIVER
FOREIGN_EACH_NORMAL_COMPLETION_RESULT=EXACT_ORIGINAL_PROTOS_FACING_RECEIVER

RAW_FOREIGN_RECEIVER_RESULT=SAME_RAW_PROTOS_FOREIGN_REFERENCE
FOREIGN_MODULE_FACADE_RESULT=SAME_ACTOR_LOCAL_PROTOS_FACADE
UNDERLYING_FOREIGN_TARGET_RESULT=NO
CALLBACK_RESULT_SELECTS_EACH_RESULT=NO
~~~

No other D188/D189/PLAT052/PLAT053 semantic or architectural delta is
authorized.

## Scope

I083 owns one bounded specification-only reconciliation.

Expected primary owner at allocation time:

~~~text
spec/semantics/VALUES_AND_COLLECTIONS.md
  Foreign Values
    Indexed access, foreign hash containers, and iteration
~~~

I083 must also advance the global specification revision and record the change in
`spec/PROTOS_SPEC_CHANGELOG.md`.

It must not change runtime Java, I082-D2 pull mechanics, concrete providers,
`std:interop`, authority, provider topology, Actor/P transfer, ForeignError,
iteration order, snapshot behavior, or callback semantics.

## Product baseline

The selected behavior is already implemented in I082-D2:

~~~text
I082_D2_REVISION=271a27662ee600212b7163d73253c2060875b0e8
I082_D2_VERSION=0.3.276-SNAPSHOT
~~~

At D192 ratification-time revalidation, current Protos had independently
advanced to:

~~~text
REVALIDATED_PROTOS_REVISION=c03370abca4592b95d35ccba5c4a185b955bd4bb
REVALIDATED_VERSION=0.3.277-SNAPSHOT
REVALIDATED_SPECIFICATION_REVISION=0.1.446
~~~

I083 must nevertheless execute from actual current HEAD, not from either
historical revision.

## Coordination result

After D192 durable publication:

~~~text
D192/#833=closed,status:completed
I083/#834=open,status:ready
I082/#830=open,status:blocked

I082_D2_SEMANTIC_CLOSURE=BLOCKED_ON_I083
I082_E_RELEASE=NO
~~~

After I083 publishes the normative reconciliation, I082-D2 semantic closure is
unblocked and I082-E may be released.

## Hierarchy note

I083 carries:

~~~text
Parent: #833
~~~

as durable textual cross-reference. The currently available connector does not
expose native Parent/Sub-issue mutation, so no false native-hierarchy claim is
made.

## D192 project-record authority

The exact D192 ratification publication is:

~~~text
D192_PROJECT_RECORD_REVISION=ac852ea6e91dd0cfe3f9ff7dd203972e967eefa7
~~~

with:

~~~text
docs/project/decisions/language/D192_FOREIGN_EACH_NORMAL_COMPLETION_RESULT.md
docs/project/evidence/D192/D192_OWNER_APPROVAL_AND_RATIFICATION.md
~~~

## AI-assistance disclosure

This allocation/evidence record was materially prepared with AI assistance from
ChatGPT using the live Protos governance files, the D188/I080 and D189/I081
precedents, the ratified D192 contract, and live Issues. No independent human
review is claimed.
