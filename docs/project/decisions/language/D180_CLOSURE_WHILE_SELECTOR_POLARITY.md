# D180 — Closure while selector polarity and Boolean-control symmetry

Status: **RATIFIED — Candidate B**

Approval date: **2026-10-01**  
Decision issue: `guillermomolina/protos#762`  
Research evidence revision: `0a5115caddba8ebb7bb4275ce32441ab90938d3d`  
Closure Protos revision: `d8dcc95d34088e942e737b98c7ad42a81a977293`  
Project-record base: `e11a676a61a4655f5d0d2774d6665b6b00daaa7a`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D180 selects **Candidate B — rename the standard Closure pre-test loop selector
from `while(body)` to `whileTrue(body)`**.

The selected boundary is:

```text
STANDARD CLOSURE PRE-TEST LOOP

old standard selector              while(body)
selected standard selector         whileTrue(body)

loop receiver                      semantic Closure
body                               semantic Closure
condition decision                 exact canonical Boolean
condition true                     invoke body, then repeat
condition false                    terminate
other condition result             Error
normal loop result                 canonical null

whileFalse(body)                   NOT_ADDED
while alias                        NOT_RETAINED
transitional while alias           NOT_ADDED_BY_DEFAULT
dedicated while syntax             NOT_ADDED
truthiness                         NOT_ADDED
selector intrinsic/sealing         NOT_ADDED
ordinary lookup/shadowing          KEEP
ordinary extraction/override       KEEP
D044 non-name semantics            KEEP
D051 syntax boundary               KEEP
```

Candidate B is a **surface rename only**. It does not change why the loop
receiver is a re-evaluable Closure, and it does not turn `whileTrue` into a
privileged Boolean operation.

## Current semantic invariant preserved from D044

The existing loop model remains authoritative except for the standard selector
name:

```text
conditionClosure.whileTrue(bodyClosure)

condition() -> canonical true
    => body()
    => repeat

condition() -> canonical false
    => terminate and return canonical null

condition() -> any other normal result
    => fresh Error
```

The selected rename preserves the existing D044 behavior for evaluation order,
receiver/body validation, zero-argument callback invocation, ignored normal body
result, Error propagation, non-local return, suspension/cancellation composition,
Future handling, absence of truthiness, and implementation freedom.

No new loop execution semantics are selected by D180.

## Why the selector changes

The current loop contract is already specifically a **true-polarity** loop. The
generic name `while` hides that polarity while the retained Boolean protocol
uses explicit names such as:

```text
ifTrue
ifFalse
ifTrueIfFalse
```

Candidate B makes the existing polarity visible without increasing the number of
standard loop selectors.

This is naming symmetry, not receiver-domain symmetry:

```text
canonicalBoolean.ifTrue(body)
conditionClosure.whileTrue(body)
```

The first receiver is an already evaluated Boolean. The second is a Closure
because the condition must be re-evaluated on every iteration. D180 does not
collapse those receiver models.

## Historical reconstruction

The investigation found that D044 deliberately selected a single ordinary
Closure-hosted pre-test loop and compared the overall control model with several
languages, including Self and Smalltalk-family systems.

However, the durable D044 rationale did **not** preserve evidence of an explicit
selector-level comparison between:

```text
while
whileTrue
whileFalse
```

The historical verdict for that narrower comparison is therefore:

```text
D044_SELECTOR_POLARITY_COMPARISON=NOT_EVIDENCED
```

This is not evidence that the original spelling was accidental. It means only
that the surviving D044 record does not establish that this exact naming choice
was separately evaluated.

D050 is materially different: its retained history explicitly records
comparison of Boolean selection against Smalltalk/Self and alternative selector
spellings.

AUD009-B5 / #608 retained the existence and complexity of the current Boolean
control and Closure loop mechanisms, including `Closure.while`; it did not
settle the later D180 selector-polarity question.

## Comparative evidence

The D180 investigation compared materially different models.

### Smalltalk / Pharo / Squeak

