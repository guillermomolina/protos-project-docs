# D149 — Custom symbolic binary operator surface

## Decision state and authority

This is the non-normative decision packet for `guillermomolina/protos#579`.

```text
D149_STATUS=NEEDS_USER_DECISION
TRIGGER=AUD009-A2/#561
OWNER_APPROVED_AUDIT_CLASSIFICATION=CUSTOM_SYMBOLIC_BINARY_OPERATORS_REMOVE_NOW_RECONSIDER_LATER
OWNER_APPROVED_AUDIT_CLASSIFICATION_2=CUSTOM_PRECEDENCE_AND_MIXING_RULES_REMOVE_WITH_CUSTOM_OPERATORS
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_DOCS_BASE=0bfeac7d6fa14861116aadd50d589366f310b1ae
```

This packet does not ratify a candidate. D149 remains open until the project owner
explicitly approves an exact semantic/surface choice.

## Exact problem

Current Core accepts arbitrary non-standard symbolic binary spellings from the
fixed symbolic alphabet:

```text
! $ % & * + - / < = > ? @ \ ^ | ~
```

After maximal munch, a complete spelling that is not one of the reserved/standard
symbolic tokens is emitted as `CUSTOM_OPERATOR`.

Custom symbolic operators:

- form a separate binary-expression grammar alternative;
- all have one shared precedence level;
- associate left-to-right;
- cannot mix unparenthesized with the standard binary-operator ladder;
- lower to ordinary one-argument message sends;
- have no direct ordinary source declaration syntax for their symbolic selector.

The dedicated conformance example therefore publishes a symbolic selector
structurally:

```protos
source: {
    combine: (other) => { 40 + other }
}
receiver: source.alias("combine", "@")
receiver @ 2
```

The design question is whether this arbitrary symbolic infix surface belongs in
Core v0.1 at all.

## Owner-approved AUD009 invariant

AUD009-A2 explicitly established:

```text
CUSTOM_SYMBOLIC_BINARY_OPERATORS = REMOVE_NOW_RECONSIDER_LATER
CUSTOM_PRECEDENCE_AND_MIXING_RULES = REMOVE_WITH_CUSTOM_OPERATORS
```

This is a strong removal invariant for D149, not final semantic approval.

A retaining/replacing candidate must therefore identify evidence strong enough to
reopen that invariant rather than merely showing that custom operators can be
coherent in another language.

## Current repository evidence

### Production use

AUD009-A2 and the D149 re-check find:

```text
STANDARD_LIBRARY_CUSTOM_SYMBOLIC_OPERATOR_USE=NONE_FOUND
TOOL_CUSTOM_SYMBOLIC_OPERATOR_USE=NONE_FOUND
APPLICATION_PRODUCTION_USE_IN_REPOSITORY=NONE_FOUND
```

The intentional guest-language use found is the dedicated conformance example:

```text
protos/tests/conformance/surface-sugar/custom-binary-ordinary-dispatch.protos
```

No real current Protos API depends on arbitrary custom symbolic infix syntax.

### Normative/public footprint

The feature owns public rules in:

- `spec/PROTOS_GRAMMAR.md`;
- `spec/PROTOS_LANGUAGE_SPEC.md`;
- `spec/semantics/CALLABLES.md`;
- `spec/runtime/ABSTRACT_RUNTIME.md`;
- `README.md`.

The public model includes:

- the fixed custom-operator alphabet;
- maximal-munch classification;
- `CUSTOM_OPERATOR`;
- the separate custom-binary grammar;
- one custom precedence domain;
- left associativity;
- mandatory parentheses when mixing custom and standard binary operators.

### Implementation/test footprint

Dedicated machinery includes at least:

- `TokenType.CUSTOM_OPERATOR`;
- non-standard symbolic classification in `ProtosLexer`;
- the custom-binary parser branch in `ProtosParser`;
- parser tests for custom chains and custom/standard mixing rejection;
- lexer tests for custom spellings;
- canonicalization tests proving ordinary-send lowering;
- conformance coverage for symbolic dispatch.

There is no distinct runtime dispatch institution after lowering; the cost is
primarily lexical/parser/public-language/tooling surface.

### Declaration asymmetry

