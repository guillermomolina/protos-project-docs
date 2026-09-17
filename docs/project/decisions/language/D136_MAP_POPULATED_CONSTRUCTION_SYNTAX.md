# D136 — Map populated construction model and source sugar

Status: **RATIFIED — Candidate F′ selected after construction-semantics reconciliation**

Initial E′ approval: **2026-09-16**
Sequence-layout amendment approval: **2026-09-16**
F′ reconciliation approval: **2026-09-17**
Decision issue: `guillermomolina/protos#542`
Source audit: `AUD011` / `guillermomolina/protos#540`
Process correction: `GITHUB021` / `guillermomolina/protos#547`
Protos baseline reviewed: `e757ba1f47b5bd2e9f7f556578a4330932812472`

Nature: non-normative durable decision/rationale record for normative grammar and
Map-construction semantics under `guillermomolina/protos/spec/`.

## Supersession and preserved owner invariant

This record supersedes the earlier D136 Candidate E′ semantic contract while
preserving the already-approved source surface and layout unless explicitly stated
otherwise below.

D136 was reopened because E′ had allowed an implementation/reasoning model to
become the semantic definition. The owner-established invariant is:

```text
%{...} defines construction-time initial Map associations.

`:` inside %{...} is association definition during construction.
`=` / indexed assignment is the ordinary later mutation operation.
```

The semantic equation:

```text
%{...} == Map() followed by repeated post-construction atPut
```

is therefore **superseded and is no longer authoritative**.

Candidate F′ preserves the construction-time-initial-state invariant while also
preserving ordinary lexical lookup of the name `Map`, without adding a generic
keyed-construction protocol.

## Selected source surface

The selected source form remains:

```protos
%{}

%{
    key1: value1
    key2: value2
}
```

On one physical line, entries use the ordinary Protos sequence separator:

```protos
%{ key1: value1; key2: value2 }
```

A comma is **not** an entry separator. A trailing `;` before `}` is not admitted.
Continuation after `:` follows the ordinary continuation-newline rules.

The surface is unchanged by the F′ reconciliation. The semantic correction is
in how the fresh Map and its initial associations are established.

## Candidate F′ — ratified semantic contract

### 1. Ordinary `Map` identifier lookup

At the evaluation point of `%{...}`, the bare identifier `Map` is resolved by the
ordinary lexical/unqualified lookup rules.

The syntax does not capture, bypass, or hard-code the standard-prelude `Map`
binding. A local or lexical binding named `Map` is therefore observable.

This preserves the D136 constraint that the source form must not introduce a
privileged unshadowable Map constructor merely for sugar.

### 2. Standard-Map-factory eligibility gate

After resolving `Map`, perform ordinary member lookup for its `call` behavior.
The construction is eligible only when that lookup selects the canonical standard
Map factory behavior **from the canonical standard `Map` object**, either because
the selected binding is the standard `Map` itself or because the selected binding
delegates to it and inherits that behavior normally.

Equivalently, the accepted cases are:

```text
Map binding is canonical standard Map
    -> eligible

Map binding is an object delegating to canonical standard Map
and ordinary call lookup reaches canonical standard Map.call
    -> eligible

Map binding has a nearer arbitrary custom call
    -> ineligible

Map binding merely copies/aliases an equivalent-looking call locally
rather than inheriting canonical standard Map.call
    -> ineligible
```

Eligibility concerns the provenance of the behavior selected by ordinary lookup,
not merely the spelling of the binding and not an object's coincidental shape.

If ordinary `Map` lookup fails, `call` lookup fails, the selected `call` value is
not invokable under the ordinary callable contract, or the selected behavior is
not the eligible canonical standard Map factory behavior, construction signals an
ordinary `Error` before invoking an arbitrary custom factory and before evaluating
any source entry key or value expression.

D136 introduces no new public `atCreate`, `constructEntry`, builder, keyed-literal
interface, generic factory protocol, or public Association requirement.

