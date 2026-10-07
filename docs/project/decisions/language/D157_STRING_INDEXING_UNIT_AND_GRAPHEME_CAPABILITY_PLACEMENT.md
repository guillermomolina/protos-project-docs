# D157 — String indexing unit and grapheme capability placement

Status: **RATIFIED — Candidate D (scalar Core indexing plus Standard Library grapheme capability)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#618`
Trigger: AUD009-B8 / `guillermomolina/protos#617`
Protos evidence revision: `a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88`
Project-record base: `6cb1d097d25c58fa93089296d358f56bf9825c02`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D157 selects **Candidate D**.

Core String indexing and counting align with the already-retained String semantic
value model:

```text
String semantic value
    = exact sequence of Unicode scalar values

String.size()
    = number of Unicode scalar values

String.at(index)
String[index]
    = one String containing exactly the scalar at scalar index
```

The index remains a semantic Integer, zero-based, dense, and non-negative.
Out-of-bounds access signals `Error`.

A successful indexed read returns a `String`; Protos does not introduce a
separate Core `Character` family.

String remains immutable and no operation selected here normalizes, case-folds,
encodes, decodes, or otherwise rewrites its scalar sequence.

The observable consequences include:

```text
"😀".size()                 == 1
"😀"[0]                     == "😀"

"e\u{301}".size()          == 2
"e\u{301}"[0]              == "e"
"e\u{301}"[1]              == "\u{301}"

"👨‍👩‍👧‍👦".size()        == 7
```

Extended grapheme-cluster support is **preserved**, not deleted. It moves out of
the mandatory Core String indexing/counting contract and becomes an explicit
Standard Library text capability.

The Standard Library capability must preserve high-quality Unicode grapheme
segmentation and explicit Unicode-version control. D157 does not ratify a final
public spelling, prototype topology, view identity model, or implementation
boundary for that capability.

An API shape such as:

```text
Graphemes.of(text).size()
Graphemes.of(text).at(index)
Graphemes.of(text).each(...)
```

remains illustrative only.

## Approval provenance

After reviewing the full D157 comparative packet, repository evidence, candidate
set, twelve-criterion scoring, falsification, implementation consequences,
future-scenario analysis, incremental-design analysis, and the exact
invariant/delta consequences, the project owner explicitly approved:

```text
aprobada Candidate D
```

in the active project-owner interaction on 2026-09-19.

```text
SELECTED_CANDIDATE=D
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## GITHUB021 invariant consistency

Candidate D preserves the owner-approved D157 retained String invariants except
for the exact indexing/counting unit that D157 was opened to decide.

Preserved:

```text
STRING_VALUE_EXACT_UNICODE_SCALAR_SEQUENCE      PRESERVED
STRING_IMMUTABLE                                PRESERVED
NO_IMPLICIT_UNICODE_NORMALIZATION               PRESERVED
NO_SEPARATE_CHARACTER_CORE_FAMILY               PRESERVED

INDEX_DOMAIN_SEMANTIC_INTEGER                   PRESERVED
INDEX_ZERO_BASED_DENSE                          PRESERVED
NO_NEGATIVE_INDEXING                            PRESERVED
STRING_AT_RETURNS_STRING                        PRESERVED
NO_IN_PLACE_STRING_MUTATION                     PRESERVED

STRING_PLUS                                     PRESERVED
STRING_CONCAT                                   PRESERVED
NO_IMPLICIT_TEXTUAL_CONVERSION                  PRESERVED

STRING_NOT_BYTES                                PRESERVED
EXPLICIT_ENCODING_BOUNDARY                      PRESERVED

LEXER_XID_NFC_UNICODE_INFRASTRUCTURE            OUT_OF_SCOPE / PRESERVED
```

Approved observable delta:

```text
CORE_STRING_SIZE_UNIT
    Unicode-17 extended grapheme cluster
    -> Unicode scalar value

