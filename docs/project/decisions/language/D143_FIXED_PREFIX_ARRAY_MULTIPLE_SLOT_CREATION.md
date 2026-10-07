# D143 — Fixed-prefix standard-Array multiple slot creation

Status: **RATIFIED — Candidate C′ + syntax S2**

Approval date: **2026-09-18**  
Decision issue: `guillermomolina/protos#573`  
Trigger: `AUD011` / `guillermomolina/protos#540`  
Reviewed Protos baseline: `90c145cc90b5cca95513c33a81cd60bc47c1a446`

This is durable non-normative rationale. Observable semantics remain authoritative
only after publication under `guillermomolina/protos:spec/**`.

## Selected surface

D143 selects fixed-prefix multiple slot creation from an eligible standard Array:

```protos
(a, b): values
(a, b, c): values
```

The target contains **two or more ordinary identifiers**. One binding continues
to use ordinary `name: value`.

`(a, b)` does not become an expression, tuple, comma-expression, reusable
destructuring target, or pattern. Existing forms remain distinct:

```protos
(a + b)           // grouping
(a, b) => a + b   // Closure parameters
```

Outside this dedicated `:` target, `(a, b)` remains invalid.

## Approval provenance

The project owner explicitly approved Candidate C′ after the complete decision
packet and examples:

```text
aprobado
```

The syntax was then reviewed separately, including its parser interactions and
the explicit invariant that `(a, b)` must not become a general expression. The
project owner approved proceeding with that exact syntax:

```text
ok pues adelante
```

No broader destructuring capability is bundled into those approvals.

## Exact semantic contract

For `N` target names:

1. evaluate the RHS exactly once;
2. require a result that **owns standard Array indexed state**;
3. require `source_length >= N`;
4. shallowly observe exactly the first `N` element references in ascending index
   order before any target slot is created;
5. create the `N` target names left-to-right using ordinary bare `:` creation in
   the **current slot-creation context**;
6. if all creations succeed, return the exact original RHS object.

Extra Array elements are ignored. No remainder object is created.

The operation invokes no user-visible `at`, iterator, matcher, `deconstruct`,
`componentN`, callback, or arbitrary-object positional projection protocol.
Open, closed, and frozen eligible Arrays behave the same for this read-only
observation.

Because the ordinary current slot-creation context is reused, targets become
activation locals in a Closure/function, module-context slots at module top
level, and object slots inside an object body. No hidden Closure activation or
new lexical scope exists.

## Failure and partial creation

RHS evaluation failure, ineligible source, or insufficient source length fails
before any target slot is created.

After prefix observation succeeds, target creation uses existing left-to-right
slot semantics. There is **no transaction and no rollback**. If a later target
conflicts, earlier successful target creations remain.

Duplicate target names gain no special static rule. For example:

```protos
(a, a): [10, 20]
```

may create the first `a` and then fail on the second through ordinary local-slot
conflict semantics. Implementations must not import Closure-parameter
unique-name validation and thereby change this approved behavior.

## Assignment boundary

Initial capability is exactly:

```text
MULTIPLE_SLOT_CREATION_ONLY
```

This is not approved:

```protos
(a, b) = values
```

Multiple assignment remains a separate future decision.

## Explicit exclusions

D143 does not add rest/remainder binding, wildcards/discards, holes, defaults,
nested destructuring, Map/named/object destructuring, arbitrary-object
deconstruction, generic tuple/deconstruct/component protocols, matchers, guards,
OR, aliases, type guards, exhaustiveness, shared `caseOf` pattern grammar, or
multiple assignment.

## Existing-decision reconciliation

**D080-A is preserved.** Arbitrary objects do not gain a universal positional
deconstruction contract. D143 observes only intrinsic standard Array indexed
state.

**D131-C is preserved.** The removed dedicated pattern language is not rebuilt.
The parenthesized name list is only a narrowly scoped slot-creation target.

The existing slot model is preserved:

```text
:  -> creation
=  -> assignment
```

## Evidence and comparative result

AUD011 found repeated real production friction, including eight consecutive
SHA-256 state reads and several Test Tool carriers. A decisive case in
`Progress.protos` uses a seven-element state while naming only the first three or
four positions, so exact-length unpacking does not cover the demonstrated need.
The selected shape is therefore fixed **prefix**, not exact arity.

