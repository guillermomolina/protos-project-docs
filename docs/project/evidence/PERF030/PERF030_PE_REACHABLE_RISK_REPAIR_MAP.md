# PERF030 — PE-reachable LocalRangeAccessor repair-family map

## Scope

This record preserves the PERF030-H investigation over the exact
`LocalRangeAccessor` PE-reachability authority published by PERF030-G.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while this compilerability class is unresolved.

This slice is investigation only. It changes no Protos product/runtime source,
tests, specification, benchmark workload, or PE checker.

## Exact authority

```text
SLICE=PERF030-H
WORK_TYPE=INVESTIGATION

PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
COMMIT_SUBJECT=PERF030-G: add PE reachability analysis to LocalRangeAccessor guard

BASELINE=tools/java_local_range_pe_reachability_baseline.json
ANALYZER=tools/java_local_range_pe_guard.py

TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_PROVEN_CONSTANT=4
PE_REACHABLE_RISK=24
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
```

The investigation verified that `guillermomolina/protos:main` was still at the
exact revision above, so no source drift existed relative to the checked-in
reachability baseline.

## External Truffle authority used

Current official GraalVM Truffle Java API documentation establishes:

1. `LocalRangeAccessor` offsets must be valid compilation-final indexes and its
   indexed get/set/clear/isCleared APIs document the offset as a partial
   evaluation constant.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/LocalRangeAccessor.html

2. Bytecode DSL `@ConstantOperand` values are supplied at parse time and have
   `CompilerDirectives.CompilationFinal` semantics. A dynamic operand supplied
   by a `LoadConstant` operation is not guaranteed to become compilation-final.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ConstantOperand.html

3. `ConstantOperand.dimensions` currently supports only `0`; array elements
   cannot be made compilation-final through that annotation. Therefore an
   `int[] ordinals` constant operand plus runtime indexing is not a durable
   solution for a `LocalRangeAccessor` offset.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ConstantOperand.html

4. `@ExplodeLoop` is compatible only with a partial-evaluation-constant
   iteration count, and it applies only to loops originating in the annotated
   method, not arbitrary loops later inlined from other methods.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/ExplodeLoop.html

5. `@TruffleBoundary` is a boundary for Truffle partial evaluation.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/CompilerDirectives.TruffleBoundary.html

6. `CompilerAsserts.partialEvaluationConstant(int)` asserts that a value is
   reduced to a constant during the initial partial-evaluation phase.
   - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/CompilerAsserts.html

## Result

All 24 current `PE_REACHABLE_RISK` sinks are accounted for exactly once as a
primary repair-family member:

```text
PERF030_H=COMPLETE

TOTAL_PE_REACHABLE_RISK=24
ACCOUNTED_FOR=24

CONFIRMED_VIOLATION=23
STRUCTURALLY_SAFE_DESPITE_STATIC_RISK=1
REQUIRES_RUNTIME_PROOF=0

REPAIR_FAMILIES=6
```

The 23 confirmed violations are source structures that can supply a genuinely
non-constant offset to a PE-reachable `LocalRangeAccessor` operation. The one
structurally safe sink is an unreachable side of a conservative static
classification; its current source nevertheless obscures that invariant and
should be simplified.

## Sink map

Source aliases used below:

```text
BRN  = src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
SBRN = src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
FLBA = src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
ICFB = src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
LOW  = src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
```