The Smalltalk family places loop control on an executable block and exposes
explicit polarity through `whileTrue:` and `whileFalse:`, while Boolean
selection uses `ifTrue:` / `ifFalse:`.

This is strong precedent for explicit polarity on a re-evaluable condition
object, but it does not require Protos to copy the complete Smalltalk protocol.

### Self

Self similarly provides block-hosted `whileTrue:` / `whileFalse:` and
Boolean conditional selection. This is especially relevant because both control
forms remain object/message oriented.

Again, precedent supports the coherence of `whileTrue`; it does not establish
that Protos needs both loop polarities now.

### Io

Io demonstrates that a message/prototype-oriented language can coherently retain
a generic `while` surface even while exposing conditional messages such as
`ifTrue` / `ifFalse`.

Io is therefore evidence that Candidate A was viable, not evidence that generic
`while` is uniquely correct. Its condition evaluation and truthiness model also
differs materially from Protos.

### Syntax-oriented languages

Conventional syntax such as:

```text
while (condition) { ... }
```

shows that implicit true-polarity is widely readable. It is weaker evidence for
Protos selector design because such languages do not expose the loop as an
ordinary selector with lookup, reflection, extraction, shadowing and override.

### Truffle/runtime evidence

Truffle loop facilities optimize semantic loop structure rather than prescribing
guest-language selector spelling.

Existing Protos prepared/structured dispatch likewise performs ordinary
selection before guarded canonical classification. A rename from `while` to
`whileTrue` therefore has no semantic optimization privilege by itself.

```text
selector spelling == "whileTrue"
    != bypass ordinary lookup
    != intrinsic
    != sealed behavior
```

## Repository usage evidence

At the research revision
`0a5115caddba8ebb7bb4275ce32441ab90938d3d`, the exact `.while(` inventory
found:

```text
exact .while( occurrences                     243
files containing exact .while(                 43

executable Protos direct .while( sends        209
executable Protos files                         31

protos/lib occurrences                         129
tools occurrences                               37
benchmarks occurrences                           9
while conformance occurrences                   27
other executable Protos test occurrences         7
guide/spec/changelog occurrences                13
Java tests/fixtures occurrences                 20
native script occurrences                        1
```

This is a substantial migration surface inside the repository, but it is a
mechanical selector rename rather than a semantic loop rewrite.

The research did not establish that there are no external consumers. Core v0.1
is still pre-release/draft, so D180 chooses the clean standard surface now rather
than permanently duplicating public spellings merely to speculate about external
compatibility.

The one commit between the research revision and the closure Protos revision
`d8dcc95d34088e942e737b98c7ad42a81a977293` changes only the Java slow-test
guard surface (`Makefile`, `tools/java_slow_test_guard.py`, and
`tools/java_slow_tests_allowlist.txt`). It does not change D180 semantics,
selectors, normative control documents, or the usage inventory that motivates
the decision.

## Candidate result

### Candidate A — keep `Closure.while(body)`

Rejected.

This is the strongest alternative. It is concise, conventional, requires no
migration, and already has a complete strict semantic contract.

It was not selected because Protos already exposes the true polarity
semantically, and making that polarity explicit improves conceptual consistency
with the retained control vocabulary without adding another standard selector.

### Candidate B — rename to `Closure.whileTrue(body)`

**Selected.**

It preserves one standard loop selector, preserves every non-name D044 semantic,
makes the existing true polarity explicit, and leaves a clean future extension
point if a real use case later justifies `whileFalse`.

### Candidate C — standardize both `whileTrue(body)` and `whileFalse(body)`

Rejected for now.

The pair is coherent and has strong Smalltalk/Self precedent, but `whileFalse`
adds a new public operation with no demonstrated present requirement.

A strict `whileFalse` would also need to preserve Protos canonical-Boolean
validation directly; it cannot be specified merely as arbitrary
`!condition()` rewriting because custom non-Boolean objects may define
`not()`.

### Candidate D — retain aliases or stage migration through aliases

Rejected as the default design.

Permanent aliases would create duplicate public spellings with reflection,
documentation, tooling, override and compatibility consequences.