Ordinary slot creation requires an identifier/member-name target. A symbolic
selector cannot be directly introduced with ordinary source such as a normal
slot declaration.

The conformance proof uses:

```protos
source.alias("combine", "@")
```

to manufacture the symbolic-named slot structurally.

This matters because the current feature is not simply "ordinary messages with
different spelling": send syntax is first-class, while declaration syntax is not.

## Comparative research

### Self — first-class binary messages

Self has unary, binary and keyword messages. Binary selectors are operator
sequences and binary slots can be declared directly. Binary messages form a
uniform message category.

Self gives all binary messages one precedence class between unary and keyword
messages. Heterogeneous binary operators require explicit grouping; identical
binary operators may associate left-to-right.

Sources:

- https://handbook.selflanguage.org/2024.1/langref.html
- https://handbook.selflanguage.org/2024.1/glossary.html

**Lesson for Protos:** arbitrary symbolic messages can fit a prototype/message
language cleanly when declaration and send are symmetric and the whole binary
message model is uniform.

However, adopting Self's uniform binary precedence would reopen the already
approved Protos fixed standard precedence ladder. Adding only declaration
symmetry while retaining Protos's separate custom domain does not obtain Self's
conceptual uniformity.

### Smalltalk — first-class binary selectors

Smalltalk also treats binary selectors as real method selectors. GNU Smalltalk's
grammar permits binary selectors constructed from a defined symbol set and gives
binary messages a single precedence class between unary and keyword messages.

Source:

- https://www.gnu.org/software/smalltalk/manual/html_node/The-syntax.html

**Lesson:** direct binary-selector declaration is a coherent complete model, but
its simplicity comes partly from not maintaining a conventional arithmetic
precedence ladder distinct from arbitrary binary selectors.

### Haskell — arbitrary infix operators plus explicit fixity

Haskell permits user-defined infix operators and named functions used infix.
Fixity declarations select left/right/non-associativity and precedence levels
0–9, and fixity participates in declaration/scoping rules.

Sources:

- https://www.haskell.org/onlinereport/decls.html
- https://www.haskell.org/tutorial/functions.html

**Lesson:** once arbitrary operators are expected to compose naturally with each
other, precedence/fixity becomes a real declaration and scope institution rather
than a small parser convenience.

This is far larger than current Protos need.

### Scala — symbolic methods with name-derived precedence

Scala permits symbolic method names and infix method application. Precedence is
derived from the operator's first character; associativity is influenced by the
last character. Scala 3 also introduces an `infix` modifier for alphanumeric
methods, while symbolic methods remain infix-capable.

Scala's own style guidance warns that nonstandard symbolic method names should be
used sparingly, mainly for genuine mathematical operators or useful DSLs.

Sources:

- https://www.scala-lang.org/files/archive/spec/2.13/06-expressions.html
- https://docs.scala-lang.org/scala3/reference/changed-features/operators.html
- https://docs.scala-lang.org/style/naming-conventions.html

**Lesson:** arbitrary symbolic methods can remain ordinary method declarations,
but parser behavior becomes coupled to selector spelling. That is not a smaller
model than removing unused operators.

### Swift — declared custom operators and precedence groups

Swift supports custom prefix/infix/postfix operators. Custom infix operators can
be assigned to precedence groups, which define precedence relationships and
associativity.

Source:

- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators/

**Lesson:** a coherent custom-operator system can make precedence explicit, but
requires declarations and a precedence-group institution. That is appropriate
when operator-oriented APIs are a real language goal, not as dormant capability.

### Kotlin — named infix functions

Kotlin does not require symbolic names for user-defined infix ergonomics. A
single-parameter member/extension function marked `infix` may be invoked
without dot/parentheses. Infix calls use fixed language-defined precedence.

Source:

- https://kotlinlang.org/docs/functions.html

**Lesson:** if the future requirement is "readable infix DSL", arbitrary symbolic
selectors are not the only design family. A future Protos requirement could
prefer ordinary named messages with opt-in infix notation.

### Elixir — fixed parser-recognized operator space

Elixir does not permit arbitrary new operator spellings. The parser recognizes a
predefined set, including some spellings not used by the core language; modules
may define/import behavior for those already-recognized operators. Precedence is
fixed by the parser.

