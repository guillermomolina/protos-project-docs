# PERF009-B Package Tool reintegration checkpoint

Status: **IN PROGRESS — FOURTH QUARANTINED CLASS REINTEGRATED**

This immutable checkpoint records the completed Package Tool reintegration under
`PERF009-B — Quarantined Java test normalization and reintegration`
(`guillermomolina/protos#546`). It does not close PERF009-B: four Java classes
remain in `JAVA_SLOW_TEST_EXCLUDES`.

## Exact product publication

```text
PROTOS_REVISION=dc785bd15884e69522b982113d758f8879ce4b22
PUBLICATION_BASE=806edb9137b8e55b9324dff3fd945d5d08f4c4a7
IMPLEMENTATION_VERSION=0.3.25-SNAPSHOT
PRODUCT_COMMIT=PERF009-B: reintegrate package tool tests
```

The publication guard proved that the product commit parent and then-current
`origin/main` were both exactly
`806edb9137b8e55b9324dff3fd945d5d08f4c4a7`; publication therefore remained a
non-force fast-forward.

No Protos-visible language, specification, package-format, or Standard Library
semantics change is part of this publication.

## Reintegration shape

The formerly quarantined `ProtosPackageToolProtosTest` mixed several distinct
responsibilities in one aggregate class. PERF009-B preserves the same host/runtime
coverage while separating those responsibilities into ordinary classes:

```text
ProtosPackageToolManifestTest
ProtosPackageToolMetadataPersistenceTest
ProtosPackageToolFilesystemSecurityTest
ProtosPackageToolContentIdentityTest
```

Shared Package Tool harness support is retained in:

```text
ProtosPackageToolProtosTestSupport
```

The final split preserves 26 Java tests in total:

```text
Manifest=5
MetadataPersistence=12
FilesystemSecurity=3
ContentIdentity=6
TOTAL=26
```

This is a responsibility split rather than a timing-only partition: manifest
loading, metadata/lock persistence, host filesystem security, and ContentIdentity
integration have distinct ownership and fixtures.

## ContentIdentity semantic ownership

Six frozen positive ContentIdentity vectors were moved from repeated Java guest
execution into the canonical TOOL002 project-tree corpus. The project-tree cases
preserve the exact historical logical trees for:

```text
minimal
ordering
binary
varuint-boundaries
exact-case-upper
exact-case-lower
```

A prepublication parity guard compared every file path and every byte of all six
materialized trees with the historical Java vectors.

Each vector now owns both exact digest coverage and positive verification
coverage in TOOL002:

```text
POSITIVE_DIGEST_VECTORS=6
POSITIVE_VERIFY_VECTORS=6
CONTENT_IDENTITY_CORPUS_CASES=12
SEMANTIC_PARITY_GUARD=PASS
```

The six expected SHA-256 digests remain frozen; the migration therefore changes
execution ownership, not the expected ContentIdentity contract.

Java coverage remains for host/runtime integration properties that are not pure
semantic vector ownership, including invalid logical trees, case-fold collision,
symlink rejection, recorded-identity validation, suspending content reads, and
varuint implementation integration.

## Resolution-input lock ownership

The existing fresh/stale resolution-input fixtures require a real project-tree
filesystem because `LockFile.isStale(...)` reads `protos.lock`. They therefore do
not belong in the source-only `case-outcomes` corpus.

The final shape keeps the ordinary source-only resolution-input corpus at 12
cases and introduces a dedicated project-tree corpus:

```text
package-tool-resolution-input=12/12
package-tool-resolution-input-lock=2/2
```

The new project-tree corpus owns one fresh and one stale lockfile comparison and
supplies the exact project-local `protos.lock` authority required by those cases.

## Multi-chunk TextReader regression

The former multi-chunk manifest case created hundreds of export entries, mixing
the host TextReader chunk-boundary property with parser-scale workload.

The normalized fixture instead uses valid TOML containing one long comment. Its
source remains larger than the production TextReader read-ahead boundary:

```text
SOURCE_READ_AHEAD=8192
SELECTED_SOURCE_BYTES=9065
BOUNDARY_MARGIN=873
```

The correctness property is therefore preserved: the manifest must cross more
than one TextReader source chunk. The parser no longer has to process hundreds
of exports merely to establish that host I/O property.

Isolated fresh-Maven invocation remained dominated by startup cost and was not
used as the reintegration timing gate. Comparable full-class execution showed
the normalized multi-chunk test below the hard ordinary-test budget.

## SHA-256 implementation normalization

The ContentIdentity path exposed repeated pure-Protos SHA-256 bit operations as
significant test cost. The publication keeps SHA-256 as a pure-Protos Standard
Library implementation and replaces recursive per-bit ternary operations with
2-bit lookup-table processing for `xor3`, `choose`, and `majority`.

The lookup implementation processes 16 two-bit pairs for each 32-bit word. The
published semantic corpus, including all six frozen ContentIdentity digest and
verification vectors, passes unchanged expected digests.

This is an implementation-performance change only; no SHA-256 observable result
or public API changes.

## Repeated timing evidence

The approved PERF009-B limits remain:

```text
HARD_PER_TEST_WALL_BUDGET=<2.0s
CLASS_QUARANTINE_THRESHOLD=<5s
```

