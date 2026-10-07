# AUD009-A2 — Existing expression and operator surface review

Status: **COMPLETE**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#561`

Parent audit: `guillermomolina/protos#522` — AUD009

Publication Protos revision: `e27b0e69808df10fa608595ffffff6fc80221124`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

## Purpose and boundary

AUD009-A2 reviewed the existing non-matching expression and operator surface of
Protos. It classified retained mechanisms according to the project-owner-approved
AUD009 vocabulary without silently changing language semantics.

Matching is excluded and remains owned by AUD009-A1 / D131. Missing capabilities
and ergonomic additions are excluded and remain AUD011 territory. Runtime,
concurrency, I/O, Standard Library, tooling, and AST/Bytecode architecture remain
for later AUD009 partitions.

The governing outcome/routing contract is
`AUD009_OUTCOME_AND_FOLLOWUP_ROUTING.md`.

## Final classification ledger

### Fundamental expression forms

```text
: slot creation                         KEEP
= slot modification                     KEEP
. member lookup                         KEEP
() invocation                           KEEP
parenthesized expression                KEEP
{...} object construction               KEEP
parent {...}                            KEEP
Closure construction                    KEEP
required parameters                     KEEP
this                                    KEEP
context                                 KEEP
super.member(...)                       KEEP
^ non-local return                      KEEP
```

These are retained because they directly express fundamental object, binding,
call, dispatch, activation, or control semantics rather than duplicating a
second institution. No removal route is required.

### Closure and call ergonomics

```text
single-parameter x =>                   KEEP
single-expression Closure body          KEEP
parameterless trailing Closure          KEEP
default parameters                      KEEP
rest parameters                         KEEP
call spread                             KEEP
args intrinsic                          REMOVE_NOW_RECONSIDER_LATER
```

The intended boundary is explicit:

```text
ambient universal caller vector         REMOVE
explicit variadic receive               KEEP
explicit variadic forwarding            KEEP
```

The `args` classification was routed through D145 / #574. D145 selected removal
of the ambient reserved intrinsic, with `args` becoming an ordinary identifier.
The removal was implemented by I042 / #580 and is complete.

Reconsideration trigger:

```text
Real Standard Library, Tool, or application code repeatedly needs the exact
original caller-supplied positional vector in addition to a meaningful
required/default/rest signature, especially to distinguish omitted defaulted
arguments from explicitly supplied arguments.
```

Reconsideration scope:

```text
Re-evaluate explicit caller/call-shape introspection from the then-current call
model. Do not assume the removed bare `args` spelling or fresh-Array
representation must return.
```

### Indexing

```text
receiver[index]                         KEEP
receiver[index] = value                 KEEP
```

Indexed read remains exact ordinary `at` protocol sugar. Indexed assignment
remains ordinary `atPut` dispatch plus the normal assignment-expression rule
that the expression evaluates to the exact RHS. These forms do not create a
second indexed-access or indexed-mutation protocol.

### Contextual ellipsis roles

```text
object-body ...source                   KEEP
parameter ...rest                       KEEP
call ...array                           KEEP
```

These roles are grammar-contextual and intentionally do not define one generic
ellipsis operation.

Horizontal composition was routed through D140 / #568. D140 ratified Candidate
A-prime: ordinary-object uniform structural flattening with contextual
`...source`, local reservation precedence, explicit source-source conflicts,
per-item atomicity, ordinary `without` / `alias`, and no distinct Trait value,
private trait state, or provenance institution.

### Collection-construction sugar

```text
D130 Array construction [...]           KEEP
D136 Map construction %{...}            KEEP
```

Both were recently ratified before A2 and the audit found no contradictory
evidence requiring reopening. Array construction remains ordinary-call sugar;
Map construction remains the D136 construction-time standard-Map association
model.

### Standard operators

```text
! unary                                 KEEP
- unary                                 KEEP
+ - * / %                               KEEP
< <= > >=                               KEEP
==                                      KEEP
!=                                      KEEP
=== !==                                 KEEP
&& ||                                   KEEP
fixed standard precedence               KEEP
```

The retained fixed operator surface is bounded. It is either exact ordinary
message sugar, lazy ordinary-message/Closure sugar, derived sugar over an
already-retained semantic relation, or an explicitly non-overridable identity
primitive.

D148 / #578 resolved the former normative contradiction for `!=` and ratified
Candidate A-prime:

```text
EQUALITY_CUSTOMIZATION_POINT="=="
!=_SOURCE_SURFACE=KEEP
!=_SEMANTICS=LOGICAL_COMPLEMENT_OF_VALIDATED_==
!=_ORDINARY_PROTOCOL_SELECTOR=NO
OBJECT_!=_STANDARD_SLOT=ABSENT
CUSTOM_!=_DIVERGENCE=ABSENT
===_!==_UNCHANGED=YES
MAP_HASH_EQUALITY_UNCHANGED=YES
```