Elixir documentation also notes that custom operators are generally discouraged
because descriptive function names are often clearer.

Source:

- https://hexdocs.pm/elixir/1.15.8/operators.html

**Lesson:** a language can preserve some DSL space without arbitrary lexical
operators, but unused predefined operator slots are still deliberate permanent
parser surface.

### Go — fixed standard operator set

Go defines a closed set of operators and fixed precedence classes. User code
cannot create new operator syntax.

Source:

- https://go.dev/ref/spec

**Lesson:** a fixed operator set is a viable long-term language design. User
extensibility can remain entirely in named functions/methods.

## Candidate construction

### Candidate A — remove arbitrary custom symbolic operators

Delete the custom operator source facility.

Retain all already-approved standard operators and their existing fixed
precedence/associativity.

Remove:

- `CUSTOM_OPERATOR` as an accepted source token;
- custom-binary grammar;
- custom precedence domain;
- custom/standard mixing rule;
- public custom-operator alphabet as an accepted extensibility surface;
- dedicated custom-operator tests/docs/examples.

Ordinary one-argument behavior remains expressible with named messages/calls.

Structural object APIs such as `alias` remain governed by their existing
decision. D149 does not redefine the universe of possible structural slot names;
it only removes arbitrary custom symbolic infix source syntax.

#### Invalid symbolic spellings after removal

Candidate A deliberately avoids a hidden expansion of prefix/operator syntax.

A non-standard maximal symbolic spelling that is invalid after removal must not
silently become a sequence of standard tokens merely because `CUSTOM_OPERATOR`
no longer exists.

For example, current custom spellings such as:

```text
@
|>
!!
^^
--
-!
!-
```

do not acquire new stacked-prefix or adjacent-standard-operator meanings through
D149.

The normative implementation may describe this as invalid/non-standard symbolic
source rather than preserving `CUSTOM_OPERATOR` as a token category, but the
observable rule is:

> removing custom operators does not reinterpret formerly-custom maximal
> symbolic spellings as new combinations of standard operators.

This keeps D149's delta limited to removal rather than accidental creation of
new syntax.

### Candidate B — retain the current model

Keep:

- broad custom symbolic alphabet;
- maximal-munch custom tokenization;
- separate custom-binary grammar;
- one custom precedence level;
- left associativity;
- prohibition on unparenthesized custom/standard mixing;
- alias-based publication where direct symbolic slot declaration is unavailable.

This is implementation-proven and semantically bounded, but current production
code does not use it.

### Candidate C — complete current custom selectors with direct declaration syntax

Retain the current custom operator domain but add direct source declaration of
symbolic one-argument selectors, eliminating the current alias asymmetry.

To avoid reopening the standard precedence invariant, this candidate initially
retains:

- one custom precedence level;
- current custom/standard mixing prohibition;
- fixed parser precedence.

This improves message/declaration symmetry but **adds** grammar rather than
removing unused grammar and still leaves two binary-operator domains.

### Candidate D — fixed spare operator set

Remove arbitrary symbolic spellings but retain a deliberately selected finite
set of currently-unused operator tokens with fixed precedence positions, similar
in architectural spirit to Elixir.

Libraries could later define behavior only for those pre-reserved spellings.

This bounds parser complexity but asks Core v0.1 to reserve concrete syntax
without a current API requiring it.

## Important rejected candidate families

### Self/Smalltalk uniform binary-message model — rejected for current D149

A truly uniform binary-message design is conceptually attractive for Protos, but
would require reconsidering the owner-approved standard precedence ladder.

Current D149 has no production need strong enough to reopen standard arithmetic,
comparison, equality and logical grouping semantics merely to preserve unused
custom operators.

### User-declared fixity / precedence groups — rejected

Haskell/Swift-style operator declarations solve custom-operator composition by
making precedence/fixity first-class source configuration.

This conflicts with D149's existing fixed-parser-precedence boundary and creates
substantial module/import/tooling interactions without current need.

### Named infix-message syntax — rejected for current D149

Kotlin-style named infix calls could provide future DSL readability without
symbolic selector names.

No current Protos API demonstrates a need for generic infix notation itself, so
adding an `infix` marker/grammar now would merely replace one unused syntax
facility with another.

It remains a valid future design family if the reconsideration trigger occurs.

