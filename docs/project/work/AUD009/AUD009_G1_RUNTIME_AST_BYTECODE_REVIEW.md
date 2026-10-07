# AUD009-G1 — Surface AST, Canonical AST, Bytecode DSL and C-prime runtime architecture review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#657`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`ecf563ed01275929d5b85330e8e6259cc85d73d8`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#657`, issue comment
`5740139296`.

Owner approval provenance: `guillermomolina/protos#657`, issue comment
`5740144822`, 2026-09-19.

Derived implementation route:

```text
I056 / #658 — Remove obsolete Closure-plan backend discriminator
```

Native hierarchy reconciliation state:

```text
I056_TEXTUAL_PARENT=#657
I056_NATIVE_PARENT=#657
NATIVE_PARENT_VERIFICATION=PASS
AUD009_G1_HIERARCHY_RECONCILED=YES
```

The repository Issue-intake workflow reconciled the explicit `Parent: #657`
declaration into GitHub's native parent/sub-issue relation and the native parent
endpoint was reverified before G1 closure.

## Scope

AUD009-G is specifically the parent audit's runtime / AST / Bytecode partition:

- source AST versus semantic/canonical AST;
- duplicated executable semantics;
- parser/tooling/debugger dependencies;
- Bytecode DSL/C-prime production authority;
- maintenance drag and regression risk.

G1 does not reopen already-ratified Process/Actor/I/O/runtime authority merely
because those mechanisms execute on the same JVM.

## Final owner-approved classification

```text
SURFACE_AST=KEEP
CANONICAL_AST=KEEP
CANONICALIZER=KEEP

BYTECODE_DSL_EXECUTION_BACKEND=KEEP
CANONICAL_TO_BYTECODE_LOWERER=KEEP
C_PRIME_CONTINUATION_COMPOSITION=KEEP

CLOSURE_EXECUTION_PLAN_BOUNDARY=KEEP
BYTECODE_PLAN_IMPLEMENTATION=KEEP

PROCESS_HOSTED_PUBLIC_PARSE_EXECUTION=KEEP
UNHOSTED_FRESH_CONTEXT_CANONICAL_EXECUTION=KEEP

LEGACY_EXECUTION_REGRESSION_GUARD=KEEP

CLOSURE_PLAN_BACKEND_DISCRIMINATOR=REMOVE_NOW_RECONSIDER_LATER

SECOND_EXECUTABLE_BACKEND=ABSENT_RETAIN_ABSENCE
LEGACY_EXECUTABLE_AST=ABSENT_RETAIN_ABSENCE
```

No G1 mechanism is classified `REMOVE_PERMANENTLY`.

## Surface AST remains

The parser-owned `Surface*` representation retains source-facing distinctions
and exact spans required by current non-execution consumers:

- Canonicalizer;
- static analysis / LSP;
- document symbols;
- definition/references analysis;
- documentation extraction;
- parser/canonicalization tests.

It preserves source forms such as grouping, unary/binary syntax, index syntax,
array/map construction, calls/member shape and slot-creation spellings before
semantic normalization erases those distinctions.

Parsing directly to Canonical AST would either erase useful source structure too
early or force source-only syntax institutions into the backend-independent
semantic representation.

Classification: **KEEP**.

## Canonical AST remains

PLAT035 explicitly establishes Canonical AST as the last backend-independent
executable semantic representation.

It exposes the normalized vocabulary consumed by execution independently of
source syntax and Truffle implementation details.

Examples include:

```text
Lookup
Create / MultipleCreate
Assign / IndexedAssign
Call
Send / SuperSend
Member
Closure
Object / Compose
Identity / NotIdentity
DerivedInequality
Intrinsic
Return
Sequence / Spread
MapConstruction
```

Lowering Surface AST directly to Bytecode would move semantic normalization into
a platform-specific backend and make future backend replacement repeat or
reverse-engineer language desugaring rules.

Classification: **KEEP**.

## Canonicalizer remains

The Canonicalizer owns one backend-independent normalization boundary, including:

- grouping erasure;
- array construction lowering;
- index read/write lowering;
- binary/unary operator lowering;
- identity / non-identity;
- derived inequality;
- lazy boolean via Closure/send;
- call-versus-send normalization;
- slot create / multiple-create / assign;
- Closure/default/rest parameter representation;
- object composition;
- super-send normalization.

This centralization avoids duplicating semantic desugaring in execution and
source tooling.

Classification: **KEEP**.

## Bytecode DSL remains the sole executable backend

PLAT035 + AUD012 already completed the large simplification anticipated by
AUD009-G:

```text
source
  -> Surface AST
  -> Canonical AST
  -> Truffle Bytecode DSL
  -> execution
```

The former executable Truffle AST path is physically retired.

Current retired-backend names such as:

```text
CanonicalToTruffleLowerer
ProtosExpressionNode
ProtosRootNode
```

survive only in governance/regression-prevention material, not as a second
source-backed execution backend.

A permanent reference/oracle executable backend is rejected because it would
reintroduce duplicated semantic lowering, equivalence maintenance and optimizer
drag already eliminated by PLAT035.

Classification: **KEEP one executable backend / RETAIN ABSENCE of a second**.

## C-prime remains

C-prime continuation composition remains the selected cooperative suspension
architecture.

It keeps continuation state proportional to genuinely suspended/resumable work,
avoids replay of completed effects and releases physical carriers while logical
Tasks are suspended.

