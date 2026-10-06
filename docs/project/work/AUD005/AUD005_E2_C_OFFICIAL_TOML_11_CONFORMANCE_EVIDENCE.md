# AUD005 / LIB010-E2-C — official TOML 1.1 conformance corpus evidence

## Status

```text
WORK_ITEM=AUD005
ISSUE=guillermomolina/protos#451
TARGET=LIB010
TARGET_ISSUE=guillermomolina/protos#418
SLICE=LIB010-E2-C
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PRODUCT_REVISION=22bdcdc455cdaff3c4509e96555f28623d6254b7
PRODUCT_REVISION_SUBJECT=LIB010-E2-C: retain official TOML 1.1 conformance corpus
IMPLEMENTATION_VERSION=0.3.232-SNAPSHOT
F3_OFFICIAL_TOML_1_1_CONFORMANCE_EVIDENCE=RESOLVED
PUBLIC_API_CHANGED=NO
SPECIFICATION_CHANGED=NO
TOML_SEMANTICS_CHANGED=NO
D104_CHANGED=NO
D109_CHANGED=NO
D087_CHANGED=NO
F4_CHANGED=NO
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This is durable non-normative implementation evidence for AUD005 finding F3.
It records the published retained official TOML 1.1 conformance corpus and its
suite-native executable projection. It does not redefine public TOML semantics.

## Published product revision

```text
22bdcdc455cdaff3c4509e96555f28623d6254b7
LIB010-E2-C: retain official TOML 1.1 conformance corpus
```

The publication advances the implementation version to:

```text
0.3.232-SNAPSHOT
```

The commit is published on `main`.

## Frozen upstream provenance

The retained corpus is pinned to:

```text
UPSTREAM_REPOSITORY=https://github.com/toml-lang/toml-test
UPSTREAM_RELEASE=v2.2.0
UPSTREAM_COMMIT=ce08da1ddb075d1c7596d663c7fcba9a2ae02c5c
UPSTREAM_TREE=05b36075fdf409db0448d3047b5dfdb7d8bd5f6a
TOML_VERSION=1.1.0
SELECTOR=tests/files-toml-1.1.0
SELECTOR_BLOB=e2bdb2a669ede0dec8813f7c12db990c9a469f42
LICENSE=MIT
LICENSE_BLOB=93b22020a83d8a03c300bbaf965ceacc7c00f926
LICENSE_COPYRIGHT=Copyright (c) 2018 TOML authors
```

The versioned retained root is:

```text
protos/tests/conformance/library/toml/official/upstream/v2.2.0/
```

It contains the exact selected fixture set plus retained provenance, case
classification, checksums and the upstream MIT license.

## Retained inventory

The published `PROVENANCE.toml` records:

```text
SELECTED_FILE_COUNT=895
SELECTED_BYTE_COUNT=106344
VALID_COUNT=214
INVALID_COUNT=467
ENCODER_COUNT=214
INVALID_APPLICABLE_COUNT=456
EXECUTABLE_COUNT=884
NOT_APPLICABLE_COUNT=11
LOGICAL_CASE_COUNT=895
SHA256SUMS_SHA256=2522b7760d06247ff61d44e6ca4ec4a793f9b6d12fbd667a9aaf7d76e9f6d749
```

The executable reconciliation is therefore:

```text
decoder-valid      214
decoder-invalid    456
encoder            214
                   ---
executable         884

not-applicable      11
                   ---
logical cases      895
```

## Deterministic offline projection

The publication adds:

```text
tools/toml_test_projection.py
tools/test_toml_test_projection.py
```

The generator validates the retained selector and exact fixture set, computes
and verifies byte-level inventory and hashes, classifies the public String-input
boundary, generates the suite-native sources, and supports offline regeneration
checking.

Normal generation/checking requires no network and no `toml-test` executable.

The executable projection contains 27 generated category shards plus the
Protos-owned suite-local:

```text
protos/tests/conformance/library/toml/official/suite/Harness.protos
```

The generated shards are registered in:

```text
protos/tests/conformance/manifest.tsv
```

so the retained official evidence uses the ordinary suite-native Test Tool path
rather than a parallel runner.

## String-input applicability reconciliation

LIB010-E2-B correctly established that malformed byte-domain fixtures that
cannot become the argument of `TOML.parse(String)` must be retained but marked
`NOT_APPLICABLE_INPUT_DOMAIN`.

E2-C improves the **membership** of that set using the actual retained bytes.
The generator applies mechanically:

```text
data.decode("utf-8", errors="strict")
```

to every invalid fixture and fails unless the resulting non-UTF-8 set exactly
matches its pinned list.

The published N/A set is:

```text
invalid/encoding/bad-codepoint
invalid/encoding/bad-utf8-at-end
invalid/encoding/bad-utf8-in-array
invalid/encoding/bad-utf8-in-comment
invalid/encoding/bad-utf8-in-multiline
invalid/encoding/bad-utf8-in-multiline-literal
invalid/encoding/bad-utf8-in-string
invalid/encoding/bad-utf8-in-string-literal
invalid/encoding/bom-not-at-start-01
invalid/encoding/bom-not-at-start-02
invalid/encoding/utf16-bom
```

This supersedes the E2-B research-time filename-based membership assumption.
In particular:

```text
bom-not-at-start-01 = NOT_APPLICABLE_INPUT_DOMAIN
bom-not-at-start-02 = NOT_APPLICABLE_INPUT_DOMAIN

