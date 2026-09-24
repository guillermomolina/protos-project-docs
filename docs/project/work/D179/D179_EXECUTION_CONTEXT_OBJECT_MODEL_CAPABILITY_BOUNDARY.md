# D179 — Execution context object-model capability boundary

## Record status

```text
FORMAL_IDENTIFIER=D179
GITHUB_ISSUE=guillermomolina/protos#703
DECISION_STATE=OPEN / RESEARCH REQUIRED
TRIGGER=PLAT036 / guillermomolina/protos#702
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
PROJECT_RECORD_BASELINE_REVISION=3a4d7a2234480e77917a110e3ec4bf62837c8dcb
PLAT036_DEPENDENCY_STATE=BLOCKED_PENDING_D179
OWNER_APPROVAL=NOT_YET_REQUESTED_FOR_ANY_CANDIDATE
```

This is a durable, non-normative investigation-opening record. It records why
D179 exists and the evidence that caused PLAT036 to stop before ratification.
It does **not** select or ratify a language semantic.

Live status, scheduling, approval, and closure authority remain in the GitHub
Issue. Normative language authority remains under `guillermomolina/protos/spec`.

## Owner-directed coordination

On 2026-09-24 the project owner explicitly requested:

1. a dedicated design Issue for the execution-context/object equivalence
   question rather than deciding it accidentally inside PLAT036;
2. live Issue reconciliation;
3. durable evidence in `guillermomolina/protos-project-docs`; and
4. PLAT036 to wait for the result of the new decision.

That instruction authorizes the allocation and dependency recording only. It is
not approval of any D179 semantic candidate.

## Trigger discovered by PLAT036

PLAT036 reconstructed the current lexical-state path and found that an execution
`context` is treated as an ordinary mutable Protos object whose lexical bindings
participate in ordinary slot operations.

The critical stress case is:

```text
current lexical binding x exists
  -> context.removeSlot("x")
  -> x becomes absent from the current execution context
  -> later lookup may continue to an outer lexical context / receiver path
```

This is not merely a slow implementation of an otherwise fixed lexical binding.
It means reflective structural mutation can change which lexical/member binding a
later source-level name denotes.

PLAT036 had previously treated the following as an invariant to preserve:

```text
execution contexts are ordinary Protos objects
lexical bindings are execution-context slots
ordinary structural context mutation remains observable
```

The owner has now explicitly reopened the **capability consequence** of that
model for dedicated semantic review. PLAT036 must therefore not ratify a runtime
representation whose correctness depends on either preserving or removing that
capability until D179 decides it.

## Why the question is broader than removeSlot

`removeSlot` is the strongest known architectural stress case, but D179 must
audit the whole execution-context structural capability boundary.

At minimum the investigation must distinguish:

- lexical binding identity;
- binding presence versus guest `null`;
- creation and late creation;
- assignment of an existing lexical binding;
- reflective read and enumeration;
- structural removal;
- dynamic context slots not originating from lexical declarations;
- alias/copy-like structural operations where current protocols permit them;
- `close` and `freeze`;
- escaped `context` observation/mutation;
- Closure capture by reference;
- debugger/tooling projection.

The key design question is whether:

```text
context is an Object
```

necessarily entails:

```text
every ordinary Object structural capability applies to every lexical binding
```

or whether execution contexts are ordinary Protos objects with explicit
capability invariants.

## Comparative evidence that caused the stop

The PLAT036 investigation compared mature Truffle implementations and found a
strong recurring representation pattern:

```text
ordinary statically known local
    -> indexed frame/local slot

captured local
    -> materialized frame and/or by-reference cell when required

dynamic/reflective lookup
    -> separate exceptional/dynamic path

debugger/scope view
    -> projection over the semantic/runtime representation
```

Representative evidence inspected during the PLAT036/D179 trigger analysis:

- `truffleruby/truffleruby@0e6fa6a950dce7154d54f3c9c63056c4eb925ffd`
  — current locals use frame slots; outer locals carry fixed frame depth and
  slot identity.
