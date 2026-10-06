# AUD005 / LIB010-E2-B — official TOML 1.1 conformance integration research

## Status

```text
WORK_ITEM=AUD005
ISSUE=guillermomolina/protos#451
TARGET=LIB010
TARGET_ISSUE=guillermomolina/protos#418
SLICE=LIB010-E2-B
SLICE_TYPE=RESEARCH
STATUS=COMPLETE
PRODUCT_REVISION_AUDITED=62f3f5710210f247aad8574d3d0d56d3254bfd70
PRODUCT_REVISION_SUBJECT=LIB010-E2-A: make TOML temporal fraction encoding linear
F3_OFFICIAL_TOML_1_1_CONFORMANCE_EVIDENCE=IMPLEMENTATION_READY
RECOMMENDED_ARCHITECTURE=RETAINED_OFFICIAL_CORPUS_PLUS_GENERATED_SUITE_NATIVE_PROJECTION
NEW_DXXX_REQUIRED=NO
OWNER_DECISION_REQUIRED=NO
PRODUCT_CODE_CHANGED=NO
PUBLIC_API_CHANGED=NO
TOML_SEMANTICS_CHANGED=NO
D104_CHANGED=NO
D109_CHANGED=NO
D087_CHANGED=NO
F4_CHANGED=NO
```

This is durable non-normative research evidence for AUD005 finding F3. It
records the reproducible upstream baseline, the compatibility analysis against
the already-ratified LIB010/D104/D109 contract, and the selected implementation
shape for retained official TOML 1.1 conformance evidence.

It does not itself integrate the corpus and therefore does not mark F3
resolved.

## Audited Protos state

The research audits:

```text
guillermomolina/protos@62f3f5710210f247aad8574d3d0d56d3254bfd70
```

At that revision:

- `std:toml/TOML` exposes the already-ratified ten semantic kinds plus
  `parse` and `encode`;
- D104 preserves `second = 60` as TOML semantic data without a leap-event
  database;
- D109 owns canonical shortest-round-trip binary64 rendering and retains
  stronger project-owned signed-zero/canonical-spelling evidence;
- D087 private TOOL001/TOOL002 TOML 1.0 ownership remains isolated;
- the retained public TOML suite is project-owned and suite-native;
- the retained case named
  `official style TOML 1.1 document round-trips semantically` is not an
  integration of the official `toml-test` corpus.

The current project-owned TOML suite contains 54 suite-native logical tests
across the existing seven TOML conformance sources. Those tests remain useful
and must not be replaced by weaker upstream coverage.

## Frozen official baseline

The implementation must pin the official corpus to:

```text
UPSTREAM_REPOSITORY=https://github.com/toml-lang/toml-test
UPSTREAM_RELEASE=v2.2.0
UPSTREAM_COMMIT=ce08da1ddb075d1c7596d663c7fcba9a2ae02c5c
UPSTREAM_TREE=05b36075fdf409db0448d3047b5dfdb7d8bd5f6a
TOML_DIALECT=1.1.0

SELECTOR=tests/files-toml-1.1.0
SELECTOR_GIT_BLOB=e2bdb2a669ede0dec8813f7c12db990c9a469f42

LICENSE=MIT
LICENSE_GIT_BLOB=93b22020a83d8a03c300bbaf965ceacc7c00f926
LICENSE_COPYRIGHT=Copyright (c) 2018 TOML authors

GITATTRIBUTES_GIT_BLOB=638165542dec9f82ef0769c871f5395c15b1ce6a
README_GIT_BLOB=c0732a780de355ffe8ca03a58057b9f7a9d282dc
RUNNER_GIT_BLOB=70109dbba905a7f9f4952f877e59454b00ad776e
JSON_COMPARATOR_GIT_BLOB=0190542a8d18069df0f9644f1670cf55dc50e6dd
TOML_COMPARATOR_GIT_BLOB=0582b316db7abf2be1b06c9e018255bee5763058
VERSION_GIT_BLOB=de801180b147f91a1f6558739d761fb26f0fc2ee
```

