# AUD009-B5 — Boolean control and looping complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#608`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence / closure revision: `82cc94664be79a3aac121b0babb70cf82b1c4284`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#608`, issue comment
`5738851349`, 2026-09-19.

## Purpose and boundary

AUD009-B5 reviewed the existing strict-Boolean control model, ordinary Boolean
protocol, logical operator lowering, truthiness boundary and standard
Closure-specific `while` protocol under the retrospective AUD009
complexity/necessity methodology.

B5 does not reopen matching/guards, object/context/binding semantics, Closure
identity/non-local-return, Error/handler/ensure semantics, collection API breadth
or concurrency/backend architecture.

## Final classification ledger

```text
canonical true / false only                      KEEP
no Boolean prototype/binding                     KEEP
Boolean selectors as ordinary Object behavior    KEEP
strict semantic-family receiver check            KEEP

not()                                             KEEP
! -> not()                                        KEEP

ifTrue                                            KEEP
ifFalse                                           KEEP
ifTrueIfFalse                                     KEEP
ordinary eager callback expressions              KEEP
selected-only callback validation/invocation      KEEP
ordinary polymorphic branch callback invocation   KEEP

and                                               KEEP
or                                                KEEP
strict Boolean result validation                  KEEP
&& -> and(() => rhs)                              KEEP
|| -> or(() => rhs)                               KEEP
no language-wide truthiness                       KEEP
no dedicated if/else conditional syntax           KEEP

Closure.while                                     KEEP
Object-hosted Closure-family while behavior       KEEP
Closure-only condition/body                       KEEP
up-front operation-domain validation              KEEP
activation-time parameter binding                 KEEP
strict true/false loop decision                   KEEP
ignored body result                               KEEP
normal while result = canonical null              KEEP
no loop-specific scheduler/control institution    KEEP
no dedicated while syntax                         KEEP

primitive for                                     ABSENT / RETAIN ABSENCE
```

No B5 mechanism is classified for removal. No B5 Dxxx route is required.

## Boolean semantic family

Core keeps exactly two semantic Boolean values: canonical `true` and
`false`.

Delegation, copying, composition, freezing, or defining Boolean selector names
does not create additional Boolean values. This keeps Boolean-requiring
operations on one exact portable domain rather than introducing truthiness,
coercion, nominal membership or a user-extensible Boolean-family registry.

Classification: **KEEP**.

Confidence: **HIGH**.

## No standard Boolean prototype

D050 deliberately retained no standard `Boolean` prototype or binding.

The alternative designs are larger:

- add `Boolean -> Object` and re-parent the canonical values;
- publish duplicated protocol state separately on both canonical values; or
- introduce hidden Boolean-specific dispatch.

The current design instead keeps canonical Boolean values in the ordinary Object
topology and performs the semantic-family check only when the standard Boolean
behavior is selected.

Classification: **KEEP**.

Confidence: **HIGH**.

## Ordinary Boolean protocol

The standard protocol remains:

```text
not()
ifTrue(block)
ifFalse(block)
ifTrueIfFalse(trueBlock, falseBlock)
and(block)
or(block)
```

These are ordinary Object-hosted Closure-valued selectors. Ordinary lookup,
shadowing, extraction and user overrides remain authoritative; selecting the
standard behavior then requires the original receiver to be exact canonical
`true` or `false`.

Publishing these selectors through Object has some root-surface cost, but avoiding
that cost would require a new Boolean prototype, hidden dispatch category or
per-value protocol publication. None is smaller.

Classification: **KEEP**.

Confidence: **HIGH**.

## Negation

D050 and AUD009-A2 already retain:

```text
!a -> a.not()
```

Conformance proves that this is ordinary dispatch and can select a custom
ordinary `not` behavior.

Removing `not` while keeping `!` would require a hidden negation primitive.
Removing `!` was already rejected by A2's fixed-operator review.

Classification: **KEEP**.

Confidence: **HIGH**.

## One-way conditional selectors

`ifTrue` and `ifFalse` are heavily used across Standard Library, Package Tool,
Test Tool, examples and benchmarks.

Encoding a one-way branch using only `ifTrueIfFalse` would require a dummy
opposite Closure at each call site. That increases source and allocation/capture
surface without removing conditional semantics.

The unselected path returns canonical `null`, giving the operation one ordinary
total expression result without creating another no-value category.

Classification: **KEEP**.

Confidence: **HIGH**.

## Two-way `ifTrueIfFalse`

D050 added `ifTrueIfFalse` after explicit review.

It supplies one expression-valued two-way selection over already-evaluated
callback objects and returns the exact selected callback result.

Reconstructing the same capability from two one-way sends requires temporary
mutable result state and careful duplication of argument-evaluation rules. That
is a larger use-site model.

Classification: **KEEP**.

Confidence: **HIGH**.

## Callback evaluation and invocation

Boolean conditional methods preserve ordinary call semantics:

- callback-producing argument expressions are evaluated left-to-right;
- Closure construction does not execute the Closure body;
- only the selected callback is callability-validated;
- only the selected callback is invoked.

