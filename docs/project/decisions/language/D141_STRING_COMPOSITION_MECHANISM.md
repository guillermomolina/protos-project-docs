# D141 — Strict finite String aggregate composition

Status: **RATIFIED — Candidate M1**

Approval date: **2026-09-18**  
Decision issue: `guillermomolina/protos#569`  
Trigger: `AUD011` / `guillermomolina/protos#540`  
Normative Protos revision: `b504b486a81ab880a51924a6eb736cba7ca2d51b`  
Specification revision: `0.1.424`

This is durable non-normative rationale. Observable semantics are authoritative
under `guillermomolina/protos:spec/**` at the exact Protos revision above.

## Selected model

D141 selects one additional ordinary standard String composition message:

```protos
"a".concat()
"a".concat("b")
"a".concat("b", "c", "d")
"a".concat(...parts)
```

The selected operation is **strict finite aggregate String composition**.

It does not select interpolation, syntactic sugar, implicit textual conversion,
formatting, a StringBuilder-like public abstraction, or collection-oriented
joining.

## Approval provenance

The project owner explicitly approved **D141-M1** after the complete decision
packet, comparative research, scoring, anti-overengineering analysis,
future-scenario stress, strongest counterargument, and invariant/delta review.

The owning Issue records the exact approval and the post-approval consistency
check. Earlier interpolation-oriented D141 proposals were explicitly withdrawn
or suspended before this final candidate was selected.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
SELECTED_CANDIDATE=D141-M1
```

## Exact semantic contract

The normative specification at Protos revision
`b504b486a81ab880a51924a6eb736cba7ca2d51b` owns the exact observable contract.

The selected model requires:

1. canonical `String` owns an ordinary standard selector named `concat`;
2. selecting that standard behavior does not make canonical `String` itself a
   semantic String receiver;
3. the original receiver must be a semantic String value;
4. the operation accepts zero or more supplied positional values after ordinary
   spread expansion;
5. every supplied value must be a semantic String;
6. no Number, Boolean, `null`, Bytes value, arbitrary object, prototype, or
   String-delegating non-String value is converted;
7. ordinary call receiver/argument/spread evaluation remains unchanged and
   completes before the standard method validates its supplied values;
8. once selected standard `concat` begins, receiver and arguments are validated
   without invoking user behavior;
9. invalid standard input signals ordinary `Error`, exposes no partial String
   result, and mutates no input;
10. success produces exactly the receiver's Unicode-scalar sequence followed by
    every supplied String's scalar sequence in positional order;
11. standard `concat` performs no normalization, encoding/decoding, locale
    operation, equality/hash dispatch, callback, mutation, or hidden suspension;
12. zero supplied arguments preserve the receiver's semantic String value, so
    `s.concat() === s` under existing String value identity;
13. standard `concat` is not semantically defined as repeated `+` dispatch;
14. existing standard String `+` remains unchanged and binary;
15. `+` and `concat` are independent ordinary selectors;
16. for standard valid binary inputs, `a.concat(b)` and standard `a + b` produce
    the same String semantic value, but this is result equivalence rather than a
    lowering or dispatch-equivalence rule; and
17. internal representation and construction strategy remain implementation
    choices when observable semantics are preserved.

## Evaluation distinction from repeated `+`

One variadic ordinary call is intentionally not equivalent to nested binary
sends.

For:

```protos
(a + bad()) + later()
```

the first standard binary `+` can fail before `later()` is evaluated.

For:

```protos
a.concat(bad(), later())
```

ordinary call semantics evaluate the supplied argument expressions before the
selected standard `concat` body validates them. Therefore `later()` may already
have executed when `concat` detects the invalid String argument.

This distinction is part of the selected model and is why `concat` is not
specified as hidden repeated `+` dispatch.

## Zero-arity and spread

Zero arguments are deliberately valid:

```protos
prefix.concat(...parts)
```

remains well-defined when `parts` expands to zero elements. The result then has
exactly the receiver's scalar sequence.

No separate empty-case branch or special spread rule is introduced.

## Relationship to existing String semantics

D141 preserves:

```text
STRING_EXACT_SCALAR_SEQUENCE_IDENTITY=PRESERVED
STRING_IMMUTABILITY=PRESERVED
STRING_FAMILY_MEMBERSHIP_NOT_CONFERRED_BY_DELEGATION=PRESERVED
STANDARD_STRING_PLUS_STRICT_BINARY=PRESERVED
STANDARD_STRING_PLUS_NO_IMPLICIT_CONVERSION=PRESERVED
STRING_LITERAL_SEMANTICS=PRESERVED
ADJACENT_LITERAL_CONCATENATION=NOT_INTRODUCED
UNIVERSAL_TEXTUAL_CONVERSION=NOT_INTRODUCED
```

The selected operation is one additional ordinary standard selector, not a new
String value kind or privileged source form.

## Comparative result

The final comparison covered materially different approaches in:

- Self — ordinary binary message composition;
- Smalltalk/Pharo — binary message composition plus separate stream construction;
- Racket — strict variadic `string-append`;
- Python — strict collection `join` plus separate mutable `StringIO`;
- Rust — collection/slice `concat` and `join`;
- Java — binary `String.concat`, separate `StringBuilder`, implementation fusion;
- C#/.NET — multi-String `String.Concat` plus object-converting overloads;
- JavaScript — receiver-oriented variadic `concat` with coercion;
- Erlang/Elixir — structured iodata/chardata fragments for I/O-scale workloads.

The comparison showed three relevant larger alternatives:

- **M0:** retain binary `+` only;
- **M2:** make aggregate composition collection-oriented;
- **M3:** introduce builder/stream/fragment-tree state.

M0 remains the strongest minimal counterargument because every finite result is
already expressible and deferring `concat` would not force a foundational
rewrite.

M2 and M3 remain future-additive but were not selected because their
collection/iteration or state/lifetime surface exceeds the present finite
positional-composition requirement.

## Anti-overengineering result

D141 deliberately does **not** preimplement:

- syntactic sugar;
- interpolation;
- textual conversion;
- formatting or localization;
- escaping domains;
- collection joining;
- public mutable builders;
- ropes or fragment trees as public semantic objects;
- streaming/unbounded text construction.

Those remain separate decisions if evidence later justifies them.

The permanent cost accepted by M1 is one additional documented standard String
selector and the need to teach and maintain its evaluation distinction from
nested binary `+`.

## Future stress and escape path

The selected operation adds no shared mutable state, scheduler rule, lock,
authority object, distributed identity, persistence format, host-encoding
contract, or Truffle-specific semantic dependency.

A plausible future requirement that could outgrow M1 is large
incremental/streaming text construction where eager final String materialization
is itself the wrong abstraction.

The escape path is a separate builder/stream/iodata-like facility with explicit
materialization or direct I/O consumption. Such a facility can coexist with M1
without redefining String identity, `+`, or `concat`.

## Explicit deferrals

D141 does not decide:

```text
SYNTACTIC_SUGAR=DEFERRED
INTERPOLATION=DEFERRED
TEXTUAL_CONVERSION=DEFERRED
FORMAT_SPECIFIERS=DEFERRED
COLLECTION_JOIN_CONCAT=DEFERRED
PUBLIC_STRING_BUILDER_OR_IODATA=DEFERRED
```

In particular, no later sugar is pre-approved to lower to `concat`. A later
decision must compare its own evaluation and failure semantics.

## Normative publication

The selected semantics were published in `guillermomolina/protos` as
specification revision `0.1.424`.

Exact product revision:

```text
PROTOS_REVISION=b504b486a81ab880a51924a6eb736cba7ca2d51b
```

The bounded published delta changed:

- `spec/semantics/VALUES_AND_COLLECTIONS.md`;
- `spec/runtime/ABSTRACT_RUNTIME.md`; and
- `spec/PROTOS_SPEC_CHANGELOG.md`.

The runtime document remains informative pseudocode; observable ownership remains
with the normative semantic specification.

## Closure state

This durable record captures the selected D141 language decision. Executable
implementation remains separately routed work and is not part of D141
ratification.

```text
D141_STATUS=RATIFIED
SELECTED_CANDIDATE=M1
SPECIFICATION_REVISION=0.1.424
PROTOS_REVISION=b504b486a81ab880a51924a6eb736cba7ca2d51b
SPECIFICATION_CHANGED=YES
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
SYNTACTIC_SUGAR=DEFERRED
TEXTUAL_CONVERSION=DEFERRED
```
