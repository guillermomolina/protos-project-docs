# PERF010-A — Third causal ablation readiness evidence

Status: INVESTIGATION COMPLETE — ABLATION 3 ESTABLISHED

This is a durable, non-normative performance-investigation record for
`PERF010-A / #691`. It does not define Protos language semantics and does not
authorize a production optimization.

## Revision identity

```text
PROTOS_REVISION=6e7d89194925ba9fa2cd9c5c45aefa72d9939621
BENCHMARK_REVISION=38f82606ab9afae6e8cf52cb6116fd5c6007b2e6
PINNED_PERF010A_REVISION=bc0471184bf6dbbf03d0c6b09ef7b9e28aede014
```

At investigation completion, current `guillermomolina/protos` was four commits
ahead of the PERF010-A pinned revision. The compared commits did not modify the
investigated lookup implementation, the Bytecode lookup operation, or any of the
four PERF010-A workloads. The investigated surface is therefore unchanged
between the retained PERF010-A pin and current main.

## Prior causal state

Two interventions preceded this investigation.

### Ablation 1 — semantic/helper Bytecode dispatch

Ablation 1 is retained as valid causal evidence. It produced positive raw
canonical timing reductions in all four workloads, but paired control movement
prevented establishing a clear positive corrected contribution for every
workload.

```text
PERF010A_ABLATION_1=VALID
CAUSAL_SIGNAL=ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
```

### Ablation 2 — direct invocation-local lookup bypass

Ablation 2 replaced:

```text
activation.lookup(name)
```

with:

```text
activation.context().readLocalSlot(name)
```

The ablation failed correctness in canonical and control modes for all four
workloads. It is retained as negative semantic-boundary evidence, not timing
evidence.

```text
PERF010A_ABLATION_2=INVALID
ABLATION_2_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
ABLATION_2_PERFORMANCE_SIGNAL=NOT_ESTABLISHED
LEXICAL_LOOKUP_DIRECT_LOCAL_BYPASS=SEMANTICALLY_INVALID
```

## Current lookup flow

Current `ProtosBytecodeRootNode.Lookup.perform` calls
`ProtosActivation.lookup(name)`.

The implementation order is:

```text
Lookup.perform
    -> ProtosActivation.lookup(name)
       -> context.readLocalSlot(name)
          -> hit: return
       -> for each capturedLexicalContexts entry, nearest first
          -> lexicalContext.readLocalSlot(name)
             -> hit: return
       -> ProtosValueLookup.readMember(receiver, name, prelude)
          -> ordinary receiver/delegation lookup
          -> method Closure binding when applicable
       -> missing
    -> UnqualifiedLookupError
```

The ordering is semantically significant: invocation-local bindings precede
captured lexical bindings, captured lexical bindings preserve nearest-first
shadowing, and receiver/delegation lookup occurs only after lexical exhaustion.

## Why Ablation 2 failed

Closure invocation creates a fresh execution context while preserving captured
lexical contexts from closure creation.

For ordinary closure capture:

```text
lexicalContextsForClosureCapture()
    = [current context, previously captured contexts...]
```

For closure invocation:

```text
current context
    = fresh prelude.newExecutionContext()

captured lexical contexts
    = closure.capturedLexicalContexts()
```

Parameters are installed in the fresh invocation-local context. Enclosing
bindings remain in captured lexical contexts.

The four PERF010-A workloads therefore contain both local and captured lexical
unqualified lookups:

```text
micro/slot-read             = MIXED
micro/closure-call          = MIXED
micro/method-call           = MIXED
runtime/monomorphic-dispatch = MIXED
```

Examples shared by the workloads include:

- `count` in the `repeat` activation: local parameter;
- `operation` from the nested `ifTrue` body: captured from the `repeat`
  activation;
- recursive `repeat`: captured from the module context;
- workload callback bindings such as `holder`, `identity`, or `receiver`:
  captured from the module context.

Therefore Ablation 2 preserved some local reads but removed required captured
lexical reads. The existence and actual use of captured lexical contexts is
sufficient to explain the observed correctness failure; no timing conclusion can
be drawn from the failed variant.

## Established third causal ablation

### Target component

`ProtosObjectValue.readLocalSlot(name)` currently performs:

```java
return localSlots.containsKey(name)
        ? Optional.of(localSlots.get(name))
        : Optional.empty();
```

On a successful local-slot lookup this performs two probes of the same
`LinkedHashMap`: `containsKey(name)` followed by `get(name)`.

The slot representation has a stronger invariant:

- `createLocalSlot(name, value)` rejects Java `null`;
- `assignLocalSlot(name, value)` rejects Java `null`;
- copied/composed slots originate from the same representation;
- Protos `null` is represented by the non-null value
  `ProtosNullValue.INSTANCE`.

Consequently, for this private slot map:

```text
localSlots.get(name) == null
    <=> the slot is absent
```

A single `get(name)` can therefore distinguish miss from every valid stored
Protos value without changing observable semantics.