The normative and implementation reconciliation was completed by I043 / #582.
This satisfies the only blocker that prevented final A2 closure.

### Arbitrary custom symbolic binary operators

```text
custom symbolic binary operators        REMOVE_NOW_RECONSIDER_LATER
custom precedence/mixing rules          REMOVE_WITH_CUSTOM_OPERATORS
```

The audit found no Standard Library or Tool production use that justified the
parallel custom-operator lexical/precedence institution. D149 / #579 ratified
Candidate A, and I044 / #583 completed removal.

The removed surface includes the generic `CUSTOM_OPERATOR` accepted-source
category, the custom-binary grammar branch, the custom precedence domain, and
the special custom/standard mixing prohibition. Fixed standard operators and
fixed standard precedence remain unchanged. Former custom spellings are invalid
source rather than newly reinterpreted stacked standard operators.

Reconsideration trigger:

```text
Real Standard Library, Tool, or application APIs repeatedly demonstrate that an
infix symbolic message materially improves readability/composability over an
ordinary named message, and the requirement cannot be met adequately by a fixed
standard operator or ordinary call syntax.
```

Reconsideration scope:

```text
Redesign symbolic/infix messaging from then-current evidence. Do not assume the
removed broad alphabet, precedence slot, mixing rule, or alias-based publication
model should return.
```

## Decision and implementation routing reconciliation

```text
D140/#568  horizontal composition                  RATIFIED/CLOSED
D145/#574  ambient Closure args intrinsic           RATIFIED/CLOSED
D148/#578  equality / inequality relationship       RATIFIED/CLOSED
D149/#579  custom symbolic binary operators         RATIFIED/CLOSED

I042/#580  remove Closure args intrinsic             CLOSED/COMPLETED
I043/#582  implement derived inequality semantics    CLOSED/COMPLETED
I044/#583  remove custom symbolic binary operators  CLOSED/COMPLETED
```

A2 itself introduced no hidden semantic change. Every substantive removal or
semantic reconciliation crossed the required Dxxx approval boundary and was
implemented by the natural owning Ixxx work item.

## Strongest KEEP cases

The strongest KEEP cases share one or more of these properties:

- they expose irreducible object/call/control semantics directly;
- they are exact, bounded sugar over one ordinary protocol;
- they are actively and broadly used in Standard Library / Tool source;
- removing them would require a new semantic institution rather than actually
  reducing one;
- they keep variability explicit at the signature or call site rather than
  ambient in every activation.

Particularly strong examples are slot creation/modification, member lookup,
Closure construction/invocation, `super`, non-local return, explicit rest/spread,
indexing, fixed standard operators, and retained horizontal composition.

## Strongest removal cases

A2 identified exactly two current surface institutions whose continuing cost was
not justified by present need:

1. ambient Closure `args`, because explicit rest/spread already covers normal
   variadic receive/forward use and no production consumer required exact
   original call-shape introspection;
2. arbitrary custom symbolic binary operators, because they duplicated ordinary
   one-argument message capability while imposing a separate lexical,
   precedence, mixing, parser, documentation, and tooling surface with no
   production use.

Both are `REMOVE_NOW_RECONSIDER_LATER`, not permanent rejection of the underlying
future problem domains.

## Reconciliation with AUD009-A1 and AUD011

A2 does not reopen matching. D131 remains the semantic authority for the
protocol-first matching model produced from AUD009-A1 evidence.

A2 also does not infer missing syntax or new capabilities from removal gaps.
Named arguments, richer suppliedness metadata, generic iterable spread, new
infix facilities, or other absent ergonomics remain outside this audit and may
only enter through AUD011 / a separately approved design route.

## Closure checklist

```text
SCOPED_INVENTORY=COMPLETE
EVIDENCE_BACKED_CLASSIFICATIONS=COMPLETE
EVERY_SCOPED_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
REMOVAL_ROUTES=PASS
D140_RECONCILIATION=PASS
D145_RECONCILIATION=PASS
D148_RECONCILIATION=PASS
D149_RECONCILIATION=PASS
I042_RECONCILIATION=PASS
I043_RECONCILIATION=PASS
I044_RECONCILIATION=PASS
MATCHING_A1_BOUNDARY=PASS
AUD011_MISSING_FEATURE_BOUNDARY=PASS
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_A2_CLOSURE=PASS
```

AUD009-A2 is therefore complete. The parent AUD009 audit should proceed to the
next bounded partition rather than continue exploring A2.