CORE_STRING_AT_UNIT
    Unicode-17 extended grapheme cluster
    -> Unicode scalar value
```

Preserved outside the Core indexing contract:

```text
EXTENDED_GRAPHEME_SEGMENTATION_CAPABILITY       PRESERVED
PLACEMENT                                        STANDARD_LIBRARY
FINAL_PUBLIC_API                                 DEFERRED
PRIVATE_IMPLEMENTATION_BOUNDARY                  DEFERRED
```

No hidden candidate refinement after owner approval changes these consequences.

## Repository evidence

At the evidence revision, standard String `size` and `at` are implemented in
`ProtosStandardStringProtocol` using ICU4J `BreakIterator` and an explicit
Unicode-17 requirement.

Existing conformance includes grapheme-specific examples such as:

```text
"e\u{301}".size() == 1
"👨‍👩‍👧‍👦".size() == 1
"e\u{301}"[0] == "e\u{301}"
```

Those tests demonstrate the current grapheme capability; they are not evidence
that grapheme segmentation must remain the universal Core String indexing unit.

Repository inspection of production Standard Library and Tool consumers found
that JSON, URI, TOML, CLI, package tooling, and test tooling commonly convert
String input explicitly to UTF-8 Bytes before indexed parsing.

The principal direct production String indexing/counting consumers found were
the IP address/endpoint modules. Their accepted syntax is ASCII, so scalar and
grapheme indexing are observably equivalent for accepted inputs.

No production caller was found whose correctness currently depends on a
multi-scalar grapheme occupying one Core String position.

## Comparative evidence

The investigation compared materially different models:

- Python: text indexing/counting follows Unicode code points, closely matching
  Protos' retained exact-scalar value model.
- Java/JVM: String indexing/counting exposes UTF-16 code units, demonstrating the
  portability cost of coupling semantics to host representation.
- JavaScript: base String indexing is UTF-16 while code-point iteration is a
  distinct layer.
- Swift: String/Character strongly embraces extended grapheme clusters, but as
  part of a broader text model that also uses Character and non-integer String
  indexes; Protos intentionally retains different invariants.
- Rust: `str` avoids integer character indexing; scalar iteration and grapheme
  segmentation are explicit higher-level operations.
- .NET: UTF-16 String, scalar `Rune`, and grapheme/text-element facilities are
  distinct layers.
- Self/prototype-family precedent: indexed String/vector behavior does not
  require UAX #29 segmentation as a universal object-model institution.
- Unicode UAX #29 / ICU: grapheme segmentation is a well-defined specialized
  text-boundary service that can remain high quality without defining the
  fundamental String value unit.

The decisive distinction is not whether grapheme segmentation is useful. It is
whether every basic Core String count/index operation should semantically depend
on a versioned text-segmentation algorithm when String itself is already defined
as an exact scalar sequence.

## Why Candidate D was selected

Candidate D keeps the smallest coherent Core model:

```text
String value unit      = scalar
String equality/identity basis = exact scalar sequence
String concatenation   = scalar-sequence concatenation
String indexing/count  = scalar
```

while preserving richer user-facing text segmentation as an explicit capability.

This aligns semantic value, composition, count, and indexed access without
forcing UAX #29 data/versioning into every basic String operation.

It also preserves the current grapheme investment. The existing ICU-backed
segmentation code and its test corpus can be relocated or wrapped behind the
Standard Library capability rather than discarded.

The design is future-compatible without future-preimplementing normalization,
collation, locale tailoring, regex, word segmentation, line segmentation, or a
general text-view hierarchy.

## Rejected candidates

### Candidate A — retain grapheme indexing in Core

Rejected because it makes a specialized, Unicode-versioned segmentation
institution mandatory for all basic String size/index operations even though
the retained String value itself is an exact scalar sequence and current
production consumers do not require grapheme-as-one-position semantics.

Swift demonstrates that a grapheme-first language model can be coherent, but
Protos does not share the accompanying Character/index/value model that makes
that choice systemic there.

### Candidate B — scalar Core indexing, no immediate grapheme replacement

Rejected because it gets the Core unit right but unnecessarily discards a
useful, already-implemented high-quality capability that can be preserved at a
more appropriate Standard Library boundary.

### Candidate C — remove direct String indexing in favor of explicit views

Rejected because it removes more Core surface than the evidence requires and
would reopen retained `String.size` / `String.at` invariants merely to avoid a
unit ambiguity that scalar indexing already resolves.

## Strongest argument against Candidate D

The strongest objection is ergonomic: extended grapheme clusters often align
better than scalar values with what a user perceives as one displayed
"character". Under Candidate D, code that wants cursor-like or user-facing text
positions must request grapheme segmentation explicitly rather than relying on
plain `String.at`.

That cost is accepted because grapheme boundaries are not identical to the
semantic value unit, are Unicode-versioned, and are not required by current
production Core String consumers. The explicit Standard Library capability
keeps the ergonomic operation available where it is actually needed.

## Future-scenario stress result

A future text-heavy Protos ecosystem may require grapheme iteration, normalization,
word/line segmentation, locale tailoring, display-column handling, or richer text
views.

Candidate D leaves a direct escape path: add ordinary Standard Library text
abstractions over the exact scalar String value without changing String identity,
equality, concatenation, encoding, or the Core index unit.

If later evidence shows that grapheme indexing must become universally implicit,
that would require reopening this semantic boundary explicitly rather than being
smuggled in through a library implementation.

## Unicode-version and implementation boundary

The current implementation uses ICU4J and checks that reported Unicode data is
at least version 17. The existing normative grapheme contract, however, requires
a controlled Unicode-17 result rather than "whatever later Unicode version the
host happens to provide".

The follow-up implementation must therefore preserve explicit grapheme-version
control at the Standard Library boundary. It may reuse ICU-backed support, use a
narrow private runtime helper, or eventually use source-backed UAX #29/data if
that proves appropriate.

D157 deliberately does not choose among those private implementation strategies.

The generated Unicode-17 XID/NFC lexer infrastructure is separate and must not be
removed or weakened as part of this work.

## Explicitly deferred questions

D157 does not select:

- the final public Standard Library module/prototype/view name for graphemes;
- whether the public abstraction is a view, iterable object, module operation,
  or another ordinary composition;
- whether that abstraction exposes exactly `size`, `at`, `each`, or another
  API;
- a general `String.each` contract;
- normalization, collation, locale-sensitive segmentation, regex, word breaking,
  or line breaking;
- the private ICU/native/source-backed implementation boundary;
- backend String storage layout or indexing caches.

Any of those that crosses the substantive design gate requires its own approved
decision.

## Normative and implementation routing

Candidate D changes observable Core String semantics, so ratification alone does
not modify the normative specification or implementation.

Follow-up reconciliation must:

1. change normative `String.size` / `String.at` semantics from Unicode-17
   extended grapheme clusters to Unicode scalar values;
2. update runtime/grammar/guide prose that currently describes grapheme indexing;
3. implement scalar counting/indexed access while preserving semantic Integer,
   bounds, receiver-family, immutability, and exact-scalar rules;
4. replace Core grapheme-specific conformance with scalar-indexing conformance;
5. preserve and relocate grapheme segmentation tests/capability rather than
   deleting them;
6. retain lexer Unicode-17 XID/NFC infrastructure unchanged; and
7. avoid selecting a concrete public Graphemes API without the required separate
   design authority.

```text
D157_STATUS=RATIFIED
SELECTED_CANDIDATE=D
CORE_STRING_INDEX_UNIT=UNICODE_SCALAR
GRAPHEME_CAPABILITY=PRESERVED_STANDARD_LIBRARY
GRAPHEME_PUBLIC_API=DEFERRED
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