The selected callback uses the ordinary polymorphic invocation protocol and need
not itself be a semantic Closure.

This keeps ordinary call evaluation and conditional execution cleanly separated.

Classification: **KEEP**.

Confidence: **HIGH**.

## Strict logical protocol

For standard `and` / `or` behavior:

```text
false.and(block) -> false
true.and(block)  -> invoke block; require exact Boolean

true.or(block)   -> true
false.or(block)  -> invoke block; require exact Boolean
```

A selected callback result must be canonical `true` or `false`.

Allowing arbitrary result values would create truthiness/value-propagation
semantics and would make logical composition inconsistent with other strict
Boolean contracts.

Classification: **KEEP**.

Confidence: **HIGH**.

## Lazy `&&` / `||`

AUD009-A2 already owner-approved:

```text
a && b -> a.and(() => b)
a || b -> a.or(() => b)
```

B5 found no contradictory evidence.

The syntax remains bounded sugar over ordinary messages and Closures, not a
hidden primitive. Conformance proves custom receivers may observe and invoke the
generated RHS Closure.

Removing the operators would permanently require explicit Closure ceremony for
ordinary logical source while retaining the same underlying `and` / `or`
semantics.

Classification: **KEEP**.

Confidence: **HIGH**.

## No language-wide truthiness

Core does not assign implicit truth values to `0`, empty Strings, `null`,
arrays, maps, ordinary objects, Futures or other values.

When Core requires a Boolean decision it requires exact canonical `true` or
`false`.

This removes a cross-cutting coercion table/protocol and avoids hidden control
semantics in unrelated value families.

Custom selector behavior remains ordinary dispatch and does not establish global
truthiness.

Classification: **KEEP**.

Confidence: **HIGH**.

## Conditional syntax boundary

D051 retains no dedicated `if` / `else` grammar.

`if` and `else` remain ordinary identifiers, including ordinary
call-plus-trailing-Closure forms such as:

```protos
if: (value, block) => { block() }
if(123) { 42 }
```

B5 found no reason to add a second conditional grammar/AST/dispatch institution.

Classification: **KEEP current boundary**.

Confidence: **HIGH**.

## Standard Closure `while`

D044's standard pre-test loop is:

```text
condition.while(body)
```

with semantic Closure receiver and semantic Closure body.

Production use is broad across Standard Library, Package Tool, Test Tool,
networking, crypto, CLI, TOML and benchmarks.

Recursion is not an equivalent general loop replacement because Protos makes no
portable constant-space recursion guarantee and recursive emulation changes
activation/return-home/suspension behavior.

Collection `each` also cannot replace general condition-controlled mutable
iteration.

Classification: **KEEP**.

Confidence: **HIGH**.

## Object-hosted Closure-family behavior

Core defines no standard `Closure` prototype.

The standard `while` selector is an ordinary local Object slot whose standard
behavior checks for semantic Closure receiver/body values.

Creating a Closure prototype, materializing `while` on each Closure or using
hidden Closure-specific lookup would each introduce a larger institution.

Classification: **KEEP**.

Confidence: **HIGH**.

## Closure-only loop callbacks

The loop requires semantic Closure condition and body values.

Accepting arbitrary invokable objects would require new observable rules for
whether their `call` behavior is looked up once, pinned, re-looked-up each
iteration or allowed to change as the receiver mutates.

Closure-only repeated activation avoids those new semantics and reuses the B2
executable-value model directly.

Classification: **KEEP**.

Confidence: **HIGH**.

## Validation timing

Before the first iteration, the standard operation validates:

1. semantic Closure receiver;
2. exact one supplied argument;
3. semantic Closure body.

Declared Closure parameter binding is not preflighted. It occurs only when the
condition or body activation is actually reached.

This preserves the distinction between an invalid loop operation and an invalid
callback activation.

Classification: **KEEP**.

Confidence: **HIGH**.

## Loop decision and result

Each condition activation returns:

```text
false -> terminate
true  -> invoke body
other -> fresh Error
```

There is no truthiness, coercion, Boolean delegation test, implicit invocation,
Future awaiting or Future adoption.

Normal body results are ignored. Normal loop termination returns canonical
`null`, including the zero-iteration case.

Returning the last body result would require a separate zero-iteration result
rule and would turn the loop into a value-aggregation construct.

Classification: **KEEP**.

Confidence: **HIGH**.

## Control/concurrency boundary

`while` introduces no hidden:

- Task or Future;
- handler or cleanup frame;
- return home;
- scheduler boundary;
- cancellation mask;
- suspension/checkpoint;
- per-iteration cancellation poll.

Existing Error, non-local return, cancellation and explicit suspension semantics
propagate through the loop unchanged. Resumption must not duplicate an already
completed condition/body effect.

Detailed implementation architecture remains AUD009-C/G.

Classification: **KEEP**.

Confidence: **HIGH**.

## No dedicated `while` syntax

The normal spelling:

```protos
condition.while() {
    body
}
```

is ordinary message syntax plus the already-existing trailing-Closure grammar.

Adding a dedicated `while (...) { ... }` construct would add grammar,
reserved-word and tooling surface without removing the underlying loop semantics.