- `SOM-st/TruffleSOM@73f6d2e654022565ec7c7e8ba95ae18340a862ce`
  — local slot identity is static; blocks capture/materialize lexical frames.
- `oracle/graaljs@b700215562db9a8b33fb391d971faeb652131184`
  — closures/block scopes use frame state and materialized enclosing frames,
  with dynamic scope mechanisms handled separately.
- `oracle/graalpython@cebcc10a20c502f1956a0c436c94f53380944d61`
  — Bytecode DSL locals are used directly for ordinary local state; cell/free
  variables pay the additional `PCell` indirection only where closure
  semantics require it.
- `apple/pkl@d5a7dc021e1ea0c71afd52509238926c1d614851`
  — lexical reads are compiled to slot plus lexical level.
- `hpi-swa/trufflesqueak@818519b2b6a6556bc524e9e0d08f7b51969cb61a`
  — guest Context objects and Truffle materialized frames are coordinated
  rather than maintained as independent authoritative value stores.
- `graalvm/simplelanguage@5a2b35790539ab1fe5c61f02b71e3485fd0e0fb1`
  — the minimal Truffle example resolves known locals to integer frame slots.

These systems do **not** decide Protos semantics. They establish only that
static lexical identity plus frame-oriented storage is a mature Truffle pattern
and that Protos' current ability to structurally remove a lexical binding and
re-open lookup is unusual enough to deserve an explicit language-value review.

The full D179 comparative-research contract remains outstanding. In particular,
the decision packet must examine Self/Smalltalk, Python `del`/unbound locals,
JavaScript Environment Records/object environments, and the complete Protos
semantic capability surface before recommending a candidate.

## Current PLAT036 consequence

PLAT036's earlier pending recommendation used a backend-neutral indexed semantic
context store as the sole guest-binding authority.

Further comparison showed that a more frame-first candidate may be materially
more aligned with mature Truffle implementations:

```text
known current binding
    -> Bytecode/frame local

known captured binding
    -> materialized local/frame or cell where required

dynamic context extension
    -> dynamic overflow

semantic context
    -> one coherent guest view, never a divergent second value authority
```

Whether that architecture is semantically admissible depends materially on
D179. The platform decision must therefore wait.

This record intentionally does **not** establish that frame-first is the eventual
PLAT036 winner, and it does not claim that lexical representation explains the
PERF010/PERF011 benchmark magnitude.

## Candidate families D179 must investigate

The owning Issue requires at least:

- **A — KEEP current:** lexical slots remain structurally removable and removal
  can expose outer/member lookup.
- **B — UNBOUND identity:** removal/unbinding preserves the lexical identity, so
  later access does not fall through merely because the local lacks a value.
- **C — RESTRICT lexical structural mutation:** lexical membership is fixed while
  dynamic context slots retain ordinary add/remove behavior.
- **D — stronger fixed lexical namespace:** dynamic structural state is clearly
  separated from the declaration-derived lexical namespace.
- **E — credible alternatives discovered by exhaustive research.**
- **F — DEFER / keep current** with explicit cost to PLAT036 and future
  implementation freedom.

No candidate is selected in this record.

## Required value question

The final D179 packet must answer both sides explicitly:

> What useful Protos program becomes impossible or materially worse if lexical
> structural removal is restricted?

and:

> What cost does every ordinary lexical access pay, semantically or
> architecturally, solely because this capability remains possible?

The decision must be based on Protos design value, not on implementation
convenience or majority practice in other languages.

## Publication scope

This opening publication changes no Protos product source, specification,
tests, benchmark, or runtime behavior.

It records only:

- the D179 allocation and semantic question;
- the evidence that caused the question to be separated from PLAT036;
- the dependency that prevents PLAT036 ratification before D179; and
- the comparative-runtime evidence already obtained at the trigger point.

## Next gate

D179 must now run the exhaustive `AGENTS.work/DESIGN.md` investigation and
produce a complete decision packet.

Only after the project owner explicitly approves an exact D179 candidate may its
semantic result be ratified. PLAT036 may then resume and re-evaluate its runtime
candidate set against the ratified D179 result.
