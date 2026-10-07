# D188 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — NORMATIVE SPECIFICATION RECONCILIATION PENDING**

Formal decision: `guillermomolina/protos#819` — D188

Parent audit: `guillermomolina/protos#818` — AUD019

Approval date: **2026-10-07**

## Approval chain

D188 completed the required GITHUB010 investigation for foreign values and
foreign-module guest semantics.

The investigation compared, at minimum:

```text
A — direct InteropLibrary mapping
B — explicit std:interop only
C — hybrid Protos semantic projection
D — universal Protos facade over every foreign value
```

The completed owner-review packet recommended:

```text
RECOMMENDED_CANDIDATE=C_HYBRID_PROTOS_SEMANTIC_PROJECTION
```

The project owner then explicitly answered:

> apruebo recomendación

This is direct approval of the recommendation immediately presented for D188.

```text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=EXPLICIT
SELECTED_CANDIDATE=C
```

## Selected boundary

The approved choice preserves these core rules:

```text
Protos ModuleKey and Actor-local module identity remain authoritative
Protos === remains authoritative
Protos == and hash remain Protos protocols
ordinary invocation remains call-based
indexed access remains at/atPut-based
ordinary = remains local Protos slot mutation
foreign failures enter the Protos Error model
foreign values receive no automatic Actor/P transfer contract
std:interop and import share one foreign-value substrate
```

The selected hybrid permits ordinary syntax only for a faithful Protos-facing
projection and uses `std:interop` or provider facades for ambiguous,
side-effecting, provider-specific, metadata-oriented, or otherwise non-faithful
foreign operations.

The approved contract is preserved in the durable language-decision record:

```text
docs/project/decisions/language/D188_FOREIGN_VALUES_AND_FOREIGN_MODULE_GUEST_SEMANTICS.md
```

## Revision revalidation

The owner-review packet was produced at:

```text
PACKET_PROTOS_REVISION=915fa3739e3a0a6e7f0934e3975e79d3bb24eba8
PACKET_PROTOS_VERSION=0.3.260-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

Before durable ratification publication, Protos advanced by exactly one product
commit to:

```text
RATIFICATION_PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
RATIFICATION_PROTOS_VERSION=0.3.261-SNAPSHOT
```

That commit is `TEST009-AD: keep unsupported lookup failure out of PE`.

Its own changelog and diff state that it makes no specification or semantic
change. It adds `CompilerDirectives.transferToInterpreter()` before an existing
`UnsupportedOperationException` in `ProtosValueLookup.delegationParent(...)`;
the exception class, message, construction site, and failure timing are
unchanged.

No D188 authority file changed between the packet baseline and ratification
revision.

```text
AUD019_INVARIANT_DELTA=NONE_RELEVANT
D188_DECISION_DELTA=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Project-record revision coupling

This publication was prepared from the live
`guillermomolina/protos-project-docs` default-branch base:

```text
PROJECT_RECORD_BASE_REVISION=4340af88d9cf9eb83aa6e675d4276f34aef5ac93
```

The final project-record revision containing this evidence and the D188 decision
record is recorded in the authoritative GitHub Issue after publication.

## Normative specification boundary

The approval is sufficient to select the D188 semantic decision, but the project
policy makes observable Protos semantics normative only through the applicable
`guillermomolina/protos:spec/` authority.

The approved packet itself concluded:

```text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=YES
SPECIFICATION_CHANGE_REQUIRED_IF_APPROVED=YES
```

Therefore durable project-record publication does not by itself complete D188.

The next D188 slice is a bounded normative specification reconciliation in
`guillermomolina/protos`. Product/runtime implementation remains unauthorized
until the later AUD019 dependency chain is satisfied.

```text
DECISION_SELECTION=RATIFIED
DURABLE_DECISION_PUBLICATION=PASS
SPECIFICATION_RECONCILIATION=PENDING
D188_CLOSE_NOW=NO
D189_UNBLOCK_NOW=NO
FOREIGN_RUNTIME_IMPLEMENTATION_AUTHORIZED=NO
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the completed D188 investigation, current live repository state, and the
project owner's explicit approval interaction. No independent human review is
claimed.