The dialect must never be inferred from an upstream moving/default setting.
The retained selector for TOML 1.1 is the authoritative case-set definition.

This pin is deliberately a tagged release plus exact commit and selector blob,
not `main`.

## Official corpus inventory

For the frozen selector above:

```text
SELECTED_PHYSICAL_FILES=895
SELECTED_PHYSICAL_BYTES=106344

VALID_CASES=214
VALID_TOML_FILES=214
VALID_TAGGED_JSON_FILES=214

INVALID_CASES=467
INVALID_TOML_FILES=467

ENCODER_CASES=214
UPSTREAM_LOGICAL_CASES=895
```

The 214 encoder cases reuse the same 214 `valid/*.json` /
`valid/*.toml` pairs in the inverse direction.

## Official tagged-JSON contract

The upstream language-neutral decoder contract represents TOML through JSON:

```text
TOML table -> JSON object
TOML array -> JSON array

atomic value ->
{
  "type": "<type>",
  "value": "<string>"
}
```

Official atomic tags are:

```text
string
integer
float
bool
datetime
datetime-local
date-local
time-local
```

The Protos semantic projection is therefore:

```text
string          -> string
integer         -> integer
float           -> float
bool            -> boolean
datetime        -> offsetDateTime
datetime-local  -> localDateTime
date-local      -> localDate
time-local      -> localTime
JSON array      -> array
JSON object     -> table
```

Arrays of tables require no new Protos semantic kind; they remain ordinary
`array` nodes whose elements are `table` nodes.

## String-input boundary and byte-domain exclusions

The already-ratified public operation is:

```text
TOML.parse(text: String)
```

It is not a byte-decoder API.

Eleven official invalid fixtures test malformed byte encodings that cannot
exist as a valid Protos String input:

```text
invalid/encoding/bad-codepoint
invalid/encoding/bad-utf8-at-end
invalid/encoding/bad-utf8-in-array
invalid/encoding/bad-utf8-in-comment
invalid/encoding/bad-utf8-in-multiline
invalid/encoding/bad-utf8-in-multiline-literal
invalid/encoding/bad-utf8-in-string
invalid/encoding/bad-utf8-in-string-literal
invalid/encoding/utf16-bom
invalid/encoding/utf16-comment
invalid/encoding/utf16-key
```

These fixtures must remain in the retained upstream snapshot and must be
explicitly classified:

```text
NOT_APPLICABLE_INPUT_DOMAIN
```

They must not be silently dropped.

The remaining executable inventory is:

```text
INVALID_UPSTREAM=467
INVALID_NOT_APPLICABLE_BYTE_DOMAIN=11
INVALID_EXECUTABLE_FOR_STRING_API=456

PARSER_VALID=214
PARSER_INVALID=456
PARSER_EXECUTABLE_CASES=670

ENCODER_EXECUTABLE_CASES=214

EXECUTABLE_TOTAL_CASES=884
```

The following encoding-related invalid cases remain applicable to String input
and must execute normally:

```text
invalid/encoding/bom-not-at-start-01
invalid/encoding/bom-not-at-start-02
invalid/encoding/ideographic-space
```

## D104 / D109 reconciliation

No new semantic decision is required.

### D104

The frozen official corpus rejects `:61` through the relevant
`second-over` invalid cases.

It does not provide a positive `:60` conformance case.

Therefore:

```text
OFFICIAL_CORPUS_PROVES_SECOND_61_REJECTED=YES
OFFICIAL_CORPUS_PROVES_D104_SECOND_60=NO
```

The retained project-owned D104 tests remain authoritative evidence for the
already-ratified Protos behavior.

### D109

The official encoder equivalence is weaker than D109 in several areas,
including signed zero and exact canonical spelling.