## Scorecard

Scores are 1–5. They are comparison aids, not decision authority.

### A — remove arbitrary custom symbolic operators

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Preserves every retained standard operator and removes only the A2-targeted facility; invalid old custom spellings do not gain new meanings. |
| Protos alignment | 5 | HIGH | Keeps ordinary named messages as the general extension mechanism and removes a parallel unused syntax institution. |
| Present-need proportionality | 5 | HIGH | No stdlib/Tool production use pays for custom operator syntax. |
| Incremental growth | 5 | HIGH | A future real infix/DSL need can select Self-like, named-infix, fixed-spare, or declared-fixity models then. |
| Future-option resilience | 5 | HIGH | Removing current assumptions preserves more design freedom than freezing today's alphabet/precedence/mixing model. |
| Scalability | 5 | HIGH | No module/import/fixity coordination problem exists because arbitrary operator definitions are absent. |
| Conceptual simplicity | 5 | HIGH | Removes the custom token/domain/mixing rules and declaration asymmetry. |
| Portability / implementation freedom | 5 | HIGH | No runtime dependence and less parser/tooling contract. |
| Runtime / resource cost | 5 | HIGH | Runtime dispatch is unchanged; parser/lexer/tooling surface shrinks. |
| Failure / operability | 5 | HIGH | Unknown non-standard symbolic spellings fail instead of creating hidden selector behavior. |
| Deferral / reversibility / migration | 4 | HIGH | Future operator syntax is additive but may conflict with post-removal use of symbolic characters in future grammar; current production migration cost is essentially zero. |
| Evidence maturity / implementation risk | 5 | HIGH | Repository non-use is direct; fixed-operator languages and multiple alternative extensibility models provide strong precedent. |

**Overengineering red flag:** none.

**Underengineering red flag:** LOW. A future DSL could prefer custom infix notation, but
there is no evidence that deferral would require foundational object/runtime redesign.

### B — retain current model

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Current behavior is specified and tested. |
| Protos alignment | 3 | HIGH | Lowering to ordinary messages fits Protos, but declaration asymmetry and parallel parser rules do not. |
| Present-need proportionality | 1 | HIGH | No real current API uses the capability. |
| Incremental growth | 3 | MEDIUM | The model is extensible only within its fixed one-level/mixing constraints; richer composition would require redesign. |
| Future-option resilience | 2 | HIGH | Retention biases future operator design toward today's alphabet and grammar. |
| Scalability | 3 | MEDIUM | No runtime scaling issue, but library growth increases symbolic readability/collision/mixing pressure. |
| Conceptual simplicity | 2 | HIGH | Users must learn alphabet, maximal munch, custom precedence and mixing prohibition. |
| Portability / implementation freedom | 4 | HIGH | Runtime is ordinary dispatch, though all frontends/tools must preserve syntax. |
| Runtime / resource cost | 5 | HIGH | Runtime cost is negligible after lowering. |
| Failure / operability | 3 | HIGH | Mixed standard/custom expressions fail by a special domain rule; selector declaration remains indirect. |
| Deferral / reversibility / migration | 2 | HIGH | Keeping it invites external dependencies that make later removal materially harder. |
| Evidence maturity / implementation risk | 5 | HIGH | The model already works; the problem is justification, not feasibility. |

**Overengineering red flag:** HIGH because public syntax is retained without demonstrated use.

### C — direct first-class declaration for current custom selectors

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 4 | MEDIUM | Can preserve standard precedence while adding declaration symmetry, but requires new target/declaration grammar. |
| Protos alignment | 4 | MEDIUM | Makes binary selectors more ordinary/message-like, closer to Self/Smalltalk. |
| Present-need proportionality | 1 | HIGH | It adds syntax to improve an unused feature. |
| Incremental growth | 3 | MEDIUM | Still inherits the one custom precedence level and mixed-domain prohibition. |
| Future-option resilience | 2 | MEDIUM | Commits Protos further to symbolic-selector syntax before the desired future model is known. |
| Scalability | 3 | MEDIUM | More direct APIs may increase custom operator use and expose precedence/readability pressure. |
| Conceptual simplicity | 3 | MEDIUM | Declaration/send symmetry improves, but two binary precedence domains remain. |
| Portability / implementation freedom | 4 | HIGH | Runtime remains ordinary slots/messages; frontend grammar grows. |
| Runtime / resource cost | 5 | HIGH | No material runtime cost. |
| Failure / operability | 3 | MEDIUM | Better declaration diagnostics, but custom/standard composition remains a special failure rule. |
| Deferral / reversibility / migration | 2 | HIGH | New source declarations would create a larger compatibility commitment than today. |
| Evidence maturity / implementation risk | 4 | MEDIUM | Self/Smalltalk validate direct binary selectors, but not this hybrid precedence model. |

