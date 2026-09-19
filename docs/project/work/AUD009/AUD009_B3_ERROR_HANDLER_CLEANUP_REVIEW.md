# AUD009-B3 — Error, handler, and protected-cleanup complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#605`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence / closure revision: `98997d173ef482210dd726a900ec75d23f02ce7b`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#605`, issue comment
`5738655131`, 2026-09-19.

## Purpose and boundary

AUD009-B3 reviewed the existing Core Error-object, signaling, failure-identity,
dynamic-handler and protected-cleanup model under the retrospective
complexity/necessity methodology approved for AUD009.

B3 does not reopen Closure/non-local-return semantics closed by B2, concurrency
architecture owned by AUD009-C, I/O lifecycle semantics owned by AUD009-D,
Standard Library API selection owned by AUD009-E, or backend/C-prime dynamic
control representation owned by AUD009-G.

## Final classification ledger

```text
Error ordinary-object model                       KEEP
Error.signal exact-receiver protocol              KEEP
fresh standard failure occurrences                KEEP
exact re-signaling / recorded identity            KEEP
non-resumable signaling                           KEEP
standard fail()                                   KEEP

shallow portable taxonomy policy                  KEEP
SlotNotFound                                      KEEP
InvalidSuper                                      KEEP
InvalidReturn                                     KEEP

Error.handle dynamic installation                 KEEP
delegation-based handler matching                 KEEP
dynamically innermost selection                   KEEP
Closure-only body/handler                         KEEP
selected-handler early deactivation               KEEP

Closure.ensure(cleanup)                           KEEP
exactly-once/LIFO cleanup                         KEEP
suspension-not-exit invariant                     KEEP
later-transfer supersession                       KEEP
no implicit composite/suppressed Error            KEEP
```

No B3 mechanism is classified for removal. No B3 Dxxx route is required.

## Errors remain ordinary Protos objects

Core defines one language-level failure family rooted at the ordinary
`Error` prototype.

Error categories are ordinary delegation prototypes. They are not classes,
checked declarations, tags, or a second hidden exception hierarchy.

This keeps handler matching, identity, reflection and user-defined error
families inside the ordinary B1 object model.

Classification: **KEEP**.

Confidence: **HIGH**.

## Standard `Error.signal()`

The standard zero-argument `signal()` behavior signals its exact Error receiver.

It does not clone, wrap, coerce or enrich the receiver. The exact same Error is
the value observed by handler matching and by the selected handler.

Production Standard Library and Tool code uses explicit signaling and exact
re-signaling of caught failures.

Replacing this ordinary message with another privileged throw authority would
add a second control surface without eliminating the Error object model.

Classification: **KEEP**.

Confidence: **HIGH**.

## Fresh standard failure occurrences and exact re-signaling

D005's identity model remains justified.

```text
new normative failure occurrence -> fresh ordinary Error object
already identified Error e        -> propagate/re-signal exact e
standard Error prototype          -> category/protocol object, not singleton failure
```

Production Package Tool code catches an Error, performs resource or diagnostic
work, then invokes `error.signal()`. Replacing that Error with a fresh
occurrence would lose observable identity.

Singleton standard failures would collapse independent occurrences under
ordinary `===`, identity hashing, storage, Future failure observation and
handler behavior.

Runtime cost is small: a fresh occurrence is an ordinary object whose parent is
the selected standard Error prototype. Allocation may still be optimized when
identity cannot become observable.

Classification: **KEEP**.

Confidence: **HIGH**.

## Non-resumable signaling

Core signaling abandons the signaling continuation.

A matching handler may determine the result of the enclosing `handle`
operation, but it cannot resume, retry, restart, or supply a value back into the
abandoned signaling point.

This is the smaller control model. Adding resumable conditions/restarts would
require a separate continuation/recovery contract spanning effects, dynamic
extent, suspension, cancellation and isolation.

Classification: **KEEP**.

Confidence: **HIGH**.

## Standard generic `fail()`

D146 already re-evaluated and ratified this convenience as one ordinary
zero-argument prelude callable.

A valid invocation creates one fresh generic Error occurrence with canonical
standard `Error` parent and immediately signals that exact object.