The comparison covered Self/Smalltalk, Python, JavaScript, Ruby, Elixir/Erlang,
Rust, Kotlin and C++.

- Self/Smalltalk show that general destructuring is not fundamental.
- Python/C++ provide fixed structural binding precedent but exact/product shape
  is too restrictive for the demonstrated Protos prefix use.
- JavaScript/Ruby support tolerant leading-value binding but include broader
  destructuring/multiple-assignment surfaces Protos does not need.
- Elixir/Erlang/Rust place destructuring in broader pattern systems, precisely
  the institution D131 removed.
- Kotlin `componentN`-style object decomposition conflicts with D080's rejection
  of a generic positional object contract.

Candidate A (no feature) and B (IIFE + spread) remain coherent alternatives, but
B creates another activation and is particularly indirect for a prefix of a
larger Array. Candidate C′ is the smallest capability that directly addresses
the demonstrated current-context binding friction.

## Syntax result

The final syntax comparison was:

```protos
(a, b): values
[a, b]: values
a, b: values
```

`[a, b]` was rejected because D130 already owns that spelling as Array
construction and it visually recreates Array-pattern syntax. The unparenthesized
comma form was rejected because comma is not a general Protos expression
separator. A new keyword/operator (`let`, `bind`, `:=`) was unnecessary because
`:` already means exactly the selected operation: slot creation.

## GITHUB021 invariant/delta check

```text
D080_NO_GENERIC_POSITIONAL_OBJECT_PROTOCOL=PRESERVED
D131_PROTOCOL_FIRST_MATCHING=PRESERVED
D131_REMOVED_PATTERN_INSTITUTIONS_REMAIN_REMOVED=PRESERVED
COLON_REMAINS_SLOT_CREATION=PRESERVED
EQUALS_REMAINS_ASSIGNMENT=PRESERVED
CURRENT_SLOT_CREATION_CONTEXT=PRESERVED

FIXED_PREFIX_STANDARD_ARRAY_SOURCE=APPROVED
SOURCE_LENGTH_GTE_TARGET_COUNT=APPROVED
SHALLOW_PREFIX_BEFORE_CREATION=APPROVED
LEFT_TO_RIGHT_ORDINARY_SLOT_CREATION=APPROVED
NO_TRANSACTION_OR_ROLLBACK=APPROVED
SUCCESS_RESULT_IS_EXACT_RHS=APPROVED
MULTIPLE_SLOT_CREATION_ONLY=APPROVED

SYNTAX_PARENTHESIZED_NAMES_COLON_RHS=APPROVED
PARENTHESIZED_COMMA_LIST_BECOMES_GENERAL_EXPRESSION=NO
TUPLE_OR_COMMA_EXPRESSION_INTRODUCED=NO
CLOSURE_PARAMETER_GRAMMAR_REINTERPRETED=NO
ARRAY_CONSTRUCTION_GRAMMAR_REINTERPRETED=NO
GENERIC_DESTRUCTURING_GRAMMAR_INTRODUCED=NO
MULTIPLE_ASSIGNMENT=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

No previously approved invariant was reopened and no hidden semantic delta
remains after syntax refinement.

## Strongest counterargument and recovery path

The strongest argument against C′ is compatibility surface: explicit indexing
already works, and D143 could have been deferred without a foundational rewrite.
If positional Arrays later become rare carriers, Core will retain syntax for a
less-common idiom.

The project accepts that risk because present repository use is already real and
repeated, while C′ keeps richer capability absent. Future rest binding, a
tuple/product abstraction, object decomposition, or multiple assignment can be
evaluated independently if actual evidence appears.

## Publication boundary

This record does not itself make D143 normative. Before implementation begins,
`guillermomolina/protos:spec/**` must publish the exact grammar plus source-domain,
prefix-observation, ordering, failure, partial-creation, context, and expression
result rules above, with normal specification revision/changelog bookkeeping.

```text
D143_STATUS=RATIFIED
SELECTED_CANDIDATE=C_PRIME
SELECTED_SYNTAX=S2
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
VALIDATION_CLASS=GOVERNANCE_DOCUMENTATION_ONLY
```