### 3. Exactly one standard factory activation

For an eligible binding, activate the selected standard Map factory once with zero
positional arguments and with the ordinary invocation receiver preserved.

The result is one fresh open **standard normal Map** whose delegation parent is
the actual receiver of that standard factory activation. Therefore a user object
that genuinely delegates to standard `Map` may inherit the standard factory and
produce fresh Maps delegating to that user object.

The standard factory completes before any source entry key or value expression is
evaluated.

No arbitrary custom `Map.call` is invoked by `%{...}`.

### 4. No hidden construction lexical context

Entry key and value expressions execute in the same enclosing activation in which
the `%{...}` expression appears.

The nascent Map does not become `this`, `context`, a lexical parent, a return home,
or a guest-visible temporary binding merely because its initial associations are
being defined.

D136 does not create a new Closure, guest-visible builder, or construction
activation for `%{...}`.

### 5. Source-order processing

Entries are processed strictly left to right.

For each source entry:

```protos
keyExpression: valueExpression
```

the observable order is:

```text
1. evaluate keyExpression exactly once;
2. evaluate valueExpression exactly once;
3. compute the new key's normal Map hash exactly once;
4. search the associations already defined by earlier source entries using the
   normal Map lookup law;
5. if that search selects an existing association, signal Error;
6. otherwise define one new initial association using the evaluated key, recorded
   hash, and value;
7. only then proceed to the next source entry.
```

The value expression therefore executes before a duplicate/equal-key conflict can
be discovered for that entry.

No static duplicate-literal preflight may suppress these required evaluations or
replace the normal Map key law with host equality.

### 6. Normal Map hash/equality law remains authoritative

Initial-association conflict detection uses the same key-search law as normal
standard Map behavior.

For the newly evaluated key as the query key:

1. its current standard `hash` is computed exactly once for that search;
2. only already-defined associations with the same recorded hash are candidates;
3. candidates are considered in initial/source insertion order;
4. equality is sent in the normal Map direction:

   ```text
   queryKey == storedRepresentativeKey
   ```

5. `==` must produce canonical `true` or `false` under the existing Map/equality
   contract; ordinary Error/control-transfer behavior otherwise propagates.

There is no identity shortcut, host-language equality shortcut, symmetric
re-check, transitive closure, or construction-specific equivalence relation.

Consequently "duplicate/equal construction key" means exactly:

> the new key, used as a normal Map query under the current hash/`==` rules,
> selects an association already defined by an earlier source entry.

### 7. Duplicate/equal initial definitions are an Error

If the new key selects an association already defined by an earlier source entry,
construction signals an ordinary `Error`.

The later entry does **not** replace the earlier value, does not replace the
representative key, and does not become an implicit mutation.

This is the deliberate construction policy selected by F′. It is not inherited
from `atPut`, because `atPut` is no longer the semantic authority for construction.

Example:

```protos
%{
    "a": first()
    "a": second()
}
```

performs `first()`, defines the first association, performs `second()`, then
encounters the second construction definition conflict and signals `Error`.

### 8. Representative key, recorded hash, and insertion order

For each successfully defined initial association:

- the source key object becomes that association's stored representative key;
- the hash computed for that source key at definition time becomes its recorded
  hash;
- the evaluated value becomes its mapped value;
- successful initial associations are ordered by their source definition order.

A failed conflicting entry does not replace any earlier representative key or
value.

Later mutation and later key mutation continue to follow the existing standard
Map rules. In particular, mutating a key or state used by its `hash`/`==` behavior
does not retroactively repair or rehash already-recorded associations.

### 9. Construction is not `atPut`

Defining an initial association during `%{...}` does **not** send `atPut`, does
not perform ordinary indexed assignment, and does not dispatch through a custom
`atPut` inherited by the fresh Map.

The implementation may share internal storage machinery with ordinary Map
operations only when that sharing is observationally equivalent to this contract.
Internal reuse must not turn `atPut` dispatch, replacement semantics, mutation
hooks, or custom receiver behavior into construction semantics.