A temporary alias remains a possible migration technique only if concrete
external compatibility evidence later justifies it. D180 does not install that
mechanism speculatively.

## Pay-for-what-you-need and growth result

Candidate B has the same steady-state selector count as Candidate A:

```text
A: while
B: whileTrue
```

It therefore does not pre-build an inverse loop protocol.

Future growth remains incremental:

```text
current selected surface
    -> whileTrue

future demonstrated inverse-loop requirement
    -> independently consider adding whileFalse
```

Adding `whileFalse` later is additive. There is no need to reserve or
preimplement it now.

## Strongest argument against Candidate B

The strongest objection is migration cost with little behavioral payoff.

Generic `while` is familiar, short, and already used heavily in the current
repository. `whileTrue` does not enable a capability that `while` lacks, and
the Boolean/Closure receiver distinction means the visual symmetry with
`ifTrue` is not a complete type/protocol symmetry.

D180 accepts that cost because the project is still in the draft Core v0.1 phase,
Candidate B keeps the same one-selector surface, and delaying the rename would
increase compatibility cost without producing a better semantic distinction.

## Invariant and boundary consistency

The selected candidate preserves the applicable established boundaries:

```text
D044 loop execution semantics                 PRESERVED
D044 re-evaluable Closure receiver            PRESERVED
D044 strict canonical Boolean decision        PRESERVED
D050 Boolean control protocol                 PRESERVED
D051 no dedicated if/else control syntax      PRESERVED
no dedicated while grammar                    PRESERVED
ordinary message lookup                       PRESERVED
ordinary shadowing/extraction/override         PRESERVED
truthiness absence                            PRESERVED
Closure invocation/capture semantics          PRESERVED
PLAT042 backend architecture boundary         NOT_REOPENED
PLAT043 Boolean-control ownership             NOT_REOPENED
```

The candidate introduces no hidden contradiction with those authorities.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Implementation consequence

D180 ratification changes the selected language design but does **not** claim
that the current product revision has already implemented it.

A follow-up implementation slice must reconcile the normative specification,
runtime, conformance/tests, shipped Protos source, documentation and tooling with
the selected standard selector.

That implementation must:

1. replace the standard `while(body)` selector with `whileTrue(body)`;
2. preserve all non-name D044 semantics exactly;
3. remove `while` as the standard loop selector rather than retaining an alias;
4. not add `whileFalse`;
5. not add a dedicated grammar keyword or statement;
6. preserve ordinary lookup, reflection, extraction, shadowing and override;
7. update canonical standard-loop selection/optimization guards without using
   selector spelling as authority;
8. migrate repository-owned executable `.while(` call sites and relevant
   fixtures/documentation;
9. update normative control documentation and specification changelog under the
   normal specification-version process; and
10. keep PLAT042/PLAT043 backend ownership decisions unchanged except for the
    mechanically necessary selector rename.

Until that follow-up is published and validated, current `protos/main` still
implements the pre-D180 `while` spelling.

```text
D180_STATUS=RATIFIED
SELECTED_CANDIDATE=B

STANDARD_LOOP_SELECTOR=whileTrue
OLD_STANDARD_LOOP_SELECTOR=REMOVE
WHILE_ALIAS=NOT_RETAINED
WHILE_FALSE=NOT_ADDED

D044_NON_NAME_SEMANTICS=KEEP
ORDINARY_LOOKUP=KEEP
DEDICATED_WHILE_SYNTAX=NOT_ADDED
TRUTHINESS=NOT_ADDED
SELECTOR_INTRINSIC=NOT_ADDED

NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Approval provenance

The decision packet recommended the exact Candidate B boundary: replace the
standard `while(body)` selector with `whileTrue(body)`, preserve all D044
non-name semantics, retain only one standard loop selector, add no
`whileFalse`, retain no alias by default, add no dedicated syntax, and preserve
ordinary-message semantics.

The project owner then reviewed the summarized candidate set in the active
interaction on 2026-10-01 and explicitly selected:

```text
Elijo B.
```

That selection is the approval source for D180 ratification.