It adds no syntax, keyword, intrinsic, payload, cause, current-error,
custom-subtype shorthand or rethrow mode. Local `fail` bindings remain ordinary
shadowing.

Current production use spans URI/networking, CSV/JSON/TOML, CLI, Package Tool and
Test Tool source.

Classification: **KEEP**.

Confidence: **HIGH**.

## Shallow portable Error taxonomy

The retained taxonomy policy remains deliberately narrow:

- one mandatory root `Error`;
- named standard categories only where a normative contract promises a portable
  distinction;
- no automatic category for every index, arity, mutation, filesystem or backend
  failure;
- user/library code may create ordinary deeper Error prototype hierarchies.

B3 classifies only Core-owned categories:

```text
SlotNotFound   KEEP
InvalidSuper   KEEP
InvalidReturn  KEEP
```

`SlotNotFound` represents lookup exhaustion.

`InvalidSuper` represents absence of the method-home metadata required to
define a super lookup origin; that is observably different from a valid super
lookup that exhausts with `SlotNotFound`.

`InvalidReturn` represents an attempted non-local return to a completed return
home; it preserves the B2 home-activation model without reinterpreting the
operation as a local return.

These categories add ordinary prototype bindings and occurrence-parent
selection, not special control machinery.

Concurrency- and I/O-owned categories listed in `ERRORS.md` are outside B3 and
remain for AUD009-C/D.

### Standard Library reuse observation

Sequential `std:collections/Array` currently explicitly signals
`InvalidPredicateResult`, `InvalidComparatorResult`, and
`InvalidComparatorOrder`, while `ERRORS.md` identifies
`PARALLEL_EXECUTION.md` as their normative trigger owner.

Because these are ordinary Error prototypes, this observation does not by itself
invalidate Core Error semantics. Whether the sequential Standard Library should
reuse those names is intentionally deferred to AUD009-E.

Classification: **KEEP** for the shallow taxonomy policy and B3 Core-owned
categories.

Confidence: **HIGH**.

## Dynamic `Error.handle(body, handler)`

The current handler protocol is directly used by Package Tool,
`std:test/Assertions`, and I/O support.

A handler receiver is an Error prototype/object. Matching walks the exact
signaled Error followed by ordinary delegation parents.

Therefore exact-identity matching and category matching use one authority:

```text
signaled Error identity
then ordinary delegation ancestry
```

The dynamically innermost matching installation wins.

The selected handler receives the exact signaled Error object.

Classification: **KEEP**.

Confidence: **HIGH**.

## Closure-only handler boundary

D034 requires semantic Closure values for both protected body and handler.

This remains a simplifying restriction. Accepting an arbitrary merely-invokable
object would require defining whether a `call` lookup is validated before
installation, pinned, repeated after installation, or itself subject to the new
handler.

Closure-only invocation uses the single executable value kind retained by B2
and avoids that additional dynamic-boundary lookup contract.

Classification: **KEEP**.

Confidence: **HIGH**.

## Selected-handler deactivation before crossed cleanup

When a matching handler is selected, that frame becomes inactive before unwind
executes crossed `ensure` cleanup.

This prevents cleanup failures from being accidentally caught by the same
one-shot destination whose protected body is already being abandoned.

If cleanup completes normally, the original Error continues to the selected
handler. If cleanup emits a later escaping control transfer, that transfer
supersedes the original route.

Classification: **KEEP**.

Confidence: **HIGH**.

## Ordinary Closure `ensure(cleanup)`

D043's protected-cleanup model remains justified and has direct production use
in `std:io/Files`, where resource acquisition may already have committed even
when failure or cancellation wins at an observation boundary.

The retained Core capability is:

```text
protected body is a Closure
cleanup is a Closure
cleanup runs exactly once on semantic scope exit
nested cleanup is LIFO
normal cleanup preserves exact body result / pending transfer
physical suspension is not semantic scope exit
```

Manual “close after the body” cannot cover Error unwind and B2 non-local return.

A dedicated `finally` syntax would add another syntax/control institution
without removing the underlying cleanup semantics.

Classification: **KEEP**.

Confidence: **HIGH**.

## Later-transfer supersession

One rule covers all protected exits:

```text
pending exit + cleanup completes normally
    -> original pending exit continues

cleanup emits later escaping transfer
    -> later transfer supersedes original pending exit
```