The official corpus is therefore additive evidence only.

Retain project-owned tests for:

- canonical `0.0` and `-0.0`;
- canonical `inf`, `-inf`, and `nan`;
- integral-looking Float preservation as TOML Float;
- difficult binary64 shortest-round-trip values;
- exact D109 deterministic spelling.

### Integer and temporal extensions

The official corpus requires/recommends the interoperable signed-64-bit Integer
domain and binary64 Float domain. Protos' unbounded Integer support is stronger
and remains covered by project-owned tests.

The official temporal comparator/tests exercise millisecond-level semantics.
Protos' exact arbitrary-width decimal temporal fraction is stronger; the
existing long-fraction evidence remains necessary.

## Architecture alternatives

Three materially relevant integration families were evaluated.

### A — invoke the official runner

Strengths:

- closest to upstream execution;
- independent blessed decoder for encoder output;
- direct upstream behavior.

Costs:

- external Go/binary dependency;
- process/stdin/stdout adapters;
- platform packaging and CI dependency;
- parallel execution path outside the suite-native Test Tool model.

Result:

```text
NORMAL_TEST_ARCHITECTURE=REJECT
EXTERNAL_AUDIT_TOOL=PERMITTED_FUTURE_OPTION
```

The official runner is not selected as mandatory normal repository test
infrastructure.

### B — retain the exact raw corpus only

Strengths:

- exact provenance;
- small retained size;
- offline;
- easy upstream diff/update.

Weakness:

- raw retention alone is not executable conformance evidence.

Result:

```text
RAW_RETENTION=REQUIRED
COMPLETE_ARCHITECTURE=INSUFFICIENT_ALONE
```

### C — retain exact raw corpus plus deterministic suite-native projection

Selected.

The repository retains the exact selector-selected official files and derives a
reviewable suite-native projection whose logical Test names retain the upstream
case identity.

The existing Test Tool already separates physical source association from
logical Case selector and derives stable Case identity from both. This permits
many upstream cases to be grouped into bounded generated source shards without
creating one module per fixture, while preserving independently selectable and
parallelizable logical Cases.

Result:

```text
SELECTED_ARCHITECTURE=C
RAW_UPSTREAM_SNAPSHOT=AUTHORITATIVE_SOURCE
GENERATED_SUITE_NATIVE_PROJECTION=EXECUTABLE_DERIVATIVE
NETWORK_REQUIRED_FOR_NORMAL_TESTS=NO
GO_REQUIRED_FOR_NORMAL_TESTS=NO
TOML_TEST_BINARY_REQUIRED_FOR_NORMAL_TESTS=NO
```

## Selected retained layout

The implementation should retain official upstream material under a dedicated
versioned root, conceptually:

```text
protos/tests/conformance/library/toml/official/upstream/v2.2.0/
    LICENSE
    .gitattributes
    files-toml-1.1.0
    PROVENANCE.toml
    CASES.tsv
    SHA256SUMS
    tests/
        valid/...
        invalid/...
```

Only files selected by `files-toml-1.1.0` belong under the retained
`tests/` subtree.

Generated executable sources should be category-sharded beneath:

```text
protos/tests/conformance/library/toml/official/generated/
```

A shared suite-native assertion/projection helper may live under:

```text
protos/tests/conformance/library/toml/official/
```

The exact number of generated source files is mechanical implementation detail,
but the logical case set is not: it must reconcile exactly to the 884 executable
cases plus the 11 explicit non-applicable upstream cases.

## Encoder evidence shape

For decoder-valid cases:

1. parse the retained TOML fixture through `TOML.parse`;
2. project the resulting Protos semantic tree into the official tagged-JSON
   semantic domain;
3. compare with the retained official expected JSON.

For decoder-invalid cases:

1. feed the applicable retained TOML semantic String to `TOML.parse`;
2. require rejection.

For encoder cases:

