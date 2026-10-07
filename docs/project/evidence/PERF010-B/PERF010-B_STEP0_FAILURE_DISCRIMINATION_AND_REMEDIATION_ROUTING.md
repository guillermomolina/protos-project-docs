# PERF010-B — Step-0 failure discrimination and remediation routing

Date: 2026-09-27

## Scope

This record consumes the retained Step-0 compiler confirmation at:

`docs/project/evidence/PERF010-B/PERF010-B_STEP0_COMPILER_CONFIRMATION_GATE.md`

and closes the bounded follow-up investigation requested by that record:

1. discriminate the captured-local PE-constant failure;
2. discriminate the `Too deep inlining` failures;
3. compare the relevant implementation shape with the pinned Truffle Bytecode DSL mechanisms and materially comparable Truffle/Pkl precedents already surveyed by PLAT036/PERF010-A;
4. determine whether PERF010-B requires another semantic/platform decision before implementation.

No Protos product code is changed by this record.

## Exact product baseline

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
PROTOS_VERSION=0.3.96-SNAPSHOT
PERF010_B_ISSUE=guillermomolina/protos#722
```

Pinned runtime line:

```text
GRAALVM_TRUFFLE=25.3.4.1
ORACLE_GRAAL_BRANCH=release/graal-vm/25.3
ORACLE_GRAAL_BRANCH_REVISION_AT_INVESTIGATION=7b025988a922a73286d1326e1eddc1ca39d3f569
```

The investigation used the release/25.3 Bytecode DSL API and tests rather than
a later development branch as architectural authority.

## Retained Step-0 failures

The prior record established:

```text
ROOT_ID=146
ROOT_CLASS=ProtosBytecodeRootNodeGen
SOURCE=closure-call.protos:20
OPT_FAILED=1
FAILURE=Partial evaluation did not reduce value to a constant

ROOT_ID=148
ROOT_CLASS=ProtosBytecodeRootNodeGen
OPT_FAILED=1
FAILURE=PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.

ROOT_ID=150
ROOT_CLASS=ProtosBytecodeRootNodeGen
OPT_FAILED=1
FAILURE=PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.
```

The follow-up investigation also identified the equivalent captured-local
PE-constant failure on the corresponding root represented as root 152 in the
retained compiler evidence.

The paired semantic roots compile; the failure is therefore asymmetric and
localized to required helper Bytecode roots, not a general inability of the
shared `repeat` / `ifTrue` driver to compile.

## Finding A — frame lexical authority construction causes the inlining-depth bailout

Current `ProtosFrameLexicalBindingAuthority` construction rebuilds immutable
layout metadata for every authority instance.

The relevant shape is:

```text
known frame-backed names/ordinals
  -> ProtosFrameLexicalBindingAuthority.<init>
  -> ArrayList / LinkedHashMap construction
  -> LinkedHashMap.put
  -> HashMap tree/comparable machinery
  -> Class generic-interface / generic-signature machinery
  -> deep host/JDK inlining expansion
```

The observed `Too deep inlining` is therefore not established as guest
`repeat` recursion. It is a compiler expansion caused by avoidable
general-purpose host collection construction on the PE-visible authority
installation path.

The lexical layout is already determined by lowering/root construction. Its
name/ordinal relation is backend metadata, not per-invocation semantic state.

### Required remediation shape

Move immutable lexical layout construction out of the per-invocation path:

```text
precomputed backend-private lexical layout
+ constant LocalRangeAccessor
+ current BytecodeNode
+ current frame
-> lightweight ProtosFrameLexicalBindingAuthority
```

The descriptor must not become a second value/presence authority.

This bounded remediation is allocated as:

```text
PERF012=guillermomolina/protos#723
TITLE=Precompute frame lexical layout for PE-safe authority installation
REPOSITORY=guillermomolina/protos
```

Acceptance is structural first: the source-equivalent roots for Step-0 roots
148/150 must no longer fail through the identified authority-constructor /
`LinkedHashMap` expansion.

## Finding B — captured-local PE failure is a representation mismatch

The captured binding identity itself is statically proven by the canonical
binding analysis. Current lowering nevertheless reaches the physical owner
accessor/node through runtime authority state:

```text
CapturedResolved
  -> runtime captured execution context
  -> ProtosFrameLexicalBindingAuthority
  -> owner LocalRangeAccessor
  -> owner BytecodeNode
  -> LocalRangeAccessor.isCleared(...)
