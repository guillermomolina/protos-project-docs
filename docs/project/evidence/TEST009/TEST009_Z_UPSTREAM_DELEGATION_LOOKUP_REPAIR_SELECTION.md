# TEST009-Z — upstream-guided delegation lookup repair selection

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-Z
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
EXAMINED_PROTOS_REVISION=5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407
EXAMINED_PROTOS_VERSION=0.3.255-SNAPSHOT
PRODUCT_CHANGES=NONE
COMMAND_EXECUTION=NONE
```

TEST009-Z selects the next structural causal repair after TEST009-Y. This record
corrects the selection method used in the initial local-only analysis: mature
Truffle language implementations are the architectural comparison authority,
and the Protos residual is interpreted only after that comparison.

The governing method remains the TEST009 systematic compilerability procedure:
real compilation evidence identifies the failing expansion, but the repair is
chosen by classifying the architecture rather than by mechanically deleting the
largest method, adding boundaries, or applying unrelated micro-patches.

## Post-Y causal baseline

TEST009-Y removed exactly the `ProtosPrelude.errorPrototype()` defensive
exception expansion selected by TEST009-X, reducing the target root's
`Throwable.fillInStackTrace()` self size from 17303 to 11440 while the root
continued to fail with `CodeTooLarge`.

The post-Y residual ranking was:

```text
ProtosValueLookup.lookup -> delegationParent                     1573 / 1573
ProtosPrelude.arrayPrototype()                                    1430
ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState     1430
ProtosBytecodeRootNode.rejectComposedInvocationProjection         1430
PreparedBooleanCall.hasCallback()                                 1287
ProtosPrelude.standardErrorPrototype(String)                      1001
ProtosBytecodeRootNode.finishPreparingComposedCall                 858
```

The two `1573` ValueLookup observations are evidence of reach, not authority to
assume an exact `3146` removable delta. `lookup()` contains more than one
checked-then-consumed `Optional`, and `delegationParent()` also owns a real
unsupported-representation failure. TEST009-Z therefore does not add those
numbers mechanically.

## Upstream comparison

### GraalJS

Inspected revision:

```text
repository=oracle/graaljs
revision=95a0db94efeb033e841648ecd68e25cf95a2f065
```

Relevant implementation:

```text
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/GetPrototypeNode.java
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/PropertyGetNode.java
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/PropertyCacheNode.java
```

`GetPrototypeNode` returns a direct `JSDynamicObject`, specializes cached shapes
by reading the prototype `Location` directly, falls back to direct prototype
lookup, and represents the non-object/no-prototype case with the runtime
`Null.instance` identity. It does not carry the normal prototype traversal
through `Optional` plus unwrap machinery.

`PropertyGetNode` similarly exposes direct property values and uses
`Undefined.instance` as the normal absence/default representation. Prototype
chain cache nodes retain direct shapes and `GetPrototypeNode` children and use
shape assumptions/guards to protect the specialization.

Classification:

```text
DIRECT_LOOKUP_REPRESENTATION=YES
NORMAL_ABSENCE_SENTINEL=YES
OPTIONAL_WRAPPER_IN_INSPECTED_HOT_LOOKUP=NO
GUARD_ASSUMPTION_SPECIALIZATION=YES
SEMANTIC_FAILURE_SEPARATED_FROM_NORMAL_TRAVERSAL=YES
```

### GraalPy

Inspected revision:

```text
repository=oracle/graalpython
revision=154205b7511c9a51772ab510821b51aaf9216366
```

Relevant implementation:

```text
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/attributes/LookupAttributeInMRONode.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/type/MroShape.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/PyObjectGetAttrO.java
```

MRO lookup returns `Object` directly and represents normal absence with
`PNone.NO_VALUE`. The optimized MRO-shape lookup retains a direct index and uses
`NOT_FOUND_INDEX = -1` for absence; a found index directly selects the MRO item.

A genuine specialization race (`MROChangedException`) is stackless and handled
by transfer-to-interpreter/invalidation. Ordinary missing attributes are not
transported as host exceptions through the lookup. A higher-level operation that
requires the attribute converts `PNone.NO_VALUE` into `AttributeError`.

Classification:

```text
DIRECT_LOOKUP_REPRESENTATION=YES
NORMAL_ABSENCE_SENTINEL=YES
OPTIONAL_WRAPPER_IN_INSPECTED_HOT_LOOKUP=NO
GUARD_ASSUMPTION_SPECIALIZATION=YES
SEMANTIC_FAILURE_SEPARATED_FROM_NORMAL_TRAVERSAL=YES
```

### TruffleRuby

Inspected revision:

```text
repository=truffleruby/truffleruby
revision=c734f26543003fefd4519a29adbca62d0c711a2d
```

Relevant implementation:

```text
src/main/java/org/truffleruby/language/methods/LookupMethodNode.java
src/main/java/org/truffleruby/core/module/ModuleOperations.java
src/main/java/org/truffleruby/core/module/MethodLookupResult.java
```

The generic ancestor/method traversal returns an `InternalMethod` directly or
`null` when no method is found. Cached lookup retains the direct method plus its
`Assumption[]`; the valid-hit specialization returns that method directly.
Visibility and required-method failures are handled by the consuming operation,
not by wrapping every ordinary traversal step in an optional carrier.

Classification:

```text
DIRECT_LOOKUP_REPRESENTATION=YES
NORMAL_ABSENCE_SENTINEL=NULL
OPTIONAL_WRAPPER_IN_INSPECTED_HOT_LOOKUP=NO
GUARD_ASSUMPTION_SPECIALIZATION=YES
SEMANTIC_FAILURE_SEPARATED_FROM_NORMAL_TRAVERSAL=YES
```

### SimpleLanguage reference

Inspected revision:

```text
repository=graalvm/simplelanguage
revision=5a2b35790539ab1fe5c61f02b71e3485fd0e0fb1
```

`SLReadPropertyNode` uses `DynamicObjectLibrary.getOrDefault(..., null)` and
turns `null` into the language-level undefined-property error only after the
lookup reports absence. This is a minimal reference rather than the main design
authority, but it follows the same separation.

## Upstream architectural rule

The mature implementations converge on a more precise rule than merely
"avoid Optional":

> A PE-critical lookup path uses the most direct structural representation that
> fits the language — direct reference, sentinel, or compact index — while
> guards, assumptions, shapes or caches protect specialization. Ordinary absence
> is represented as data. A semantic error is produced at the layer that knows
> that absence or an invalid state is illegal.

Therefore the repair criterion for Protos is architectural convergence of the
lookup representation, not source-level elimination of one `orElseThrow()`.

## Current Protos comparison

At the examined revision, `ProtosValueLookup.lookup()` uses:

```text
ordinary.readLocalSlot(name)
  -> Optional<Object>
  -> isPresent()
  -> orElseThrow()