Classification: **KEEP current boundary**.

Confidence: **HIGH**.

## No primitive `for`

B5 found no Core-level requirement for a separate primitive `for`.

Existing ordinary mechanisms divide the need cleanly:

- collection/library iteration for traversal of existing aggregates/ranges;
- `Closure.while` for general condition-controlled iteration.

Detailed collection iteration breadth remains AUD009-E.

```text
primitive for = ABSENT / RETAIN ABSENCE
```

This is a retained boundary, not a removal classification.

## Strongest attempted removals

### `ifTrue` / `ifFalse`

Technically derivable from `ifTrueIfFalse`, but only by adding a dummy Closure
to the dominant one-way branch case.

Result: **KEEP**.

### `ifTrueIfFalse`

Technically derivable through two one-way sends plus mutable temporary state, but
that is more complex and less expression-oriented.

Result: **KEEP**.

### `and` / `or`

Removing these while retaining `&&` / `||` requires a hidden logical
primitive/lowering authority. Removing both selector and operator permanently
makes ordinary logical code Closure-heavy.

Result: **KEEP**.

### Object-hosted Boolean/while selectors

Replacing the current receiver-domain checks requires new Boolean/Closure
prototypes, hidden dispatch or per-value publication.

Result: **KEEP**.

### `Closure.while`

Neither recursion nor collection iteration is a semantically equivalent general
replacement.

Result: **KEEP**.

### Dedicated conditional/loop syntax

Current ordinary protocol plus trailing-Closure forms are already usable and
widely used; dedicated syntax would add an institution rather than remove one.

Result: **KEEP current boundary**.

No attempted removal produces a smaller coherent Core model.

## Reconciliation boundaries

- A2 `!`, `&&`, `||` KEEP outcomes remain unchanged.
- B1 ordinary Object publication/reflection remains unchanged.
- B2 Closure/call/non-local-return semantics remain unchanged.
- B3 Error/ensure semantics remain unchanged.
- B4 evaluation-order/Sequence semantics remain unchanged.
- D044 standard `while` protocol remains unchanged.
- D050 standard Boolean protocol completion remains unchanged.
- D051 conditional surface boundary remains unchanged.
- Future/Task/cancellation architecture remains AUD009-C.
- collection iteration breadth remains AUD009-E.
- backend/Bytecode realization remains AUD009-G.
- future syntax ergonomics remain AUD011 / explicit Dxxx territory.

## Owner approval

The project owner explicitly approved all B5 classifications.

```text
ISSUE=guillermomolina/protos#608
APPROVAL_COMMENT=5738851349
DATE=2026-09-19
```

All approved outcomes retain existing semantics. B5 itself introduces no
normative or implementation delta.

## Closure checklist

```text
CANONICAL_BOOLEAN_DOMAIN=KEEP
NO_BOOLEAN_PROTOTYPE=KEEP
BOOLEAN_OBJECT_HOSTED_PROTOCOL=KEEP
STRICT_BOOLEAN_RECEIVER_DOMAIN=KEEP

NOT_PROTOCOL=KEEP
UNARY_NOT_LOWERING=KEEP

IF_TRUE=KEEP
IF_FALSE=KEEP
IF_TRUE_IF_FALSE=KEEP
CALLBACK_EXPRESSION_EAGERNESS=KEEP
SELECTED_ONLY_CALLBACK_INVOCATION=KEEP
POLYMORPHIC_BRANCH_CALLBACK=KEEP

AND=KEEP
OR=KEEP
STRICT_LOGICAL_RESULT_BOOLEAN=KEEP
LAZY_AND_OPERATOR=KEEP
LAZY_OR_OPERATOR=KEEP
NO_TRUTHINESS=KEEP
NO_DEDICATED_IF_ELSE_SYNTAX=KEEP

CLOSURE_WHILE=KEEP
OBJECT_HOSTED_WHILE=KEEP
CLOSURE_ONLY_LOOP_CALLBACKS=KEEP
WHILE_UPFRONT_DOMAIN_VALIDATION=KEEP
WHILE_ACTIVATION_TIME_PARAMETER_BINDING=KEEP
WHILE_STRICT_BOOLEAN_DECISION=KEEP
WHILE_BODY_RESULT_IGNORED=KEEP
WHILE_NORMAL_RESULT_NULL=KEEP
NO_LOOP_SPECIFIC_SCHEDULER_CONTROL=KEEP
NO_DEDICATED_WHILE_SYNTAX=KEEP

PRIMITIVE_FOR=ABSENT_RETAIN_ABSENCE

EVERY_SCOPED_EXISTING_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
A2_BOUNDARY=PASS
B1_BOUNDARY=PASS
B2_BOUNDARY=PASS
B3_BOUNDARY=PASS
B4_BOUNDARY=PASS
D044_RECONCILIATION=PASS
D050_RECONCILIATION=PASS
D051_RECONCILIATION=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTES=NOT_REQUIRED
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B5_CLASSIFICATION=COMPLETE
```

AUD009-B5 is complete once this durable record and the required live GitHub
closure postconditions are verified.