```

`LocalRangeAccessor.isCleared` requires the accessor/node identity needed for
the access to be PE-constant. Because Protos rediscovers those values through
runtime authority objects in the child root, the captured path fails at:

```text
CompilerAsserts.partialEvaluationConstant
  -> LocalRangeAccessor.isCleared
  -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
  -> ReadCapturedFrameLocal.perform
```

The frame itself is not required to be constant. The dynamic value is the
owner frame; the local identity/declaration topology should be static.

## Pinned Truffle mechanism

Truffle 25.3 provides `MaterializedLocalAccessor` specifically for this
situation.

The release-line implementation records:

```text
MaterializedLocalAccessor
  = rootIndex
  + localOffset
  + localIndex
```

and its operations require the accessor and current `BytecodeNode` to be
partial-evaluation constants while accepting a dynamic `MaterializedFrame`.

For example the official API shape is:

```text
accessor.isCleared(currentBytecodeNode, materializedFrame)
accessor.getObject(currentBytecodeNode, materializedFrame)
accessor.setObject(currentBytecodeNode, materializedFrame, value)
```

The declaring node is resolved through:

```text
currentBytecodeNode
  -> getBytecodeRootNode()
  -> getRootNodes()
  -> getNode(rootIndex)
  -> getBytecodeNode()
```

This matters because Bytecode DSL reparsing/configuration updates may replace a
physical `BytecodeNode`. A durable captured-local optimization must therefore
retain logical local/root identity rather than permanently caching one physical
owner `BytecodeNode`.

### Rejected shortcut

The investigation considered a smaller specialization that cached the runtime
owner `LocalRangeAccessor` and owner `BytecodeNode`.

That may make the immediate PE assertion constant, but it is not selected as the
durable fix because the Bytecode DSL explicitly supports node replacement on
`BytecodeRootNodes.update(...)` / reparse. The official
`MaterializedLocalAccessor` mechanism avoids that stale-node coupling.

## Shared BytecodeRootNodes consequence

A `MaterializedLocalAccessor` may address locals from the current root or an
enclosing root in the same generated `BytecodeRootNodes` group.

Current Protos creates source/Closure helper roots independently with
`ProtosBytecodeRootNodeGen.create(...)`.

The selected implementation consequence is therefore to establish the smallest
shared helper-root grouping needed for captured lexical identity while retaining:

- `ProtosSemanticBytecodeRootNode` as the semantic/tagged wrapper boundary;
- helper `ProtosBytecodeRootNode` roots as backend-private/untagged;
- PLAT026 / PLAT034 tooling semantics;
- semantic Closure identity separate from executable projection;
- source/instrumentation behavior.

The existing `ProtosClosureExecutionPlan` path already supports construction
from an existing Bytecode body root, so the investigation found no need for a
mutable placeholder solely to construct Closure plans.

## Context-local rematerialization consequence

Current Context-local projection can rebuild a Closure execution plan
independently.

If captured access depends on a logical root index in a shared
`BytecodeRootNodes` group, the backend rematerialization unit must rebuild the
required lexical root group coherently rather than reconstructing a captured
child in isolation.

PLAT035 does not require one independently generated Bytecode root per semantic
Closure. It requires Bytecode DSL as the sole executable backend and preserves
the implementation boundary.

Current Protos requirements also do not define live lexical-state migration
between independent polyglot Contexts as a language semantic requirement.

This is therefore backend implementation machinery, not a new semantic or
platform choice.

The bounded remediation is allocated as:

```text
PERF013=guillermomolina/protos#724
TITLE=Lower captured lexicals through MaterializedLocalAccessor
REPOSITORY=guillermomolina/protos
```

## Current D179/I071 semantics

The investigation initially revisited the historical C3 assumption and corrected
it against current authority.

Current ratified/implemented state is:

```text
D179_SELECTED_CANDIDATE=C0_KEEP_CURRENT_EXECUTION_CONTEXT_SEMANTICS
I071_IMPLEMENTATION_CANDIDATE=E_FRAME_NATIVE_CLEARED_PRESENCE