utf16-comment = EXECUTABLE
utf16-key     = EXECUTABLE
```

The count remains exactly 11. The distinction is not semantic policy: it is a
mechanical consequence of whether the retained byte sequence is strict UTF-8.
Fixtures such as `utf16-comment` and `utf16-key` are valid UTF-8 byte
sequences containing interleaved NULs, so they can be represented as Protos
String input and the TOML parser is required to reject them for TOML syntax /
control-character reasons.

No byte-oriented public TOML API is introduced.

## Suite-local official semantic comparator

`Harness.protos` maps official tagged JSON into the already-ratified Protos
TOML semantic model and mirrors the official comparator semantics needed by the
corpus.

It explicitly keeps the official corpus weaker than the stronger Protos-owned
guarantees where appropriate. Existing project-owned tests remain authoritative
for:

- D104 positive `second = 60`;
- D109 deterministic Float spelling and signed zero;
- unbounded Protos Integer behavior;
- exact arbitrary-width temporal fraction preservation.

The encoder path uses `TOML.encode` followed by `TOML.parse` and semantic
comparison. The harness documents that this oracle is less implementation-
independent than the official Go runner's blessed decoder; the parser itself is
independently constrained by the retained official decoder cases.

## Parser defects exposed and repaired

The retained official corpus exposed two real `TOML.parse` conformance defects
under already-ratified TOML 1.1 semantics.

### Empty array / empty inline-table continuation

Before E2-C, an empty array or empty inline table followed by other document
content could be rejected because the nested-value state machine re-read the
lookahead after closing the frame instead of preserving the already-observed
closing condition.

E2-C makes the closing/trailing condition explicit before frame closure.

### Dotted-key extension through inline-table values

Before E2-C, a dotted key within an inline table could extend a table that had
already been supplied as an inline-table value.

E2-C tracks tables created by dotted-key expansion within that inline table and
permits extension only through those tables. A completed inline-table value is
therefore no longer silently reopened.

The publication adds Protos-owned regressions in the existing parser positive
and negative conformance sources.

These are mechanical conformance fixes under the ratified TOML 1.1 contract.
They introduce no new Dxxx decision and no public API change.

## License / third-party notice

The upstream MIT `LICENSE` is retained with the corpus, and
`THIRD_PARTY_NOTICES.md` records the bundled toml-test v2.2.0 test data and
license notice.

Exact upstream fixture copies are not rewritten with Protos license headers.

## Preserved boundaries

```text
PUBLIC_MODULE=std:toml/TOML
PUBLIC_API_CHANGED=NO
SPECIFICATION_CHANGED=NO
TOML_SEMANTICS_CHANGED=NO
D104=KEEP
D109=KEEP
D087_PRIVATE_TOML_1_0_BOUNDARY=KEEP
F4_COMPOSITE_CONSTRUCTOR_POLICY=UNCHANGED
DOCUMENT_CST_SCOPE=UNCHANGED
FILESYSTEM_NETWORK_AUTHORITY=UNCHANGED
TOOL001_TOOL002_DIALECT_MIGRATION=NO
```

## Validation

The maintainer reports after publication:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

No narrower command-level result is invented here beyond the retained structural
evidence and the maintainer-reported all-local-tests result.

## AUD005 state after E2-C

```text
AUD005_STATUS=IN_PROGRESS
LIB010_E1_STATUS=CLOSED
LIB010_E2_STATUS=IN_PROGRESS

F1_COMMENT_CONTROL_CONFORMANCE=RESOLVED
F2_QUOTE_RUN_HOST_RECURSION=RESOLVED
F3_OFFICIAL_TOML_1_1_CONFORMANCE_EVIDENCE=RESOLVED
F4_COMPOSITE_CONSTRUCTOR_VALIDATION=NEEDS_USER_DECISION_OR_EXPLICIT_DEFER
F5_V9_REUSABLE_EVIDENCE_GATE_RECONCILIATION=OPEN
F6_TEMPORAL_FRACTION_SCALING=RESOLVED
F7_FINAL_GITHUB_DURABLE_CLOSURE_RECONCILIATION=OPEN

LIB010_FINAL_CLOSURE=DEFERRED
LIB010_ISSUE_418=KEEP_OPEN
AUD005_CLOSURE=BLOCKED_BY_F4_F5_F7_AND_FINAL_VALIDATION
```

F3 is closed by this publication. AUD005 and LIB010 remain open because F4,
F5, F7 and final integrated closure validation/reconciliation remain.