After successful construction, ordinary later mutation is unchanged:

```protos
map[key] = value
```

continues to use ordinary indexed assignment and ordinary `atPut` selection and
invocation, including any applicable custom behavior on that Map object.

### 10. Failure, effects, control transfer, and suspension

Construction is fail-fast.

An Error, non-local control transfer, or other ordinary control transfer during:

- `Map` lookup;
- `call` lookup/eligibility;
- standard factory activation;
- a key expression;
- a value expression;
- `hash`;
- `==`;
- or initial-association definition

stops the construction immediately. Later entry expressions are not evaluated.

Already-completed external effects are not rolled back.

The nascent Map is not returned when construction fails. F′ does not require a
particular failed partial backing representation to survive when it is otherwise
unobservable.

Existing standard Map comparison-scope, Error, control-transfer, and suspension
rules remain authoritative. F′ introduces no hidden transaction, rollback,
atomic block, or new suspension point.

### 11. Empty construction

`%{}` follows the same ordinary `Map` lookup and standard-factory eligibility
rules as a non-empty construction.

On success, it activates the eligible standard Map factory once with zero
arguments and returns the resulting fresh open standard normal Map with zero
initial associations.

There is no separate empty-form semantic family.

### 12. Shadowing consequences

Shadowing remains observable, but the old E′ custom-factory participation rule is
superseded.

```text
canonical standard Map binding
    -> works

binding delegating to canonical standard Map and inheriting canonical Map.call
    -> works

binding with a nearer arbitrary custom call
    -> Error before that custom call executes and before entry evaluation

unrelated object merely supporting atPut
    -> not a Map-construction factory
```

This is a material narrowing from E′ and was explicitly surfaced before F′ owner
approval.

The narrowing is deliberate: being invokable, being index-assignable, or merely
having Map-like slots does not establish authority to create the standard normal
Map whose construction-time initial state `%{...}` defines.

### 13. `:` remains contextual construction punctuation

Inside expression-position `%{...}`, `:` separates the entry key expression from
the entry value expression and denotes one construction-time initial association
definition.

It is not ordinary object-slot creation at that location and does not change the
meaning of `:` elsewhere.

Key expressions are ordinary expressions, not implicit names or Strings:

```protos
%{
    name: value
}
```

uses the value produced by the expression `name` as the Map key. A String key is
written explicitly as `"name"`.

### 14. Layout and grammar remain as previously approved

Across physical lines, newline separates entries. On one physical line, `;`
separates entries. Comma does not separate Map-construction entries. A trailing
`;` before `}` is not admitted. Continuation after `:` follows ordinary
continuation-newline behavior.

`%{...}` remains an expression-position Map construction form. Existing
pattern-position `%{...}` remains owned by matching and retains its independent
pattern grammar and semantics.

D136 changes no Map-pattern subset/exactness/remainder, snapshot, capture,
separator, or matching behavior.

### 15. Ordinary postfix composition remains available

Because `%{...}` is an expression, existing postfix operations may follow the
completed successful construction subject to the normal grammar and invocation
rules.

No postfix operation observes a result from a failed construction.

## Explicit F′ delta from superseded E′

The following changes were surfaced and explicitly approved before F′
ratification:

```text
E′ repeated post-construction atPut semantics
    -> F′ construction-time initial association definition

E′ duplicate/equal key: ordinary atPut replacement / later value wins
    -> F′ duplicate/equal initial definition: Error

E′ arbitrary shadowed custom Map.call may participate
    -> F′ only canonical standard Map.call selected directly or by inheritance

E′ arbitrary atPut-capable custom factory result may be populated
    -> F′ result is always a fresh standard normal Map from the eligible factory

E′ custom atPut dispatch occurs for every construction entry
    -> F′ construction performs no atPut dispatch
```

These are semantic corrections, not parser/layout changes.