ABSENT -> PRESENT = allowed while OPEN
PRESENT -> ABSENT = allowed while OPEN
PRESENT -> PRESENT = allowed while writable
```

Static local identity/layout remains stable while frame-native cleared/not-cleared
state represents semantic presence.

Therefore a captured static owner may not be loaded unconditionally.

Required captured read semantics remain:

```text
check current/nearer candidates according to exact lexical topology/presence

owner:
  MaterializedLocalAccessor.isCleared(...)
    PRESENT -> getObject(...)
    ABSENT  -> exact farther lexical / receiver fallback
```

Captured writes must still pin the destination before RHS evaluation and write
that previously selected destination afterward.

`MaterializedLocalAccessor.isCleared` is directly compatible with this C0/E
presence model.

## Decision classification

No new decision gate was found.

PLAT036 already selected frame-backed Candidate D and states that proven captured
lexical reads/writes should use Bytecode DSL materialized-local/frame mechanisms.

Its implementation contract classified captured/materialized lexical lowering as
implementation work and explicitly left the exact Java carrier
(`MaterializedFrame`, accessors, adapter state) to bounded implementation
slices.

D179 C0 / I071 changes presence/removal behavior but explicitly preserves the
PLAT036 frame-backed single-authority architecture and does not require reopening
I068.

PLAT026/PLAT034 remain preserved because helper-root grouping does not require
promoting helper roots to semantic/tagged roots.

Therefore:

```text
NEW_SEMANTIC_DECISION_REQUIRED=NO
NEW_PLAT_DECISION_REQUIRED=NO
PLAT036_REOPEN_REQUIRED=NO
I068_REOPEN_REQUIRED=NO
PLAT026_REOPEN_REQUIRED=NO
PLAT034_REOPEN_REQUIRED=NO
```

## Issue/slice routing

The two remediations have independent closure and can fail/block independently.
The captured-local remediation is additionally expected to span multiple
publication slices due to shared root grouping and Context-local
rematerialization.

Under `AGENTS.work/COORDINATION.md` they therefore cross the Issue-promotion
boundary instead of remaining hidden implementation slices in #722.

```text
PERF012=#723
PERF013=#724
PARENT_REQUIRED=#722
```

The active connector used for this coordination does not expose GitHub native
Parent/Sub-issue mutation. The Issue bodies contain `Parent: #722` only as
GITHUB006 bootstrap input. Native parent reconciliation remains pending and is
not claimed as PASS.

No direct native blocker edge is asserted between PERF012 and PERF013. PERF012
is the recommended execution order because it is smaller and removes a separate
compiler bailout before the next combined PERF010-B compiler/timing gate.

## Next work

```text
NEXT_WORK_ITEM=PERF012/#723
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
```

PERF012 should be implemented first.

After publication of PERF012, rerun the exact Step-0 compiler source/lifecycle
gate before interpreting timing.

PERF013 remains independently tracked and should then remove the captured-local
PE-constant failure.

After both remediations, PERF010-B must rerun its compiler/timing gate before
starting Step 2.

Target discriminator after both are complete:

```text
FRAME_AUTHORITY_TOO_DEEP_INLINING=ABSENT
CAPTURED_LOCAL_PE_CONSTANT_FAILURE=ABSENT
```

Only if the required roots compile and the guest-call increment remains
material should Step 2 proceed as the next causal optimization.