**Overengineering red flag:** HIGH. This spends more syntax to rescue a feature with no
present production requirement.

### D — fixed spare operator set

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | A finite set can coexist with fixed standard precedence if explicitly assigned. |
| Protos alignment | 3 | MEDIUM | Bounded syntax is preferable to arbitrary parser growth, but pre-reserved spellings are still privileged surface. |
| Present-need proportionality | 1 | HIGH | There is no current API from which to derive the correct spare set. |
| Incremental growth | 3 | MEDIUM | New requirements outside the reserved set force later grammar expansion. |
| Future-option resilience | 2 | HIGH | Choosing spellings/precedence now arbitrarily constrains future DSL needs. |
| Scalability | 4 | MEDIUM | Bounded parser set scales operationally, though ecosystem naming collisions remain possible. |
| Conceptual simplicity | 4 | MEDIUM | Simpler than arbitrary operators but still requires users to know unused reserved operator slots. |
| Portability / implementation freedom | 4 | HIGH | Fixed parser behavior is straightforward. |
| Runtime / resource cost | 5 | HIGH | Ordinary dispatch after parsing. |
| Failure / operability | 4 | MEDIUM | Closed syntax yields predictable errors. |
| Deferral / reversibility / migration | 3 | MEDIUM | Reserved spellings are hard to reclaim and cannot be justified from current evidence. |
| Evidence maturity / implementation risk | 4 | HIGH | Elixir demonstrates the architecture, but Protos lacks evidence selecting the right finite set. |

**Overengineering red flag:** HIGH because syntax is reserved speculatively.

## Mandatory adversarial answers

### 1. What current API is materially clearer with a custom symbolic selector?

None found in Standard Library or Tools.

The only intentional repository example exists to prove the feature itself.

### 2. Is alias-based publication evidence against the feature?

It is evidence against **the current model's completeness**.

A binary selector that is ordinary in send position but cannot be declared through
ordinary source syntax is not fully symmetric ordinary-message syntax.

That does not prove Protos should add declaration syntax; with no use, removal is
smaller.

### 3. Would direct symbolic declaration solve the conceptual problem?

It solves declaration asymmetry but not the second precedence domain, mixing ban,
symbolic alphabet, tooling cost, or lack of present need.

### 4. Does one shared custom precedence level justify itself?

No current source demonstrates that it does.

It avoids user-defined fixity complexity but achieves that by prohibiting natural
unparenthesized interaction with every standard binary group.

### 5. Would declared precedence/fixity solve mixing?

Yes, but at a far larger cost.

Haskell and Swift show that coherent arbitrary-operator composition requires
declaration/scoping/precedence machinery. Protos currently has no requirement
that justifies such an institution.

### 6. Are Tools/DSLs blocked by removal?

No current Tool or library API is blocked.

Future DSLs can use ordinary named messages now and may trigger a fresh infix
design if repeated real code demonstrates material readability/composability
benefit.

### 7. Could a future fixed operator be smaller?

Yes.

If one concrete recurring domain operation later deserves a fixed operator, that
decision can add one exact spelling/precedence without reintroducing arbitrary
user-defined symbolic syntax.

### 8. What lexical space changes?

Non-standard symbolic spellings cease to be accepted operator source.

Candidate A does not reserve the old broad custom alphabet for future use.

However, removal must not silently reinterpret old maximal custom spellings as
new compositions of standard operators.

### 9. What migration exists today?

Repository migration is confined to dedicated custom-operator specification,
tests, README/explanatory material, and implementation machinery.

No Standard Library or Tool behavior needs source migration.

External users of arbitrary custom operators would incur a source compatibility
break and must replace those sends with ordinary named messages or other current
source.