1. project retained tagged JSON into the Protos TOML semantic tree;
2. encode through `TOML.encode`;
3. parse the emitted TOML through the independently corpus-tested Protos parser;
4. compare the resulting semantic value with the expected official semantic
   value.

This encoder oracle is not as implementation-independent as the upstream
runner's blessed decoder. The limitation must be stated rather than hidden.
It is acceptable for F3 because the same Protos parser is independently
constrained by the 670 applicable official decoder cases.

## Provenance and update contract

Normal test execution must be completely offline.

The retained provenance must make updates mechanical and reviewable:

```text
repository
release
exact upstream commit
exact upstream tree
TOML dialect
selector path + blob
license path + blob
retained file set
per-file SHA-256
case classifications
case counts
generated projection revision/method
```

A future upstream update must:

1. explicitly select a new tagged release/commit;
2. compare old and new selectors;
3. report added, removed, and changed fixtures;
4. refresh retained exact bytes;
5. regenerate hashes, applicability and suite-native projection;
6. fail closed if retained file sets/counts/hashes disagree;
7. require review of semantic differences before publication.

The generated projection is never authoritative over the raw retained corpus.

## License

The upstream corpus is MIT licensed with:

```text
Copyright (c) 2018 TOML authors
```

The implementation must retain the upstream MIT LICENSE in the vendored
versioned corpus root. It must not invent Protos license headers inside exact
upstream fixture copies.

## Preserved boundaries

```text
PUBLIC_MODULE=std:toml/TOML
PUBLIC_API_CHANGE=NO
TOML_SEMANTICS_CHANGE=NO
D104=KEEP
D109=KEEP
D087_PRIVATE_TOML_1_0_BOUNDARY=KEEP
F4_COMPOSITE_CONSTRUCTOR_POLICY=UNCHANGED
SOURCE_PRESERVING_DOCUMENT_SCOPE=UNCHANGED
FILESYSTEM_NETWORK_AUTHORITY=UNCHANGED
TOOL001_TOOL002_DIALECT_MIGRATION=NO
```

The official conformance harness must not introduce a runtime dependency from
TOOL001/TOOL002 to the public TOML library or vice versa.

## Research conclusion

```text
LIB010_E2_B_RESEARCH=COMPLETE
F3_STATUS=IMPLEMENTATION_READY
SELECTED_ARCHITECTURE=C
NEW_DXXX_REQUIRED=NO
OWNER_DECISION_REQUIRED=NO
NEXT_SLICE=LIB010-E2-C
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

LIB010-E2-C may implement the retained official TOML 1.1 corpus integration
mechanically under this evidence. If implementation discovers a genuine
semantic contradiction with the ratified public TOML contract, it must stop and
route that contradiction through the existing decision process rather than
silently changing TOML semantics.

## AUD005 state after E2-B research

```text
AUD005_STATUS=IN_PROGRESS
LIB010_E1_STATUS=CLOSED
LIB010_E2_STATUS=IN_PROGRESS

F1_COMMENT_CONTROL_CONFORMANCE=RESOLVED
F2_QUOTE_RUN_HOST_RECURSION=RESOLVED
F3_OFFICIAL_TOML_1_1_CONFORMANCE_EVIDENCE=IMPLEMENTATION_READY
F4_COMPOSITE_CONSTRUCTOR_VALIDATION=NEEDS_USER_DECISION_OR_EXPLICIT_DEFER
F5_V9_REUSABLE_EVIDENCE_GATE_RECONCILIATION=OPEN
F6_TEMPORAL_FRACTION_SCALING=RESOLVED
F7_FINAL_GITHUB_DURABLE_CLOSURE_RECONCILIATION=OPEN

LIB010_FINAL_CLOSURE=DEFERRED
LIB010_ISSUE_418=KEEP_OPEN
AUD005_CLOSURE=BLOCKED_BY_F3_IMPLEMENTATION_F4_F5_F7_AND_FINAL_VALIDATION
```
