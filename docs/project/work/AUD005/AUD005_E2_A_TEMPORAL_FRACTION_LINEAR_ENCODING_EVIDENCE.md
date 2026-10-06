# AUD005 / LIB010-E2-A — linear temporal fraction encoding evidence

## Status

```text
WORK_ITEM=AUD005
ISSUE=guillermomolina/protos#451
TARGET=LIB010
TARGET_ISSUE=guillermomolina/protos#418
SLICE=LIB010-E2-A
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PRODUCT_REVISION=62f3f5710210f247aad8574d3d0d56d3254bfd70
PRODUCT_REVISION_SUBJECT=LIB010-E2-A: make TOML temporal fraction encoding linear
IMPLEMENTATION_VERSION=0.3.230-SNAPSHOT
F6_TEMPORAL_FRACTION_SCALING=RESOLVED
FRACTION_TEXT_COMPLEXITY=LINEAR_IN_EMITTED_LENGTH
SPECIFICATION_CHANGED=NO
PUBLIC_API_CHANGED=NO
TOML_SEMANTICS_CHANGED=NO
ENCODED_SPELLING_CHANGED=NO
D104_CHANGED=NO
D109_CHANGED=NO
D087_CHANGED=NO
F4_CHANGED=NO
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This is durable non-normative implementation evidence for AUD005 finding F6.
It records the published LIB010-E2-A repair and does not define TOML or Protos
Standard Library semantics.

## Published implementation

Exact Protos product revision:

```text
62f3f5710210f247aad8574d3d0d56d3254bfd70
LIB010-E2-A: make TOML temporal fraction encoding linear
```

The commit changes exactly:

```text
CHANGELOG.md
pom.xml
protos/lib/toml/TOML.protos
protos/tests/conformance/library/toml/encoder-positive.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTomlClosureConformanceTest.java
```

The implementation version advances to:

```text
0.3.230-SNAPSHOT
```

No specification file is changed.

## F6 defect and repair

Before E2-A, `fractionText` encoded a retained temporal fraction by obtaining
the coefficient text and repeatedly prepending one immutable String zero until
the requested decimal width was reached:

```text
raw = "0" + raw
```

For a fraction requiring P leading zeroes, that pattern repeatedly copies a
growing String. The cumulative copied prefix is proportional to
`1 + 2 + ... + P`, so the padding path is quadratic in the number of leading
zeroes.

E2-A replaces that path with one mutable UTF-8 octet buffer:

1. encode the coefficient text once to octets;
2. initialize the output buffer with `.`;
3. append exactly the required number of ASCII zero octets;
4. append each coefficient octet exactly once; and
5. decode the completed buffer once.

The relevant published shape is:

```text
raw: Encoding.UTF8.encode(integerText(coefficient))
buffer: Encoding.UTF8.encode(".")
padding: digits - raw.size()
while padding > 0:
    buffer.add(48)
raw.each:
    buffer.add(octet)
result = Encoding.UTF8.decode(buffer)
```

## Complexity proof

Let:

- `R` be the number of decimal digits in the coefficient;
- `D` be the retained TOML fraction width;
- `P = D - R` be the required leading-zero count when positive.

The new path performs:

```text
coefficient conversion / UTF-8 encoding: O(R)
zero emission:                         O(P)
coefficient octet append:              O(R)
single final UTF-8 decode:             O(D + 1)
```

Since `P + R = D` for a padded fraction, the complete fraction-text path is
linear in the emitted length:

```text
O(R) + O(P) + O(R) + O(D) = O(D)
```

No growing immutable String prefix is recreated per padding digit.

Final F6 classification:

```text
REPEATED_IMMUTABLE_PREFIX_PREPEND=REMOVED
ZERO_PADDING_EMISSION=ONE_APPEND_PER_ZERO
COEFFICIENT_OCTETS=ONE_APPEND_PER_OCTET
FINAL_DECODE_COUNT=1
FRACTION_TEXT_COMPLEXITY=O_EMITTED_LENGTH
F6_STATUS=RESOLVED
```

## Retained evidence

The suite-native TOML encoder conformance source now includes:

```text
protos/tests/conformance/library/toml/encoder-positive.protos
Test("encode round-trips a long zero-padded temporal fraction", ...)
```

The case constructs a local time with coefficient `7` and
`fractionDigits = 400`, encodes it, parses it again, and verifies:

- the expected encoded size;
- exact retained coefficient `7`; and
- exact retained fraction width `400`.

This is bounded semantic evidence, not a timing benchmark.

The source-level closure guard also now rejects reintroduction of the concrete
quadratic prepend pattern:

```text
src/test/java/com/guillermomolina/protos/execution/ProtosTomlClosureConformanceTest.java

assertFalse(source.contains("raw = \"0\" + raw"), source)
```

Semantic TOML ownership remains in the suite-native Protos conformance tests;
the Java test remains a structurally distinct source-level architecture guard.

## Preserved boundaries

E2-A changes only encoder accumulation mechanics.

```text
PUBLIC_MODULE=std:toml/TOML
PUBLIC_API_CHANGE=NO
SEMANTIC_NODE_MODEL_CHANGE=NO
TEMPORAL_REPRESENTATION_CHANGE=NO
ENCODED_SPELLING_CHANGE=NO
D104=KEEP
D109=KEEP
D087_PRIVATE_TOML_1_0_BOUNDARY=KEEP
COMPOSITE_CONSTRUCTOR_POLICY_F4=UNCHANGED
SOURCE_PRESERVING_DOCUMENT_SCOPE=UNCHANGED
FILESYSTEM_NETWORK_AUTHORITY=UNCHANGED
```

No deep-validation policy for `TOML.array(...)` or `TOML.table(...)` is
selected or changed by this slice.

## Validation

After publication, the maintainer reported:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This record deliberately does not invent individual command-level results that
were not separately reported.

## AUD005 state after E2-A

```text
AUD005_STATUS=IN_PROGRESS
LIB010_E1_STATUS=CLOSED
LIB010_E2_STATUS=IN_PROGRESS

F1_COMMENT_CONTROL_CONFORMANCE=RESOLVED
F2_QUOTE_RUN_HOST_RECURSION=RESOLVED
F3_OFFICIAL_TOML_1_1_CONFORMANCE_EVIDENCE=OPEN
F4_COMPOSITE_CONSTRUCTOR_VALIDATION=NEEDS_USER_DECISION_OR_EXPLICIT_DEFER
F5_V9_REUSABLE_EVIDENCE_GATE_RECONCILIATION=OPEN
F6_TEMPORAL_FRACTION_SCALING=RESOLVED
F7_FINAL_GITHUB_DURABLE_CLOSURE_RECONCILIATION=OPEN

LIB010_FINAL_CLOSURE=DEFERRED
LIB010_ISSUE_418=KEEP_OPEN
AUD005_CLOSURE=BLOCKED_BY_F3_F4_F5_F7_AND_FINAL_VALIDATION
```

The next corrective work must not silently redefine TOML semantics. In
particular, the official TOML 1.1 corpus/equivalent-provenance requirement
remains a separate conformance-evidence problem, and the composite-constructor
depth question remains outside E2-A.