### 10. Can custom operators return later without dormant hooks?

Yes.

All credible future designs are frontend additions over ordinary message
semantics. No runtime object-model reservation is needed now.

### 11. Strongest case for retaining current model

Protos is message-oriented, and symbolic one-argument messages are a natural
historical capability in Self/Smalltalk-like languages. The current lowering is
already ordinary dispatch, and the model is implemented/tested.

The objection is substantial but insufficient: Protos has deliberately retained
a conventional fixed standard precedence ladder rather than Self/Smalltalk's
uniform binary-message model, direct symbolic declaration is absent, and real
Protos production source does not use the arbitrary extension surface.

Historical implementation cost is sunk cost.

### 12. Strongest future-loss concern

Removing the feature means future source may legitimately allocate currently
unused symbolic characters to other grammar.

Therefore reintroducing exactly today's broad operator alphabet may later be
incompatible.

That is intentional under REMOVE_NOW_RECONSIDER_LATER: the future need should
compete with whatever the language has become, rather than reserving syntax
forever without present use.

## Smallest-sufficient analysis

Current Core needs:

- the already-approved fixed standard operators;
- ordinary named message/call extension;
- fixed parser precedence for the standard operator surface.

It does not currently need:

- arbitrary user-selected symbolic infix names;
- a second binary precedence domain;
- custom/standard mixing restrictions;
- direct symbolic-selector declaration syntax;
- fixity declarations;
- spare future operator tokens;
- generic named infix notation.

Candidate A is therefore the smallest sufficient current language.

## Deferral / recovery path

If Candidate A is selected and the reconsideration trigger later fires, compare
the then-current design space from first principles.

Plausible future designs include:

- one or more exact fixed operators for demonstrated domains;
- Self/Smalltalk-like first-class binary selectors if the standard precedence
  model is explicitly reopened;
- Kotlin-like named infix messages;
- Elixir-like finite parser-recognized operator slots;
- Haskell/Swift-like explicit fixity declarations if real DSL requirements justify
  their module/parser cost.

None requires retaining `CUSTOM_OPERATOR`, the current custom precedence slot,
or the current broad alphabet today.

## GITHUB021 invariant / delta check

Applicable owner-approved invariants:

```text
CUSTOM_SYMBOLIC_BINARY_OPERATORS = REMOVE_NOW_RECONSIDER_LATER
CUSTOM_PRECEDENCE_AND_MIXING_RULES = REMOVE_WITH_CUSTOM_OPERATORS
STANDARD_OPERATOR_SURFACE = KEEP
FIXED_STANDARD_PRECEDENCE = KEEP
```

Candidate A:

```text
PRESERVES_CUSTOM_OPERATOR_REMOVAL_INVARIANT=YES
PRESERVES_STANDARD_OPERATOR_SURFACE=YES
PRESERVES_FIXED_STANDARD_PRECEDENCE=YES
NEW_OPERATOR_DECLARATION_INSTITUTION=NO
NEW_FIXITY_INSTITUTION=NO
NEW_NAMED_INFIX_INSTITUTION=NO
NEW_SPARE_OPERATOR_RESERVATION=NO
HIDDEN_REOPENING=NO
```

Candidate B contradicts the owner-approved removal classification and would
require explicit reopening.

Candidates C/D preserve some form of the rejected capability and therefore also
require explicit reopening plus independent present-need justification.

The rejected Self/Smalltalk-uniform and user-fixity families additionally reopen
the approved standard-precedence boundary.

## Proposal pending owner approval

**Proposed candidate: A — remove arbitrary custom symbolic binary operators.**

Exact proposed decision:

```text
D149_CANDIDATE=A
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

RECONSIDER_TRIGGER=REAL_REPEATED_STDLIB_TOOL_OR_APPLICATION_INFIX_NEED
```

The reconsideration trigger remains:

> Real Standard Library, Tool, or application APIs repeatedly demonstrate that an
> infix symbolic message materially improves readability/composability over an
> ordinary named message, and the requirement cannot be met adequately by a fixed
> standard operator or ordinary call syntax.

At that point, redesign from the then-current language rather than restoring the
old broad alphabet, one-level precedence model, custom/standard mixing ban, or
alias-based declaration route by default.
