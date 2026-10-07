# PLAT052 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — CANDIDATE B′ SELECTED; READY TO CLOSE AFTER PUBLICATION**

Formal decision: `guillermomolina/protos#821` — PLAT052

Parent audit: `guillermomolina/protos#818` — AUD019

Next decision: `guillermomolina/protos#822` — PLAT053

Approval date: **2026-10-07**

## Approval chain

PLAT052 completed its GITHUB010 investigation for foreign authority and sandbox
enforcement.

The investigation compared at minimum:

~~~text
A  — trusted/all-access foreign runtime default
B  — deny-by-default in-process authority mapping
B′ — capability-preserving deny-by-default with provider enforcement gate
C  — out-of-process isolation for every foreign runtime
~~~

The owner-review packet recommended:

~~~text
RECOMMENDED_CANDIDATE=
  B_PRIME_CAPABILITY_PRESERVING_DENY_BY_DEFAULT_WITH_PROVIDER_ENFORCEMENT_GATE
~~~

The project owner then explicitly answered:

> apruebo B'

This is exact-candidate approval of Candidate B′ immediately presented for
PLAT052.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=EXPLICIT
SELECTED_CANDIDATE=B_PRIME_CAPABILITY_PRESERVING_DENY_BY_DEFAULT_WITH_PROVIDER_ENFORCEMENT_GATE
~~~

## Selected boundary

The ratified boundary is:

~~~text
foreign execution starts with zero ambient authority

import/dependency/language availability do not grant host authority

explicit Protos capabilities may be mapped only through scope-preserving
non-amplifying enforcement

restricted in-process execution is admitted only when the relevant provider /
library / feature has a real non-amplification proof

otherwise:
  explicit trusted host/embedder policy
  OR stronger provider-specific isolation
  OR fail closed

trusted mode cannot be selected by guest code or import

Context-wide switches may not silently replace narrower possession-based
Protos authority

provider code-loading authority is implementation-internal and non-exportable

D188 foreign-value semantics unchanged
D189 callback semantics unchanged
PLAT053 owns physical provider/Context/process topology
~~~

The full selected contract is preserved in:

~~~text
docs/project/decisions/platform/PLAT052_FOREIGN_AUTHORITY_SANDBOX_ENFORCEMENT.md
~~~

## GITHUB021 invariant/delta consistency

Applicable fixed constraints from PLAT052/#821 and prerequisite authority were
rechecked before ratification:

~~~text
import itself is not an authority grant
Process does not imply filesystem/network/subprocess/native authority
dependency presence is not execution authority
foreign mutable state must not bypass Actor isolation
allowAllAccess(true) is not an acceptable default
~~~

Candidate B′ preserves all five.

The refinement from simple Candidate B to B′ adds an implementation enforcement
gate and conditional stronger isolation/fail-closed behavior. It does not narrow,
reverse or contradict any prior approved semantic/architectural invariant.

It also preserves:

- D188/#819 foreign-value/module semantics;
- D189/#820 same-Actor/same-Task synchronous callback contract;
- existing Filesystem/Network/Process capability boundaries;
- PLAT022 context source-readability authority as implementation precedent;
- Actor/P non-transferability and no shared mutable foreign-state rules;
- import/dependency separation from execution authority.

~~~text
PLAT052_FIXED_CONSTRAINT_DELTA=NONE
D188_DELTA=NONE
D189_DELTA=NONE
PROTOS_CAPABILITY_MODEL_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

No previously owner-approved PLAT052 invariant is reopened by B′.

## Revision revalidation

The investigation first froze one coherent baseline while the repository was
moving concurrently. During the investigation, D189 normative reconciliation
landed as specification revision 0.1.446 and unrelated Standard Library work
advanced `main`.

Immediately before ratification publication, live Protos was revalidated at:

~~~text
RATIFICATION_PROTOS_REVISION=b10569680b1532b277ba3ce49eced2e818833d93
RATIFICATION_PROTOS_VERSION=0.3.268-SNAPSHOT
RATIFICATION_SPECIFICATION_REVISION=0.1.446
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

The latest intervening product change is:

~~~text
b10569680b1532b277ba3ce49eced2e818833d93
  LIB020-B: add pure deterministic std:text/ANSI renderer
~~~

It changes `std:text` implementation/tests/bootstrap surfaces and does not alter
PLAT052 foreign authority, Context permission, provider, module, D188/D189,
Actor/P or Process capability contracts.

~~~text
RATIFICATION_DRIFT=MATERIAL_TO_PLAT052:NO
~~~

## Existing authority preserved

The selected architecture remains consistent with the current normative and
durable owners:

- D188 / specification revision 0.1.445+ for foreign values/modules;
- D189 / specification revision 0.1.446 for synchronous foreign callbacks;
- Process/Filesystem/I/O/Network capability authority;
- Actor/P isolation and foreign mutable-state boundaries;
- PLAT013 debugger/interop projection being non-authority;
- PLAT022 source-readability authority;
- PLAT028 callback/continuation ownership;
- AUD019-A/B generic foreign substrate + per-language provider conclusion.

## Project-record revision coupling

This publication is prepared from:

~~~text
PROJECT_RECORD_BASE_REVISION=0ce4eed7f0cb7a18a0724c7e1643a410e8727577
~~~

The exact final project-record revision containing this evidence, the PLAT052
decision record and the platform registry reference is recorded on the
authoritative GitHub Issue after publication.

## Decision closure and next routing

PLAT052 is a platform decision owner. Exact owner approval plus durable
ratification publication are sufficient to close PLAT052 as completed.

No normative specification reconciliation is required:

~~~text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
NEXT_REQUIRED_PLAT052_SLICE=NONE
~~~

PLAT052 therefore releases the next already-allocated AUD019 decision:

~~~text
NEXT_WORK_ITEM=PLAT053
NEXT_WORK_ITEM_ISSUE=#822
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=NO
~~~

PLAT053 must consume the ratified authority constraint:

~~~text
AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT
~~~

and decide the provider/Engine/Context/session/process topology. It remains a
separate owner-review decision and does not implement during investigation.

## Validation class

This is governance/documentation-only publication. No Protos source,
specification, implementation version, runtime code or product test is changed.

Changed documentation is checked for diff/whitespace hygiene before publication
and the exact published revision is re-read after publication.

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the completed PLAT052 investigation, live repository state and the project
owner's explicit approval interaction. No independent human review is claimed.