Core does not expose a dual-transfer object, suppressed-error list,
cause-wrapping protocol, or composite Error automatically.

This is a small deterministic rule even though runtime/backend code must retain
the pending transfer while cleanup executes.

Classification: **KEEP**.

Confidence: **HIGH**.

## Runtime and concurrency boundary

Handler and ensure installations are task-local dynamic control state.

B3 retains only the semantic invariants needed by Core, including that physical
suspension/replay is not semantic scope exit.

Detailed Future/Task cancellation and structured execution consequences remain
AUD009-C.

PLAT021/AUD009-G own how C-prime/Bytecode runtime machinery transports and
represents dynamic control. Backend duplication is not counted as evidence for
removing semantically observable Error/cleanup behavior.

## Strongest attempted removals

### Core-specific Error categories

Collapsing `SlotNotFound`, `InvalidSuper`, and `InvalidReturn` into generic
`Error` would save only a few ordinary prototype bindings while deleting
portable distinctions already used by semantics, conformance and tooling.

Result: **KEEP**.

### `fail()`

Removing it eliminates only a small ordinary convenience recently justified by
broad production repetition; it does not simplify signaling itself.

Result: **KEEP**.

### Closure-only `handle`

Generalizing it to arbitrary callable objects introduces lookup/pinning timing
questions inside a dynamic control boundary.

Result: **KEEP**.

### Handler deactivation ordering

Removing the early deactivation rule permits one selected handler to catch an
Error generated while unwinding toward itself, requiring another special rule to
recover deterministic behavior.

Result: **KEEP**.

### `ensure`

Ordinary post-body cleanup is not equivalent across Error and non-local-return
exits, while dedicated `finally` syntax would be a larger institution.

Result: **KEEP**.

## Reconciliation boundaries

- AUD009-B2 non-local-return semantics remain unchanged.
- D005 identity/taxonomy invariants remain unchanged.
- D034 Closure-specific `Error.handle` remains unchanged.
- D043 `ensure` semantics remain unchanged.
- D146 `fail()` remains unchanged.
- concurrency-specific Error categories/cancellation remain AUD009-C.
- I/O categories/lifecycle semantics remain AUD009-D.
- sequential Standard Library category reuse remains AUD009-E.
- dynamic-control backend representation remains AUD009-G.

## Owner approval

The project owner explicitly approved all B3 classifications.

```text
ISSUE=guillermomolina/protos#605
APPROVAL_COMMENT=5738655131
DATE=2026-09-19
```

All approved outcomes retain existing semantics. B3 itself introduces no
normative or implementation delta.

## Closure checklist

```text
ERROR_OBJECT_MODEL=KEEP
ERROR_SIGNAL_EXACT_RECEIVER=KEEP
FRESH_FAILURE_OCCURRENCES=KEEP
EXACT_RESIGNAL_IDENTITY=KEEP
NON_RESUMABLE_SIGNALING=KEEP
STANDARD_FAIL=KEEP
SHALLOW_TAXONOMY=KEEP
SLOT_NOT_FOUND=KEEP
INVALID_SUPER=KEEP
INVALID_RETURN=KEEP
ERROR_HANDLE=KEEP
DELEGATION_MATCHING=KEEP
INNERMOST_HANDLER_SELECTION=KEEP
CLOSURE_ONLY_HANDLER_BOUNDARY=KEEP
SELECTED_HANDLER_EARLY_DEACTIVATION=KEEP
ENSURE=KEEP
EXACTLY_ONCE_LIFO_CLEANUP=KEEP
SUSPENSION_NOT_EXIT=KEEP
LATER_TRANSFER_SUPERSESSION=KEEP
NO_IMPLICIT_COMPOSITE_ERROR=KEEP
EVERY_SCOPED_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
B2_BOUNDARY=PASS
AUD009_C_BOUNDARY=PASS
AUD009_D_BOUNDARY=PASS
AUD009_E_BOUNDARY=PASS
AUD009_G_BOUNDARY=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTES=NOT_REQUIRED
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B3_CLASSIFICATION=COMPLETE
```

AUD009-B3 is complete once this durable record and the required live GitHub
closure postconditions are verified. The parent AUD009 audit may then advance to
the next bounded Core-semantics slice.