| # | Sink | Local provenance | Contract status | Primary family |
|---:|---|---|---|---|
| 1 | `BRN.createCurrentFrameBinding / isCleared(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 2 | `BRN.createCurrentFrameBinding / setObject(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 3 | `FLBA.ensureGeneralEstablishmentOrder / isCleared(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 4 | `FLBA.hasFrameBackedBindingAt / isCleared(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F2 |
| 5 | `FLBA.readFrameBackedBindingAt / getObject(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F2 |
| 6 | `FLBA.assignFrameBackedBindingAt / setObject(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F2 |
| 7 | `FLBA.createFrameBackedBindingAt / isCleared(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 8 | `FLBA.createFrameBackedBindingAt / setObject(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 9 | `FLBA.adoptPresentFrameBackedBindings / isCleared(ordinal)` occurrence 0 | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 10 | `FLBA.adoptPresentFrameBackedBindings / isCleared(ordinal)` occurrence 1 | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 11 | `FLBA.isEmpty / isCleared(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F5 |
| 12 | `FLBA.containsBinding / isCleared(offset)` | RUNTIME_NAME_DERIVED | CONFIRMED_VIOLATION | F3 |
| 13 | `FLBA.readBinding / isCleared(offset)` | RUNTIME_NAME_DERIVED | CONFIRMED_VIOLATION | F3 |
| 14 | `FLBA.readBinding / getObject(offset)` | RUNTIME_NAME_DERIVED | CONFIRMED_VIOLATION | F3 |
| 15 | `FLBA.appendBindingsTo / isCleared(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 16 | `FLBA.appendBindingsTo / getObject(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 17 | `FLBA.appendBindingsTo / isCleared(offset)` | RUNTIME_NAME_DERIVED | CONFIRMED_VIOLATION | F4 |
| 18 | `FLBA.appendBindingsTo / getObject(offset)` | RUNTIME_NAME_DERIVED | CONFIRMED_VIOLATION | F4 |
| 19 | `ICFB.create / isCleared(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 20 | `ICFB.create / setObject(ordinal)` | METHOD_PARAMETER | CONFIRMED_VIOLATION | F1 |
| 21 | `ICFB.admitsCapturedAccess / isCleared(offsetOf(name))` | RUNTIME_NAME_DERIVED | STRUCTURALLY_SAFE_DESPITE_STATIC_RISK | F6 |
| 22 | `ICFB.durableActivation / isCleared(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 23 | `ICFB.durableActivation / getObject(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |
| 24 | `ICFB.durableActivation / clear(ordinal)` | LOOP_INDEX | CONFIRMED_VIOLATION | F4 |

## Family F1 — lowerer-known establishment ordinal loses constancy

```text
FAMILY=F1_LOWERER_KNOWN_ESTABLISHMENT_ORDINAL_LOST
SINK_COUNT=6
SINKS=1,2,7,8,19,20

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=MEDIUM
IMPLEMENTATION_RISK=MEDIUM
ORDER=1
```

### Root cause

The lowerer knows a binding's layout ordinal at bytecode build time, but some
operations transport that ordinal as an ordinary dynamic operand or through an
array subsequently indexed at runtime.

Current correct precedents already exist:

- `SBRN.BindClosureFrameParameter.ordinal` is an `int` `@ConstantOperand`;
- `SBRN.CreateCurrentFrameLocal.ordinal` is an `int` `@ConstantOperand`;
- `CreateCurrentIndexedLocalSlot` already carries its indexed creation ordinal
  structurally.

The remaining family includes scalar parameter/rest/inline establishment paths
and frame-native multiple creation.

### Repair

- Encode every scalar lowerer-known establishment ordinal as a real
  `@ConstantOperand(type = int.class)`.
- Do not replace `int[] ordinals` with an array constant operand: Bytecode DSL
  array elements are not compilation-final.
- Statically expand frame-native multiple creation into scalar constant-index
  establishment operations while preserving:
  - complete source-prefix observation before the first creation;
  - source-order establishment; and
  - no rollback after a later creation error.
- Do not change OPEN/CLOSED/FROZEN, PRESENT/ABSENT, materialization, D179, error,
  or establishment-order semantics.

Likely files:

```text
SBRN
BRN
ICFB
LOW
```

Acceptance starts with the static PE guard. After the whole family is coherent,
focused lowering/semantic regressions and a selected
`CompilerAsserts.partialEvaluationConstant` seam may verify the constant
invariant before one external compiler acceptance.

## Family F2 — captured owner ordinal remains runtime data

```text
FAMILY=F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE
SINK_COUNT=3
SINKS=4,5,6

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=MEDIUM
IMPLEMENTATION_RISK=HIGH
ORDER=2
```

### Root cause

Captured binding identity and its frame ordinal are known by canonical analysis
and lowering, but ordinary captured frame operations currently receive that
ordinal as runtime stack data.

For writes, the selected ordinal is additionally retained in
`CapturedLexicalWriteTarget.frameOrdinal` across RHS evaluation. A runtime field
does not establish the PE-constant contract at the later indexed frame access.

### Repair

- Encode the statically known captured ordinal structurally for captured read
  and write-target resolution.
- Preserve the selected owner/destination across RHS evaluation.
- Supply the statically known ordinal again as a constant operand at the actual
  captured frame assignment seam instead of relying on the runtime target field
  to satisfy the PE invariant.
- Preserve all nearer-binding retargeting, D179 presence checks, pre-RHS
  destination selection, FROZEN checks, and invalid-mutation behavior.

Likely files:

```text
SBRN
BRN
ICFB
LOW
```

Focused captured read/write and RHS-mutation regressions are mandatory because
the semantic risk of destination retention is material even though the intended
observable behavior is unchanged.

## Family F3 — generic runtime name resolves to a frame-range index

```text
FAMILY=F3_GENERIC_NAME_TO_RANGE_INDEX
SINK_COUNT=3
SINKS=12,13,14

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=LARGE
IMPLEMENTATION_RISK=HIGH
ORDER=6
```

### Root cause

The generic String-keyed authority APIs:

```text
containsBinding(name)
readBinding(name)
```

perform:

```text
frameBackedLayout.offsetOf(name)
 -> LocalRangeAccessor(..., offset)
```

where `name` is genuinely runtime data.

This is the same constant-index mismatch already documented by HEAD in the
existing `@TruffleBoundary putBinding(String,Object)` path.

### Repair

Do not simply boundary every current caller.

First separate statically selected lexical execution from genuinely generic
String-keyed fallback:

```text
known lexical binding
 -> constant ordinal / indexed API
    or a representation-specific presence seam

genuinely generic runtime name
 -> String-keyed authority fallback
 -> PE boundary / slow path
```

F2 and F6 should land before this family so captured static execution no longer
depends on the generic API shape.

Likely files include:

```text
FLBA
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalFallback.java
BRN
SBRN
LOW
```

## Family F4 — PE-visible lifecycle range scans

```text
FAMILY=F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS
SINK_COUNT=10
SINKS=3,9,10,15,16,17,18,22,23,24

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=MEDIUM
IMPLEMENTATION_RISK=MEDIUM
ORDER=5
```

### Root cause

Whole-range transition, snapshot, and materialization operations iterate a
frame-backed range with ordinary loop variables inside PE-reachable methods.

The relevant operations are not one hot scalar lexical access. They are
lifecycle work:

- compact-to-general establishment-order materialization;
- adoption of already-present frame locals when installing authority;
- authority snapshot/migration;
- first materialization of an inline callback activation.

### Loop decision

No current family member should be repaired with `@ExplodeLoop`.

Even where the layout object itself is a constant operand and its length can
fold, Protos has no architectural small maximum on the number of bindings. Full
graph expansion therefore has unbounded/unknown source-size growth.

### Repair

Move the range scan itself into dedicated `@TruffleBoundary` slow helpers while
keeping cheap state guards outside the boundary where useful.

For `ICFB.durableActivation`, preserve the fast:

```text
child.frameBindingsTransferred()
```

check in PE and boundary only the first-transfer body.

Acceptance should primarily be that the ten primary family sinks become
`BOUNDARY_CUT` in the static reachability guard, together with focused
transition/order/materialization regressions.

## Family F5 — isEmpty rescans state already represented by authority metadata

```text
FAMILY=F5_EMPTY_QUERY_RESCANS_FRAME
SINK_COUNT=1
SINKS=11

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=SMALL
IMPLEMENTATION_RISK=LOW
ORDER=4
```

### Root cause

`FLBA.isEmpty()` scans every frame-backed local even though the authority
already maintains representation metadata whose purpose includes exact current
establishment state:

- compact representation: `lastCompactFrameOrdinal`;
- general representation: `establishmentOrder`;
- dynamic bindings: `dynamicOverflow`.

### Repair

Replace the range scan with an O(1) representation query, preserving the existing
metadata invariants and removal/recreation semantics.

This should not use either `@ExplodeLoop` or a boundary.

## Family F6 — inline captured-access range check is structurally unreachable

```text
FAMILY=F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD
SINK_COUNT=1
SINKS=21

CONTRACT_STATUS=STRUCTURALLY_SAFE_DESPITE_STATIC_RISK

SEMANTIC_CHANGE=NO
DESIGN_DECISION_REQUIRED=NO
IMPLEMENTATION_SIZE=SMALL
IMPLEMENTATION_RISK=LOW
ORDER=3
```

### Structural proof

`CanonicalBindingAnalyzer.resolve(name, scope)` stops at the nearest lexical
scope declaring `name`.

At depth zero:

- an already-established binding is `Resolved`;
- a declared-but-not-yet-established binding is `Candidate`.

`CapturedResolved` is produced only after walking outward to an already
established binding, and its `lexicalDepth` is strictly positive.

The lowerer emits the inline captured frame operations reaching
`ICFB.admitsCapturedAccess` only for `CapturedResolved`.

Therefore a name that belongs to the current inline callback's frame-backed
layout cannot simultaneously be the `CapturedResolved` name supplied to that
operation. While the callback remains unmaterialized, a non-layout dynamic
current binding also cannot exist without taking the activation/context path,
and `admitsCapturedAccess` immediately rejects a materialized activation.

Thus the branch:

```text
frameBackedLayout.offsetOf(name) != null
 -> frameBackedLocals.isCleared(..., ordinal)
```

is unreachable for every current emitted inline-captured operation.

This is an actual HEAD structural argument, not reliance on opportunistic Graal
constant folding, so the sink is:

```text
STRUCTURALLY_SAFE_DESPITE_STATIC_RISK
```

and does not require runtime compiler proof.

### Repair

Remove or restructure the redundant range check so the source directly expresses
the already-proven ownership invariant and the PE guard no longer needs to
conservatively classify the impossible side.

## Compiler assertion strategy

`CompilerAsserts.partialEvaluationConstant(int)` is useful only at a small
number of invariant-owning seams after the corresponding repair is implemented.

Candidate seams:

| Assertion point | Invariant | Covers |
|---|---|---|
| direct frame branch of `BRN.createCurrentFrameBinding` | establishment `ordinal` is PE constant | F1 sinks 1-2 |
| `FLBA.createFrameBackedBindingAt` | admitted indexed creation ordinal is PE constant | F1 sinks 7-8 |
| direct frame branch of `ICFB.create` | inline establishment ordinal is PE constant | F1 sinks 19-20 |
| captured read/resolve indexed seam | captured owner ordinal is PE constant | F2 sinks 4-5 |
| captured frame assignment seam after F2 repair | assignment ordinal is PE constant | F2 sink 6 |

No compiler assertion is useful for F3, F4, F5 or F6: F3 is deliberately
dynamic, F4 must leave PE, F5 removes the indexed scan, and F6 removes an
unreachable branch.

## Ordered repair map

| Order | Family | Sinks | Size | Risk | Dependency |
|---:|---|---:|---|---|---|
| 1 | F1 lowerer-known establishment ordinal loses constancy | 6 | MEDIUM | MEDIUM | none |
| 2 | F2 captured owner ordinal runtime pipeline | 3 | MEDIUM | HIGH | none |
| 3 | F6 inline captured redundant range guard | 1 | SMALL | LOW | none |
| 4 | F5 isEmpty rescans frame | 1 | SMALL | LOW | none |
| 5 | F4 PE-visible lifecycle range scans | 10 | MEDIUM | MEDIUM | F1 desirable first for caller clarity |
| 6 | F3 generic name to range index | 3 | LARGE | HIGH | F2/F6 should land first |

The family sink counts sum to 24 exactly.

## Verification strategy

For each implementation family, use the cheapest adequate sequence:

```text
1. make test-local-range-pe-guard
2. focused Java/Protos semantic and lowering regressions
3. selected CompilerAsserts.partialEvaluationConstant seam where the family
   owns a constant invariant
4. one focused unchanged external JFR/IGV compiler acceptance after a coherent
   family is complete
5. full make test only when repository impact policy requires it
```

Do not chase historical Graal `Pi` node identifiers. The static guard is the
first signal; external compiler diagnostics are acceptance evidence after a
coherent repair family, not after each source edit.

No command, build, test, benchmark, IGV/JFR run, or repository edit in
`guillermomolina/protos` was performed by PERF030-H.

## Next slice

The first bounded implementation family is F1:

```text
NEXT_SLICE=PERF030-I
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F1_LOWERER_KNOWN_ESTABLISHMENT_ORDINAL_LOST
SINKS_COVERED=6
```

PERF030 remains open. PERF024 remains blocked from physical cross-runtime graph
interpretation until the PERF030 compilerability class is repaired and accepted.

AI assistance: this durable investigation record was drafted with ChatGPT from
the exact PERF030-G product/tooling authority, current HEAD source, the checked-in
PE reachability baseline, live GitHub Issue state, and current official GraalVM
Truffle Java API documentation.
