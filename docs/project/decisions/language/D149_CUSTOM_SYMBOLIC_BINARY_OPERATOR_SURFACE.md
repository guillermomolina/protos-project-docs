# D149 — Custom symbolic binary operator surface

## Decision

D149 ratifies **Candidate A — remove arbitrary custom symbolic binary operators**.

```text
D149_STATUS=RATIFIED
SELECTED_CANDIDATE=A
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_RECORD_BASE=d305e6a62f08954bc11c89e40f5418eb824c478b
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Owner approval

The project owner explicitly approved Candidate A in the active D149 interaction:

```text
ok pues entonces apruebo
```

The approval refers to the exact candidate in the corrected decision packet:

```text
docs/project/work/D149/D149_CUSTOM_SYMBOLIC_BINARY_OPERATOR_DECISION_PACKET.md
PACKET_REVISION=d305e6a62f08954bc11c89e40f5418eb824c478b
```

## Ratified outcome

```text
CUSTOM_SYMBOLIC_BINARY_OPERATORS=REMOVE
CUSTOM_OPERATOR_TOKEN_AS_ACCEPTED_SOURCE=REMOVE
CUSTOM_BINARY_GRAMMAR=REMOVE
CUSTOM_PRECEDENCE_DOMAIN=REMOVE
CUSTOM_STANDARD_MIXING_RULE=REMOVE

STANDARD_OPERATORS=UNCHANGED
STANDARD_OPERATOR_PRECEDENCE=UNCHANGED
D148_INEQUALITY_DECISION=UNAFFECTED

ORDINARY_NAMED_MESSAGES=UNCHANGED
OBJECT_ALIAS_PROTOCOL=UNCHANGED

DIRECT_SYMBOLIC_SLOT_DECLARATION=NOT_ADDED
NAMED_INFIX_MESSAGES=NOT_ADDED
FIXITY_DECLARATIONS=NOT_ADDED
SPARE_CUSTOM_OPERATOR_SET=NOT_RESERVED

FORMER_CUSTOM_SYMBOLIC_SPELLINGS=INVALID_SOURCE
FORMER_CUSTOM_SPELLINGS_REINTERPRETED_AS_STACKED_STANDARD_TOKENS=NO

OLD_CUSTOM_OPERATOR_ALPHABET_RESERVED_FOR_FUTURE=NO
OLD_CUSTOM_PRECEDENCE_SLOT_RESERVED_FOR_FUTURE=NO
```

D149 removes the current arbitrary custom infix source facility. It does not
remove or alter the fixed standard operator surface.

## Scope boundary

The decision removes the language surface that currently accepts arbitrary
non-standard symbolic spellings through `CUSTOM_OPERATOR`.

It also removes the special grammar and precedence rules that exist solely for
that surface.

The decision does **not**:

- remove standard `+ - * / % < <= > >= == != === !== && || !`;
- change the standard precedence ladder;
- change ordinary one-argument message dispatch;
- change structural `Object.alias`;
- introduce direct symbolic selector declarations;
- introduce user-declared fixity or precedence;
- introduce named infix messages;
- reserve a finite spare operator set.

## Lexical compatibility boundary

Removal must not accidentally create new syntax by splitting formerly-custom
maximal spellings into sequences of standard operators.

Former custom spellings such as:

```text
@
|>
!!
^^
--
-!
!-
```

become invalid source unless some separate current grammar already assigns the
exact spelling another meaning.

D149 does not make forms like `!!x`, `^^x`, `--x`, `-!x`, or `!-x`
valid stacked-prefix syntax.

## Rationale

Current repository evidence found no Standard Library or Tool production use of
arbitrary custom symbolic operators.

The dedicated guest-language use found is the conformance example proving the
feature itself:

```text
protos/tests/conformance/surface-sugar/custom-binary-ordinary-dispatch.protos
```

The feature nevertheless imposes a broad public/frontend contract:

- custom symbolic alphabet;
- maximal-munch classification;
- `CUSTOM_OPERATOR`;
- separate custom-binary grammar;
- one custom precedence domain;
- custom/standard mixing prohibition;
- dedicated parser/lexer/tooling/test obligations.

Its runtime lowering is ordinary message dispatch, so no special object/runtime
architecture must be preserved to support a possible future infix design.

## Comparative result

The decision packet compared Self, Smalltalk, Haskell, Scala, Swift, Kotlin,
Elixir, and Go.

The comparison showed several coherent operator-extension families:

- Self/Smalltalk-style directly-declarable binary selectors;
- Haskell/Swift-style declared fixity/precedence;
- Scala-style spelling-derived precedence;
- Kotlin-style named infix functions;
- Elixir-style finite parser-recognized operator space;
- closed fixed-operator languages such as Go.

Current Protos production evidence does not justify selecting any of those
larger institutions now.

## GITHUB021 consistency

AUD009-A2 established:

```text
CUSTOM_SYMBOLIC_BINARY_OPERATORS = REMOVE_NOW_RECONSIDER_LATER
CUSTOM_PRECEDENCE_AND_MIXING_RULES = REMOVE_WITH_CUSTOM_OPERATORS
STANDARD_OPERATOR_SURFACE = KEEP
FIXED_STANDARD_PRECEDENCE = KEEP
```

Candidate A preserves those invariants.

```text
PRESERVES_OWNER_INVARIANTS=YES
NEW_OPERATOR_DECLARATION_INSTITUTION=NO
NEW_FIXITY_INSTITUTION=NO
NEW_NAMED_INFIX_INSTITUTION=NO
NEW_SPARE_OPERATOR_RESERVATION=NO
HIDDEN_REOPENING=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Reconsideration trigger

This is **REMOVE_NOW / RECONSIDER_LATER**, not permanent prohibition.

Reconsider custom/infix extensibility if real Standard Library, Tool, or
application APIs repeatedly demonstrate that an infix message materially
improves readability/composability over an ordinary named message and the need
cannot be met adequately by a fixed standard operator or ordinary call syntax.

At that point, redesign from the then-current language. Do not assume the old
broad alphabet, single custom precedence level, mixing prohibition, or
alias-based publication route should return.

The future design may instead choose a fixed operator, named infix messages,
directly declarable binary selectors, a finite operator set, or explicit
fixity/precedence if evidence supports it.

## Deferred implementation

This record selects semantics only.

Normative specification, lexer/parser/tooling/test cleanup and implementation
versioning are separate implementation work and must preserve the ratified
boundary above.