Five comparable runs of all four responsibility classes produced:

```text
RUN 1
ProtosPackageToolManifestTest:             class=1.786s max_test=1.254s
ProtosPackageToolMetadataPersistenceTest: class=1.578s max_test=0.396s
ProtosPackageToolFilesystemSecurityTest:  class=0.053s max_test=0.040s
ProtosPackageToolContentIdentityTest:      class=2.026s max_test=0.868s

RUN 2
ProtosPackageToolManifestTest:             class=1.749s max_test=1.188s
ProtosPackageToolMetadataPersistenceTest: class=1.666s max_test=0.375s
ProtosPackageToolFilesystemSecurityTest:  class=0.048s max_test=0.033s
ProtosPackageToolContentIdentityTest:      class=1.837s max_test=0.745s

RUN 3
ProtosPackageToolManifestTest:             class=1.975s max_test=1.341s
ProtosPackageToolMetadataPersistenceTest: class=1.941s max_test=0.547s
ProtosPackageToolFilesystemSecurityTest:  class=0.055s max_test=0.041s
ProtosPackageToolContentIdentityTest:      class=2.216s max_test=0.984s

RUN 4
ProtosPackageToolManifestTest:             class=1.907s max_test=1.347s
ProtosPackageToolMetadataPersistenceTest: class=1.889s max_test=0.456s
ProtosPackageToolFilesystemSecurityTest:  class=0.066s max_test=0.049s
ProtosPackageToolContentIdentityTest:      class=2.024s max_test=0.872s

RUN 5
ProtosPackageToolManifestTest:             class=1.729s max_test=1.275s
ProtosPackageToolMetadataPersistenceTest: class=1.747s max_test=0.485s
ProtosPackageToolFilesystemSecurityTest:  class=0.062s max_test=0.042s
ProtosPackageToolContentIdentityTest:      class=2.087s max_test=0.906s
```

Therefore:

```text
MANIFEST_CLASS_MAX=1.975s
METADATA_PERSISTENCE_CLASS_MAX=1.941s
FILESYSTEM_SECURITY_CLASS_MAX=0.066s
CONTENT_IDENTITY_CLASS_MAX=2.216s
MAX_ORDINARY_TESTCASE=1.347s
EVERY_ORDINARY_TESTCASE_LT_2S=YES
EVERY_ORDINARY_CLASS_LT_5S=YES
REPEATED_TIMING_EVIDENCE=PASS
```

## Final repository validation

After rebasing the working state onto the exact publication base, completing the
`0.3.25-SNAPSHOT` implementation bump, and restoring six-of-six positive
ContentIdentity verification parity, the final repository validation passed:

```text
JAVA_PARALLEL_TESTS=1878
JAVA_PARALLEL_FAILURES=0
JAVA_PARALLEL_ERRORS=0
JAVA_SERIAL_TESTS=3
JAVA_SERIAL_FAILURES=0
JAVA_SERIAL_ERRORS=0
PROTOS_TESTS=1285
PROTOS_FAILURES=0
PACKAGE_TOOL_RESOLUTION_INPUT=12/12_PASS
PACKAGE_TOOL_CONTENT_IDENTITY=12/12_PASS
PACKAGE_TOOL_RESOLUTION_INPUT_LOCK=2/2_PASS
FULL_TESTS=PASS
GIT_DIFF_CHECK=PASS
```

The Protos corpus count differs from the earlier pre-rebase local run because the
intervening published `0.3.21`-`0.3.24` work removed 38 superseded dedicated
matching cases, while this slice added five additional ContentIdentity positive
verification cases. The final product-bound evidence is the 1285/1285 result at
this checkpoint's publication state.

## Quarantine state after publication

The published `JAVA_SLOW_TEST_EXCLUDES` now contains four PERF009-B classes:

```text
ProtosExternalPackagePlanningPreflightTest
ProtosWorkspaceRunCliTest
ProtosJsonParserModuleTest
ProtosPackageExecutionPlanAdapterTest
```

The former `ProtosPackageToolProtosTest` no longer exists as a test class and is
no longer quarantined. Its 26 Java tests are retained under the four ordinary
responsibility classes above.

## PERF009-B state

```text
QUARANTINED_JAVA_CLASSES_INITIAL=8
QUARANTINED_JAVA_CLASSES_CURRENT=4
REINTEGRATED_CLASSES=4
MANIFEST_REINTEGRATION=PASS
TOML_PARSER_REINTEGRATION=PASS
TOML_ENCODER_REINTEGRATION=PASS
PACKAGE_TOOL_REINTEGRATION=PASS
ORDINARY_TEST_HARD_BUDGET=<2.0s_PER_TEST
CLASS_QUARANTINE_THRESHOLD=<5s
REPEATED_TIMING_EVIDENCE=PASS
SEMANTIC_COVERAGE=EQUIVALENT_OR_STRONGER
FULL_JAVA_LANE=GREEN
FULL_PROTOS_LANE=GREEN
PERF009_B_CLOSED=NO
```

The remaining PERF009-B work continues class by class under the same timing and
coverage-preservation policy. This checkpoint records the Package Tool slice
only and does not pre-approve remediation choices for the four remaining
quarantined classes.
