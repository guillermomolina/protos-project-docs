# AUD009-B8 — String, Bytes, and text/binary boundary complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#617`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`

Closure revalidation revision: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`

No Protos product-repository content changed between the B8 evidence baseline and
the closure revalidation revision.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#617`, issue comment
`5739144584`, 2026-09-19.

Derived decision routes:

- `D157 / guillermomolina/protos#618 — String indexing unit and grapheme capability placement`
- `D158 / guillermomolina/protos#619 — Multiline String literal model`

## Purpose and boundary

AUD009-B8 reviewed the standard String/Bytes value model and the Core
text-versus-binary boundary under the retrospective complexity/necessity
methodology.

B8 is an audit/classification slice only. It does not itself alter normative
String semantics, grammar, encoding policy, runtime behavior, public protocols,
or implementation.

The owner-approved classification retains the String/Bytes value boundary and
almost all current behavior, while routing two separate institutions for later
decision:

1. mandatory Unicode-17 extended-grapheme indexing/counting in Core String; and
2. the current triple-double multiline literal plus its structural indentation
   normalization contract.

## Final classification ledger

```text
String exact Unicode-scalar semantic value          KEEP
String immutability                                 KEEP
no implicit Unicode normalization                   KEEP
no Character family                                 KEEP current boundary

String.size / String.at                             KEEP
zero-based exact-Integer indexing                   KEEP
no String indexed mutation                          KEEP

strict binary String +                              KEEP
String.concat                                       KEEP
no implicit textual conversion                      KEEP

single-quoted String literal                        KEEP
double-quoted String literal                        KEEP
no Character literal                               KEEP current boundary
no interpolation                                    KEEP current boundary

String != Bytes                                     KEEP
explicit Encoding conversion boundary               KEEP

Bytes semantic family                               KEEP
Bytes not mandatory Core-prelude binding            KEEP
Bytes octets = Integer 0..255                       KEEP
fresh Bytes identity                                KEEP
Bytes open/closed/frozen integration                KEEP
Bytes size/at/atPut                                 KEEP
Bytes add                                           KEEP
Bytes removeAt                                      KEEP
Bytes each snapshot                                 KEEP
no implicit Bytes text interpretation               KEEP

Unicode-17 extended-grapheme clusters as mandatory
Core String.size/String.at unit
    REMOVE_NOW_RECONSIDER_LATER

current triple-double multiline String literal plus
structural indentation-normalization contract
    REMOVE_NOW_RECONSIDER_LATER
```

## String semantic-value analysis

The retained String model is deliberately abstract and representation-independent:

```text
String semantic value = exact sequence of Unicode scalar values
```

This keeps UTF-8, UTF-16, JVM `String` layout, or another backend encoding out
of semantic identity.

String remains immutable. No implicit Unicode normalization is introduced.
Canonically equivalent but scalar-distinct strings therefore remain distinct
unless an explicit future text operation says otherwise.

A one-scalar String remains an ordinary String; B8 found no justification for a
separate Character/Char semantic family.

These properties are all **KEEP**.

## Grapheme/indexing analysis

The current String semantic value is a Unicode-scalar sequence, but
`String.size` and `String.at` currently project a second structure:
Unicode-17 default extended grapheme clusters.

The current implementation makes that structure concrete through ICU4J
`BreakIterator` in `ProtosStandardStringProtocol` and checks for Unicode
17.0.0 data before installing the String protocol.

That creates continuing Core obligations for:

- Unicode-version-sensitive grapheme segmentation;
- UAX #29 behavior in a fundamental indexing API;
- ICU-backed segmentation work for basic String size/index operations;
- tests and compatibility consequences when Unicode grapheme rules evolve;
- a semantic mismatch between the retained scalar-sequence value model and the
  default indexing unit.

Repository evidence found no production consumer that requires a multi-scalar
grapheme to be one `String.at` position. Direct indexing found in production
network/parsing code is ASCII-constrained, while several other parsers operate
over Bytes explicitly.

The approved B8 classification is therefore:

```text
Unicode-17 extended-grapheme cluster
as mandatory Core String.size/String.at unit
    REMOVE_NOW_RECONSIDER_LATER
```

This does **not** mean that grapheme capability itself is rejected.

### Grapheme relocation refinement

The owner explicitly approved the following boundary:

- removal applies to the mandatory Core indexing unit;
- existing grapheme capability must not be discarded merely because Core
  indexing changes;
- Standard Library relocation/preservation is a mandatory first-class candidate
  in D157;
- existing ICU/Unicode-17 implementation and tests should be reused when the
  selected architecture permits it rather than deleted and later recreated;