### Diagnostic operation

The established diagnostic intervention is intentionally narrower than changing
`readLocalSlot` globally.

Add a diagnostic single-probe local-slot reader in `ProtosObjectValue` that
performs one `localSlots.get(name)` and maps Java `null` to
`Optional.empty()`.

Use that reader only for the two lexical lookup positions inside
`ProtosActivation.lookup`:

```text
current activation context
captured lexical contexts
```

Leave unchanged:

```text
ProtosValueLookup
receiver/delegation member lookup
ordinary ProtosObjectValue.readLocalSlot callers
lookup order
captured-context traversal
shadowing
missing-name behavior
UnqualifiedLookupError
method binding
```

This isolates one causal component: the redundant second map probe on successful
activation lexical local-slot reads.

A global replacement of `readLocalSlot` is specifically excluded because
`ProtosValueLookup` also calls it during receiver/member lookup; changing that
path would mix lexical-resolution cost with member/delegation lookup cost.

## Semantic equivalence

The diagnostic variant preserves by construction:

```text
same current-context precedence
same captured-context order
same number of lexical contexts traversed
same nearest-first shadowing
same returned value identity
same ProtosNullValue behavior
same receiver/delegation fallback
same missing-name behavior
same error behavior
```

The only intended operational difference is:

```text
successful activation lexical read

baseline:
    Map.containsKey(name)
    Map.get(name)

diagnostic:
    Map.get(name)
```

A miss likewise remains one map probe in both variants.

No new semantic or platform decision is required.

## Structural confirmation

The next diagnostic harness must fail closed unless structural evidence confirms
all of the following:

1. the diagnostic activation lexical reader performs `Map.get` without the
   baseline `Map.containsKey` probe;
2. both the current-context and captured-context reads in
   `ProtosActivation.lookup` use the diagnostic reader;
3. `ProtosValueLookup` continues through ordinary `readLocalSlot`;
4. receiver/member lookup and the semantic/helper Bytecode structure remain
   unchanged.

Source/bytecode structural confirmation is preferable to requiring the new
reader to remain visible as a JFR frame because Graal may inline it.

## Relation to Ablation 1

```text
RELATION_TO_ABLATION_1=OUTSIDE
```

Ablation 1 removes the semantic/helper Bytecode wrapper-dispatch component.
Ablation 3 retains that execution structure and removes work inside activation
lexical local-slot resolution. The interventions therefore target different
causal components, although they occur on the same dynamic execution path.

Their percentages must not be summed without experimental evidence.

## Investigation result

```text
PERF010A_ABLATION_3=ESTABLISHED

PROTOS_REVISION=6e7d89194925ba9fa2cd9c5c45aefa72d9939621
BENCHMARK_REVISION=38f82606ab9afae6e8cf52cb6116fd5c6007b2e6

SLOT_READ_LOOKUP=MIXED
CLOSURE_CALL_LOOKUP=MIXED
METHOD_CALL_LOOKUP=MIXED
MONOMORPHIC_DISPATCH_LOOKUP=MIXED

ABLATION_2_FAILURE_CAUSE=local-only bypass preserved local reads but removed required captured lexical reads

ABLATION_3_TARGET=redundant containsKey+get double map probe on successful activation lexical local-slot reads
ABLATION_3_FILE=src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java;src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
ABLATION_3_METHOD=ProtosActivation.lookup plus diagnostic ProtosObjectValue single-probe local-slot reader
ABLATION_3_CURRENT_OPERATION=context/captured readLocalSlot(name) performs containsKey(name) then get(name) on a hit
ABLATION_3_DIAGNOSTIC_OPERATION=context/captured lexical reads use one get(name), mapping Java null to Optional.empty and non-null to Optional.of(value), while ordinary readLocalSlot and receiver/member lookup remain unchanged

COMMON_PATH=YES
SEMANTICS_PRESERVED_BY_CONSTRUCTION=YES
FOUR_WORKLOADS_SUPPORTED=YES
SINGLE_CAUSAL_COMPONENT=YES
IMPLEMENTATION_BOUNDED=YES
STRUCTURAL_MARKER_POSSIBLE=YES
TIMING_COMPARISON_MEANINGFUL=YES

RELATION_TO_ABLATION_1=OUTSIDE
NEW_DECISION_REQUIRED=NO

PRODUCTION_OPTIMIZATION=NO
PERF010_READY=NO
```

## Next step

The next bounded step is implementation of exactly one diagnostic Ablation 3 in
`guillermomolina/protos-benchmarks`, against the existing PERF010-A comparable
evidence contract.

It must not publish a production optimization to `guillermomolina/protos`.
Correctness must pass before timing can be interpreted. If the diagnostic
variant does not preserve correctness, its timing is invalid and must fail
closed as Ablation 2 did.

Only after execution may PERF010-A determine whether this component carries a
measurable attributable cost or whether investigation must move to another
common-path component.