PERF006 established that the prior per-expression replay/interpreter-transfer
architecture caused pathological optimization behavior under the real Truffle
runtime. Returning to replay/interpreter-only execution would restore that known
defect rather than simplify the current architecture safely.

Classification: **KEEP**.

## Generic Closure execution-plan boundary remains

`ProtosClosureValue` and generic runtime code retain
`ProtosClosureExecutionPlan` rather than exposing
`ProtosBytecodeClosureExecutionPlan` directly.

That boundary remains useful even though there is currently one backend because
PLAT035 explicitly requires the Bytecode DSL to remain behind an implementation
boundary and not become semantic/frontend authority.

Keeping an implementation-neutral plan object does not authorize or imply a
permanent second backend.

Classification: **KEEP**.

## Migration-era backend discriminator is removed

The one approved G1 cleanup is:

```java
boolean isBytecodeBackendForRuntime() {
    return true;
}
```

plus the now-impossible negative branch and migration-only assertions whose sole
purpose is to test that boolean.

Under PLAT035:

- every executable source-backed Closure plan is Bytecode-backed;
- `ProtosClosureExecutionPlan` has no alternate backend implementation;
- the predicate is constant true;
- the product branch testing false is unreachable by construction.

The useful abstraction is the execution-plan boundary itself, not a pre-paid
backend-kind discriminator.

Approved classification:

```text
CLOSURE_PLAN_BACKEND_DISCRIMINATOR=REMOVE_NOW_RECONSIDER_LATER
```

If a future explicitly approved runtime/backend redesign introduces another plan
kind, that design can define the admission/compatibility mechanism it actually
needs.

Implementation is routed to **I056 / #658**.

## Hosted and deliberately unhosted execution remain

Production Process-backed module execution remains tied to the Process's exact
execution host.

Legitimate Java-owned semantic/runtime/bootstrap harnesses may use a fresh
`ProtosPolyglotExecutionContext`.

Those are different hosting/lifecycle boundaries, not different semantic
backends: executable guest source still converges on public parse and the same
Bytecode DSL backend.

AUD012/BUG006 already established and validated this split.

Classification: **KEEP**.

## Legacy-backend regression guard remains

The physical legacy backend is gone, but the fail-closed guard remains useful.

It prevents future compiler/runtime work from silently recreating dependencies
on retired executable-AST families while preserving explicit lower-level
compiler/frontend uses that remain legitimate.

Its continuing maintenance cost is small compared with the risk of a second
execution backend regrowing unnoticed.

Classification: **KEEP**.

## Strongest attempted simplifications

```text
delete Surface AST
    rejected -> loses/misplaces source-facing structure used by tooling

delete Canonical AST
    rejected -> makes Truffle backend own semantic desugaring

merge Surface + Canonical AST
    rejected -> conflates source syntax with normalized semantics

lower Surface directly to Bytecode
    rejected -> platform-specific semantic normalization

restore executable AST as oracle
    rejected -> permanent duplicate backend and PLAT035 contradiction

expose Bytecode plan directly to Closure runtime values
    rejected -> violates Bytecode implementation boundary

remove C-prime / restore replay
    rejected -> restores known optimizer pathology

forbid all unhosted Java execution
    rejected -> hosting is not backend identity

remove legacy-backend regression guard
    rejected -> cheap protection against backend regrowth

remove constant backend discriminator
    survives -> REMOVE_NOW_RECONSIDER_LATER
```

## Required implementation routing

Exactly one implementation route exists:

```text
I056 / #658
    remove ProtosClosureExecutionPlan.isBytecodeBackendForRuntime()
    remove the impossible non-Bytecode product branch
    reconcile migration-only tests/assertions
    preserve ProtosClosureExecutionPlan abstraction
    preserve Bytecode/C-prime architecture
```

No specification or public semantic change is authorized.

## AUD009-G partition state

The technical classification scope defined by AUD009 #522 is complete:

```text
SURFACE_AST_REVIEW=COMPLETE
CANONICAL_AST_REVIEW=COMPLETE
EXECUTABLE_BACKEND_DUPLICATION_REVIEW=COMPLETE
BYTECODE_DSL_REVIEW=COMPLETE
C_PRIME_REVIEW=COMPLETE
HOSTED_UNHOSTED_BACKEND_CONVERGENCE_REVIEW=COMPLETE
LEGACY_BACKEND_REGRESSION_REVIEW=COMPLETE

AUD009_G_CLASSIFICATION=COMPLETE
```

The #657 -> #658 native sub-issue relation is reconciled and verified.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=ecf563ed01275929d5b85330e8e6259cc85d73d8

SURFACE_AST=KEEP
CANONICAL_AST=KEEP
BYTECODE_DSL_SINGLE_BACKEND=KEEP
C_PRIME=KEEP
CLOSURE_EXECUTION_PLAN_BOUNDARY=KEEP
LEGACY_BACKEND_ABSENCE=KEEP

CLOSURE_PLAN_BACKEND_DISCRIMINATOR=REMOVE_NOW_RECONSIDER_LATER
DERIVED_IMPLEMENTATION=I056/#658

I056_NATIVE_PARENT=#657
HIERARCHY_RECONCILED=YES
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO

AUD009_G1_CLASSIFICATION=COMPLETE
AUD009_G_PARTITION_CLASSIFICATION=COMPLETE
AUD009_G1_COORDINATION_CLOSURE=COMPLETE
```