- lexer/grammar Unicode XID/NFC infrastructure is a separate concern and is not
  part of this removal.

The generated `UnicodeData17`, `UnicodeXid`, and `UnicodeNfc17` machinery
serves lexical identifier/XID/NFC semantics. B8 does not classify that
infrastructure for removal.

D157 must compare at least:

```text
A. retain current Core grapheme indexing
B. scalar indexing/count in Core
C. remove direct String indexing in favor of explicit views
D. scalar Core indexing plus explicit Standard Library grapheme capability
```

Candidate D is mandatory and first-class.

B8 does not select the final D157 candidate, view/module spelling, native support
boundary, or implementation strategy.

## String composition analysis

The retained composition surface is intentionally strict:

```text
String + String
String.concat(String...)
```

No implicit textual conversion is added. Concatenation does not normalize
Unicode text.

D141/I047 remains authoritative for `String.concat`; B8 found no contradictory
evidence requiring that decision to be reopened.

Classification: **KEEP**.

D150 continues to own any separate future String-composition syntax question.

## String literal/value analysis

Single-quoted and double-quoted String literals remain justified and map directly
to the retained String value model.

B8 retains:

- single-quoted String literals;
- double-quoted String literals;
- no Character literal;
- no interpolation as a current Core/language institution.

The current triple-double multiline literal is different. Its contract includes
a substantial structural source transformation institution, including:

- a third delimiter;
- multiline newline handling;
- opening/trailing newline removal;
- closing-delimiter-defined indentation;
- exact SPACE/TAB prefix matching;
- blank-line handling;
- lexical failure on invalid prefixes;
- indentation processing before escape processing;
- CR/LF/CRLF treatment.

Repository search found no production guest `.protos` source using this
language-level triple-double multiline form. Relevant matches were specification,
lexer/parser/conformance, or other data-format concerns rather than demonstrated
ordinary Protos source need.

The approved classification is therefore:

```text
triple-double multiline String literal
plus structural indentation normalization
    REMOVE_NOW_RECONSIDER_LATER
```

D158 owns the replacement decision and must compare at least:

```text
A. retain current model
B. remove multiline literal syntax for now
C. simpler triple-quoted multiline syntax without automatic dedent
D. credible raw/heredoc/explicit-dedent alternatives
```

B8 does not select any D158 candidate.

## String / Bytes / Encoding boundary analysis

The retained model keeps three concepts separate:

```text
String   abstract Unicode text
Bytes    raw octet sequence
Encoding explicit conversion/codec boundary
```

Neither String nor Bytes is an implicit encoding of the other.

This keeps text identity independent of UTF-8/UTF-16 and keeps binary data free
from hidden textual interpretation.

Detailed codec catalogue, malformed-input policy, streaming behavior, and
TextReader/TextWriter policy remain owned by the later AUD009-D I/O audit.

Classification: **KEEP**.

## Bytes semantic-object and Core-placement analysis

B8 attempted the strongest removal case: move Bytes entirely out of Core value
semantics and make it only an I/O/Standard-Library detail.

The evidence did not support that removal.

Bytes is a broadly useful ordinary byte-sequence value consumed across encoding,
file/network adapters, parsers, cryptographic/package-content operations, and
other binary boundaries.

The retained Bytes model is:

- ordinary identity-bearing object;
- receiver-owned mutable byte-sequence state while open;
- fresh identity for Bytes-producing operations;
- octets represented as ordinary semantic Integer values restricted to
  `0..255`;
- no UInt8 requirement;
- zero-based dense indexing;
- no holes and no negative indexing;
- `size`, `at`, `atPut`, `add`, `removeAt`, and `each`;
- shallow snapshot iteration for `each`;
- ordinary open/closed/frozen mutation integration;
- no implicit text interpretation.

The current architecture already distinguishes semantic ownership from prelude
surface: Bytes remains a Core-owned semantic family while not being required as
a direct Core-prelude binding.

Classification: **KEEP**.

## Strongest attempted removals

### Remove String indexing entirely

Rejected at B8 classification level. Current production use and ordinary
sequence ergonomics still justify `String.size` / `String.at` as a concept.

The unit of those operations is routed to D157.

### Remove grapheme capability entirely

Rejected as an interpretation of B8.

The audit removes only the requirement that grapheme clusters be the mandatory
Core indexing unit. Grapheme capability remains available for relocation.

### Remove String `+` / `concat`

Rejected. Their strict String-only semantics are compact, compositional, and
already justified.

### Remove Bytes semantic family

Rejected. Binary consumers justify an ordinary byte-sequence value independent
of text encoding.

### Remove `Bytes.removeAt`

Rejected. It completes the retained mutable sequence surface and does not create
a separate semantic institution.

