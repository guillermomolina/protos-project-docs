# D145 — Closure caller-argument introspection and `args` intrinsic

## Decision

D145 ratifies **Candidate A — remove the ambient reserved `args` intrinsic**.

```text
D145_STATUS=RATIFIED
SELECTED_CANDIDATE=A
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_RECORD_BASE=973d57c0c06ec2aa32b3b8efe8731b50d490513c
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This record is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Owner approval

The project owner explicitly approved the exact Candidate A in the active D145
interaction:

```text
ok pues apruebo A
```

The approved candidate is the exact proposal published in the D145 decision packet:

```text
docs/project/work/D145/D145_CALLER_ARGUMENT_INTROSPECTION_DECISION_PACKET.md
```

## Ratified semantics

The following outcome is selected:

```text
ARGS_INTRINSIC=REMOVE
ARGS_IDENTIFIER_AFTER_REMOVAL=ORDINARY_IDENTIFIER

REQUIRED_PARAMETERS=UNCHANGED
DEFAULT_PARAMETERS=UNCHANGED
REST_PARAMETERS=UNCHANGED
CALL_SPREAD=UNCHANGED

PROCESS_ARGS_API=UNCHANGED

DEBUGGER_INTERNAL_ARGUMENT_VISIBILITY=NOT_GUEST_SEMANTICS
CALLER_SUPPLIED_VECTOR_INTERNAL_BINDING_NEED=UNCHANGED

EXPLICIT_CALL_METADATA=NOT_ADDED
PARAMETER_SUPPLIEDNESS=NOT_ADDED
FULL_VECTOR_CAPTURE_MARKER=NOT_ADDED

OLD_ARGS_SPELLING_RESERVED_FOR_FUTURE=NO
```

Bare `args` therefore ceases to be an intrinsic pseudo-identifier.

After the normative and implementation change is applied, `args` is an ordinary
identifier and may be used according to the normal identifier, parameter, slot, and
lookup rules.

This decision does not remove or change `process.args()`. That ordinary Process API
continues to expose application/process command-line arguments according to its own
specification.

## Capability intentionally removed

The removed capability is ambient access from every Closure invocation to the complete
flattened caller-supplied positional vector as a fresh frozen standard Array.

In particular, after removal a Closure such as:

```protos
f: (x = 42) => {
    ...
}
```

does not have a built-in guest-language facility that necessarily distinguishes:

```text
f()
f(42)
```

when the relevant bound parameter value is otherwise the same.

Explicit rest parameters remain available for variadic capture and forwarding, but they
are not claimed to reproduce this omitted-default versus explicitly-supplied distinction.

## Internal/runtime boundary

The caller-supplied positional vector remains part of the invocation/binding model as
needed to implement:

- arity checking;
- required parameter binding;
- default suppression/evaluation;
- rest capture;
- spread-flattened positional invocation.

D145 removes the requirement that this vector also exist as a universally guest-visible
fresh `args` Array.

Implementations remain free to retain, reshape, lazily materialize, or otherwise represent
internal argument data as long as the normative call semantics are preserved.

Debugger or runtime tooling may retain internal argument visibility. Such observability is
not guest-language `args` semantics and does not reserve the source identifier.

## Rationale

Repository evidence at the decision revision found no Standard Library or Tool use of the
bare `args` intrinsic.

Non-test bare use was confined to tutorial/example material demonstrating the feature.
Other Tool hits named `args` were principally `process.args()`, which is a separate API.

The one unique capability beyond ordinary parameters/rest is exact original call-shape
introspection alongside named/defaulted signatures. No current production requirement
justifies exposing that capability ambiently in every Closure.

Candidate A is therefore the smallest sufficient design:

- forwarding remains explicit through rest + spread;
- defaults remain explicit in the signature;
- no universal call-metadata institution is installed;
- no new suppliedness state is added to parameters;
- no new signature syntax is introduced merely to preserve an unused capability;
- the common identifier `args` stops being globally reserved.

## Comparative result

The decision packet compared Self, Smalltalk/Pharo, JavaScript, Python, Ruby, Lua, and Io.

JavaScript's `arguments` is the strongest precedent for retaining ambient original-call
data. Io provides even richer per-call metadata, but in support of a materially different
message/evaluation model.

Self, Ruby, and Lua emphasize declared argument/vararg mechanisms. Python separates
explicit `*args` from signature/binding reflection. Pharo can expose execution-context
information reflectively without making a fresh argument Array the ordinary call surface.

The prior art therefore establishes that ambient caller-vector introspection is coherent,
but not that Protos needs to pay for it without a present use case.

## GITHUB021 invariant consistency

AUD009-A2 established the owner-approved invariant:

```text
ARGS_INTRINSIC = REMOVE_NOW_RECONSIDER_LATER
```

D145 Candidate A preserves that invariant exactly.

```text
PRESERVES_OWNER_INVARIANT=YES
HIDDEN_REOPENING=NO
NEW_CALL_METADATA_INSTITUTION=NO
NEW_PARAMETER_SUPPLIEDNESS_INSTITUTION=NO
NEW_FULL_VECTOR_CAPTURE_SYNTAX=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Reconsideration trigger

Reconsider caller-shape introspection only if real Tool, Standard Library, or application
code repeatedly needs the exact original supplied positional vector in addition to a
meaningful named/default/rest signature, especially to distinguish omitted defaulted
arguments from explicitly supplied ones.

If that trigger occurs, evaluate the then-current design space. Do not assume the old
ambient `args` intrinsic or its fresh-Array representation should return.

Because `args` becomes an ordinary identifier after removal, this decision deliberately
does not reserve the old spelling for future reintroduction.

## Deferred implementation and normative publication

This ratification record selects semantics only.

The following remain separate repository work:

- update the normative specification owners to remove the `args` intrinsic and describe
  `args` as an ordinary identifier;
- remove lexer/parser/canonical/runtime special handling;
- reconcile conformance, Java tests, tutorials, examples, and README text;
- preserve `process.args()`;
- preserve parameter/default/rest/spread behavior;
- apply normal implementation versioning and specification changelog requirements.

No implementation or normative `guillermomolina/protos` repository change is made by this
decision-record publication.