...

delegationParent(current, prelude)
  -> Optional<Object>
  -> isEmpty()
  -> orElseThrow()
```

The public `delegationParent(...)` surface also wraps an ordinary object's
already-direct parent representation in `Optional`.

However, Protos already has the upstream-style representation in its guarded
lookup architecture:

```text
ProtosObjectValue.directParentForGuardedLookup() -> Object
SharedInheritedLookup.exactParent -> Object
```

The shared inherited valid-hit guard compares the exact direct parent identity
without traversing an `Optional`. The generic authoritative lookup therefore
uses a heavier internal representation than the already-established guarded
path for the same semantic parent relation.

This is the structural divergence selected by Z.

## Candidate comparison

### A. ProtosValueLookup internal delegation traversal — selected

```text
OWNER=ProtosValueLookup
POST_Y_REACH=1573 delegationParent owner + 1573 Optional/orElseThrow owner, without assuming exact additive removability
UPSTREAM_MATCH=STRONG
PROTOS_INTERNAL_PRECEDENT=directParentForGuardedLookup + SharedInheritedLookup.exactParent
FAILURE_PATH_REALLY_REACHABLE=PARTIAL
REAL_FAILURE_TO_PRESERVE=unsupported runtime representation and null represented delegation parent
STRUCTURAL_REPAIR=Direct internal delegation-parent representation for lookup traversal
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
SEMANTIC_CHANGE=NO
MICRO_PATCH_RISK=LOW
```

The repair should make the generic authoritative traversal use the same direct
parent concept already used by guarded lookup. For ordinary objects, `null` may
represent only the canonical root object's absence of a parent. For represented
values, a null represented parent remains invalid and must retain the existing
failure. Unsupported runtime representations must continue to fail exactly as
unsupported; they must not become ordinary lookup misses.

The public convenience API:

```text
public Optional<Object> delegationParent(...)
```

is not selected for removal. It may remain as a wrapper around the authoritative
semantic parent relation for external consumers that want an Optional surface.

### B. ProtosPrelude.arrayPrototype() — not selected yet

GraalJS provides strong upstream precedent for retaining canonical prototypes in
realm state, including direct `realm.getArrayPrototype()` use. That makes the
long-term architectural direction plausible.

Protos nevertheless permits `ProtosPrelude` construction outside the canonical
Core bootstrap contract and currently validates Array lazily. Retaining a typed,
validated Array identity at every Prelude construction would move validation
failure timing. A raw-retention design could preserve timing but has not yet been
shown to remove the measured residual as directly as the ValueLookup repair.

```text
UPSTREAM_MATCH=STRONG_DIRECTIONALLY
CURRENT_PROTOS_CONTRACT_MATCH=INCOMPLETE
SELECT_NOW=NO
```

### C. attachTaskOrInheritDynamicControlState — not selected

The repeated task Optional access is locally avoidable, but it is a small helper
repair rather than a shared lookup representation decision. Selecting it now
would regress toward the micro-repair strategy rejected by TEST009-W.

### D. rejectComposedInvocationProjection — rejected

The failure protects a genuine invalid state for a source Closure requiring a
Context-local execution projection. Mature runtimes separate such invalid states
from ordinary successful traversal; they do not justify deleting them merely
because the exception constructor is visible in expansion evidence.

### E. PreparedBooleanCall exhaustive-switch fallback — rejected

No upstream evidence from this comparison establishes that generated exhaustive
switch failure machinery may be removed safely. It remains outside the selected
repair.

### F. standardErrorPrototype(String) — not selected

The method retains meaningful lazy type/hierarchy validation for individual
standard Error subtypes. Y's constructor-certified canonical Error proof does not
automatically generalize to every subtype.

### G. finishPreparingComposedCall — not selected

Its structured-capability consistency guard may contain optimizable paths, but
proving all protocol-entry invariants would require a broader audit. Its current
reach is also below the selected ValueLookup family.

### LOCAL_FRAME

Still excluded. TEST009-Z found no upstream or Protos evidence that overrides the
existing `LOCAL_FRAME_REPAIR_AUTHORIZED=NO` checkpoint.

## Selected TEST009-AA implementation contract

TEST009-AA is one implementation slice in `guillermomolina/protos`.

It should implement exactly this structural outcome:

```text
1. Generic authoritative lookup traverses a direct semantic delegation parent.
2. Ordinary ProtosObjectValue traversal reuses the existing direct parent representation.
3. Root ordinary Object terminates lookup as an ordinary miss without exception machinery.
4. ProtosRepresentedValue traversal still uses its representation-specific parent contract.
5. A null represented parent remains an invariant failure, not a lookup miss.
6. Unsupported runtime representations retain the existing UnsupportedOperationException behavior.
7. public delegationParent(...) retains its Optional<Object> API and semantics.
8. The local-slot Optional path is outside AA and must not be opportunistically changed.
9. No task, Prelude, Boolean-switch, composed-call, or LOCAL_FRAME cleanup is bundled into AA.
10. No new @TruffleBoundary is selected by Z.
```

The implementation may choose the smallest coherent private helper shape that
realizes those invariants against current HEAD. It must not preserve the old
internal Optional carrier merely to minimize the textual diff if doing so would
miss the selected structural repair.

## Functional validation requirements for AA

Focused coverage must preserve at least:

```text
ordinary local lookup value/home
ordinary inherited lookup value/home
root traversal -> normal miss
public delegationParent(root) -> Optional.empty()
public delegationParent(child) -> exact parent
represented-value delegation -> same semantic prototype
represented family requiring Prelude -> same existing failure without Prelude
null represented delegation parent -> failure, never root/miss
unsupported representation -> same UnsupportedOperationException
existing guarded/shared-inherited lookup behavior and parent identity guards
```

The exact affected test set must be derived from the current changed-file delta
under the repository validation policy. Because this is shared runtime lookup
production code, final validation is expected to require the repository's full
integrated suite unless current impact tooling proves otherwise.

## Causal compiler validation after AA

The same compiler root remains the authority:

```text
SELECTOR=protos-root:088d2ae81075aba8
SOURCE_FAMILY=protos/tools/test/Manifest.protos
POST_Y_TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
POST_Y_THROWABLE_FILL_IN_STACK_TRACE=11440
POST_Y_VALUE_LOOKUP_DELEGATION_PARENT_OWNER=1573
POST_Y_VALUE_LOOKUP_OPTIONAL_OR_ELSE_THROW_OWNER=1573
```

AA must obtain a fresh same-root capture and compare the ValueLookup owners.
Acceptance is causal reduction/elimination of the expansion attributable to the
old internal parent Optional carrier without displacement to an equivalent new
owner.

Do **not** encode an expected `-3146` gate. The current durable evidence does not
prove that both 1573 buckets are wholly the same eliminable path.

A valid causal result is therefore:

```text
DIRECT_PARENT_TRAVERSAL_PRESENT=YES
PUBLIC_OPTIONAL_API_PRESERVED=YES
UNSUPPORTED_REPRESENTATION_FAILURE_PRESERVED=YES
VALUE_LOOKUP_PARENT_OPTIONAL_EXPANSION_REDUCED=YES
UNRELATED_OWNER_BATCH=NO
TARGET_ROOT_REMEASURED=YES
```

If the target root also stops failing with `CodeTooLarge`, that is stronger
closure evidence, but it is not required to prove that AA repaired its selected
axis.

## Decision

```text
TEST009_Z_RESULT=NEXT_CAUSAL_STRUCTURAL_REPAIR_IDENTIFIED
TEST009_Z_AUTHORITY=UPSTREAM_MATURE_TRUFFLE_LANGUAGE_PATTERN_PLUS_POST_Y_CAUSAL_EVIDENCE

SELECTED_OWNER=ProtosValueLookup
SELECTED_MECHANISM=Generic authoritative lookup uses Optional-based delegation-parent transport while Protos guarded lookup and mature upstream runtimes use direct parent/result representations
SELECTED_REPAIR=Unify internal lookup traversal on a direct semantic parent representation while preserving represented-parent validation, unsupported-representation failure, and the public Optional delegationParent API

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
BOUNDARY_CHASING=NO
LOCAL_FRAME_INCLUDED=NO
MICRO_REPAIR_BATCH=NO

IMPLEMENTATION_READY=YES
NEXT_SLICE=TEST009-AA
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEW_FORMAL_ISSUE_REQUIRED=NO
```

AI assistance: this record was drafted with ChatGPT from the exact Protos and
upstream source revisions named above. No claim of human-only authorship or
review is made.