### Remove current multiline literal institution

Survives the audit and is routed to D158 because present production need does not
justify its structural grammar/indentation contract.

## Derived decision routing

### D157 / #618

`D157 — String indexing unit and grapheme capability placement`

State at B8 closure: **NEEDS_USER_DECISION**.

D157 is unresolved. It must run the full current Dxxx comparative
research/scoring/falsification process.

Mandatory B8 invariant:

```text
STANDARD_LIBRARY_GRAPHEME_RELOCATION_CANDIDATE=FIRST_CLASS
GRAPHEME_CAPABILITY_REJECTED=NO
LEXER_XID_NFC_INFRASTRUCTURE_IN_SCOPE=NO
```

### D158 / #619

`D158 — Multiline String literal model`

State at B8 closure: **NEEDS_USER_DECISION**.

D158 is unresolved. It must independently evaluate the multiline literal design
space and obtain exact owner approval before grammar/specification or
implementation changes occur.

The D157 and D158 decisions are intentionally separate.

## Boundary handoffs

- **AUD009-D** owns detailed I/O/Encoding/TextReader/TextWriter policy.
- **AUD009-E** owns Standard Library text breadth such as normalization,
  collation, locale, regex, and other optional text processing where applicable.
- **AUD009-G** owns backend String/Bytes representation and runtime architecture.
- **D141/I047** remains authoritative for `String.concat`.
- **D150** remains authoritative for future String-composition syntax and is not
  reopened by B8.
- **D156** remains the separate fixed-width Integer placement decision.
- B8 does not authorize implementation cleanup for either D157 or D158.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#617
CHECKPOINT_COMMENT=5739085972
APPROVAL_COMMENT=5739144584
DATE=2026-09-19

GRAPHEME_CORE_INDEX_UNIT=REMOVE_NOW_RECONSIDER_LATER
GRAPHEME_CAPABILITY_REJECTED=NO
STANDARD_LIBRARY_GRAPHEME_RELOCATION_CANDIDATE=MANDATORY
GRAPHEME_DECISION=D157/#618
GRAPHEME_DECISION_STATE=NEEDS_USER_DECISION

TRIPLE_DOUBLE_MULTILINE_INSTITUTION=REMOVE_NOW_RECONSIDER_LATER
MULTILINE_DECISION=D158/#619
MULTILINE_DECISION_STATE=NEEDS_USER_DECISION

NORMATIVE_CHANGE_AUTHORIZED_BY_B8=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_B8=NO
```

## Closure checklist

```text
STRING_SCALAR_VALUE=KEEP
STRING_IMMUTABLE=KEEP
NO_IMPLICIT_NORMALIZATION=KEEP
NO_CHARACTER_FAMILY=KEEP

STRING_SIZE_AT_CONCEPT=KEEP
ZERO_BASED_INTEGER_INDEXING=KEEP
NO_STRING_INDEXED_MUTATION=KEEP

STRING_PLUS=KEEP
STRING_CONCAT=KEEP
NO_IMPLICIT_TEXT_CONVERSION=KEEP

SINGLE_QUOTED_STRING=KEEP
DOUBLE_QUOTED_STRING=KEEP
NO_CHARACTER_LITERAL=KEEP
NO_INTERPOLATION=KEEP

STRING_BYTES_SEPARATION=KEEP
EXPLICIT_ENCODING_BOUNDARY=KEEP

BYTES_FAMILY=KEEP
BYTES_NOT_REQUIRED_PRELUDE_BINDING=KEEP
BYTES_INTEGER_OCTETS=KEEP
BYTES_IDENTITY_MUTABILITY=KEEP
BYTES_SIZE_AT_ATPUT_ADD_REMOVEAT_EACH=KEEP

CORE_GRAPHEME_INDEX_UNIT=REMOVE_NOW_RECONSIDER_LATER
GRAPHEME_CAPABILITY_RETAINABLE_OUTSIDE_CORE=YES
STANDARD_LIBRARY_RELOCATION_MANDATORY_D157_CANDIDATE=PASS

CURRENT_TRIPLE_DOUBLE_MULTILINE_LITERAL=REMOVE_NOW_RECONSIDER_LATER
CURRENT_STRUCTURAL_INDENT_NORMALIZATION=REMOVE_NOW_RECONSIDER_LATER

OWNER_APPROVAL_PROVENANCE=PASS
GRAPHEME_ROUTE=D157/#618
MULTILINE_ROUTE=D158/#619
REMOVALS_IMPLEMENTED_BY_AUDIT=NO
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B8_CLASSIFICATION=COMPLETE
```

AUD009-B8 is complete once this durable record and the required live GitHub
closure postconditions are verified.