## GITHUB021 invariant/delta consistency check

The final Candidate F′ was compared against the recorded owner-approved D136
invariants before approval and ratification.

```text
PRESERVE  %{...} defines construction-time initial Map associations       PASS
PRESERVE  `:` construction definition distinct from later `=` mutation     PASS
PRESERVE  ordinary lexical lookup of the name Map                          PASS
PRESERVE  no privileged unshadowable Map constructor                       PASS
PRESERVE  normal standard Map hash/== key law                              PASS
PRESERVE  representative-key / recorded-hash model                         PASS
PRESERVE  source-order initial association order                            PASS
PRESERVE  enclosing-activation evaluation / no hidden lexical context       PASS
PRESERVE  selected %{...}, `:`, newline/`;` surface                         PASS
PRESERVE  IdentityMap and matching remain separate                          PASS
PRESERVE  no generic iterable/pair/keyed-construction institution           PASS

CHANGE    duplicate/equal initial key: replacement -> Error                 EXPLICITLY APPROVED
CHANGE    arbitrary custom Map.call participation -> rejected                EXPLICITLY APPROVED
CHANGE    arbitrary atPut-capable factory result -> rejected                 EXPLICITLY APPROVED
CHANGE    atPut dispatch during construction -> none                         EXPLICITLY APPROVED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

No hidden contradictory consequence remains bundled into the selected candidate.
If a future decision wants arbitrary custom factories or a generalized keyed
construction protocol, that capability must be designed and approved explicitly
rather than inferred from F′.

## Boundaries and deferred work

D136 does **not** add or change:

- `IdentityMap` construction sugar;
- generalized `%Factory{...}` syntax;
- a public `Association`, Pair, Tuple, or KeyValue type;
- generic iterable/spread/merge construction;
- comprehensions;
- matching semantics;
- object-slot / Map-entry unification;
- immutable/frozen Map-literal mode;
- a new Error subtype merely for duplicate construction definitions.

Those remain independent future decisions if evidence establishes need.

## Implementation requirements for follow-up I040

The reopened implementation work must reconcile the already-delivered E′ path
against this F′ contract. At minimum it must prove:

- ordinary lexical `Map` lookup still occurs;
- only canonical standard Map factory behavior selected directly or by inheritance
  is eligible;
- arbitrary custom `Map.call` is not invoked;
- one fresh standard normal Map is produced by the eligible standard factory;
- construction entries do not dispatch `atPut`;
- key then value evaluation occurs exactly once per reached entry;
- normal Map `hash` / directed `==` search law is reused;
- duplicate/equal initial definitions signal Error after the reached value
  expression has evaluated;
- representative key, recorded hash, and source order follow this record;
- failure/control-transfer effects and later-entry suppression follow this record;
- the already-approved surface and pattern boundary are preserved.

The implementation may optimize internal allocation or storage only when all
observable behavior above remains identical.

## Publication boundary

This F′ ratification publication changes only the non-normative durable decision
record in `guillermomolina/protos-project-docs`.

At durable-ratification publication time:

```text
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
MATCHING_SEMANTICS_CHANGED=NO
IDENTITY_MAP_SURFACE_CHANGED=NO
GENERALIZED_FACTORY_SURFACE_CHANGED=NO
PUBLIC_ASSOCIATION_INTRODUCED=NO
VALIDATION_CLASS=GOVERNANCE_DOCUMENTATION_ONLY
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REQUIRED_DURABLE_PUBLICATION=THIS_FILE
```

`PROJECT_RECORD_REVISION` is the exact `protos-project-docs` commit that publishes
this replacement record. It is recorded in the final D136 GitHub Issue ratification
and closure comment after publication and re-read.

D136 is not formally closed merely by preparing this file. Closure occurs only
after the exact published project-record revision is re-read and the live Issue /
Project postconditions required by the canonical `guillermomolina/protos/AGENTS.md`
flow are verified.
