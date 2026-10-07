# D188 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — D188 CLOSED; I080 ALLOCATED FOR SPECIFICATION RECONCILIATION**

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

## Decision closure and implementation routing correction

The approval is sufficient to select and close the D188 design decision once its
durable ratification is published. Observable Protos semantics still become
normative only through the applicable `guillermomolina/protos:spec/` authority,
but mutating that specification is implementation/reconciliation work and must
not be kept inside the Dxxx lifecycle.

A prior coordination note in this evidence incorrectly coupled D188 closure and
D189 release to specification reconciliation. The project owner challenged that
classification, and live project precedent confirms the correction: D180 closed
after ratification and its specification/implementation reconciliation was
allocated separately as I078.

The corrected routing is:

```text
D188 / #819
  -> decision owner
  -> owner-approved + durably ratified
  -> CLOSED / status:completed

I080 / #828
  -> implementation owner
  -> normative D188 specification reconciliation
  -> status:ready

D189 / #820
  -> next implementation-independent semantic decision
  -> consumes ratified D188
  -> status:ready
```

I080 does not implement foreign runtime/providers. Foreign runtime/provider
implementation remains gated by D189, PLAT052, PLAT053, and later implementation
allocation.

```text
DECISION_SELECTION=RATIFIED
DURABLE_DECISION_PUBLICATION=PASS
D188_CLOSE_NOW=YES
D188_STATUS=COMPLETED

SPECIFICATION_RECONCILIATION=ROUTED_TO_I080/#828
I080_IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY

D189_UNBLOCK_NOW=YES
D189_STATUS=READY

FOREIGN_RUNTIME_IMPLEMENTATION_AUTHORIZED=NO
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the completed D188 investigation, current live repository state, and the
project owner's explicit approval interaction. No independent human review is
claimed.
