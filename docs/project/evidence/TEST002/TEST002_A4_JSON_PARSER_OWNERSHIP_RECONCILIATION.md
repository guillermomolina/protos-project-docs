# TEST002-A4 JSON parser semantic ownership reconciliation

Date: 2026-10-06

Owner: `TEST002` / `guillermomolina/protos#538`

This is durable snapshot evidence for the published TEST002-A4 implementation.
It records the exact Protos revision and maintainer-reported local validation;
it is not a replacement for live GitHub coordination and is not a test-ownership
registry.

## Exact publication

```text
PROTOS_REVISION=0d478c5bf4aabaaac781b80cc7f5889c8bbc9160
SUBJECT=TEST002-A4: reconcile JSON parser semantic test ownership
IMPLEMENTATION_VERSION=0.3.226-SNAPSHOT
```

The publication changed:

- `CHANGELOG.md`;
- `pom.xml`;
- `protos/tests/conformance/library/json/final-deep-stress.protos`;
- `protos/tests/conformance/library/json/final-large-materialization.protos`;
- removed
  `src/test/java/com/guillermomolina/protos/execution/ProtosJsonParserModuleTest.java`;
- made
  `src/test/java/com/guillermomolina/protos/execution/ProtosJsonParserStress.java`
  self-contained by moving in its explicit stress-only helper logic.

No JSON production source, Test Tool source, or normative specification changed.

## Reconciled semantic ownership

The old `nestedArraysPreserveStructureAcrossMultipleLevels` Java contract is
now covered more strongly by the suite-native
`library/json/final-deep-stress.protos` case: it walks the complete 2048-level
materialized structure, verifies one element per level, reaches the terminal
node and verifies the terminal JSON kind.

The old
`arrayMaterializationPreservesElementsAcrossChunkBoundaries` contract is now
owned by the suite-native
`library/json/final-large-materialization.protos` case
`final array materialization chunk boundary`. It parses distinguishable values
0 through 63 and verifies indices 0, 1, 2, 31, 32 and 63 with exact coefficient
and exponent values, preserving the legacy chunk-boundary/order contract.

The same suite-native case proves guest-observable open state of a parsed number
node and its decimal payload by successfully creating local `probe` slots.
The parsed Array frozen-state contract remains owned by existing suite-native
JSON structural-error coverage rather than by a duplicate Java semantic owner.

The explicit `ProtosJsonParserStress` harness remains Java because it owns
non-default parser implementation-scale stress evidence. TEST002-A4 only made
its previously borrowed helpers local; it did not migrate or broaden that stress
scope.

## Validation provenance

The maintainer reported after publication:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

No separate remote-CI result is claimed here.

## Execution cohort result

TEST002-A2 originally classified the 259 surviving pre-policy
`execution` Java files. A3 removed two pure TOML semantic owners and A4 removed
the final pure JSON parser semantic owner. The two TOML `SPLIT` classes remain
only for their distinct Java bootstrap/runtime assertions.

At exact A4 revision, intersection with the policy cutoff
`e99d0baba547ac41b3894f32ddca450172ee1f8b` yields:

```text
CURRENT_LEGACY_JAVA_SURVIVORS=390
CURRENT_LEGACY_EXECUTION_SURVIVORS=256
EXECUTION_AUDIT_AND_RECONCILIATION=COMPLETE
EXECUTION_KNOWN_MIGRATE_TO_PROTOS_RESIDUAL=0
```

The remaining unaudited pre-policy population is exactly 134 files:

| Package | Files |
| --- | ---: |
| runtime | 59 |
| parser | 18 |
| semantic | 17 |
| cli | 14 |
| conformance | 7 |
| lsp | 7 |
| analysis | 5 |
| lexer | 4 |
| documentation | 3 |
| **Total** | **134** |

The four LM008 parser-negative fixture classes were already independently
classified `KEEP_JAVA_BOOTSTRAP` in the initial TEST002 inventory, so they are
not unresolved migration candidates even though they remain part of the
remaining package-level accounting.

## Result and next slice

```text
TEST002_A4=COMPLETE
JSON_PARSER_LEGACY_DISPOSITION=MIGRATED_TO_PROTOS
DEEP_NESTING_COVERAGE=STRONGER
CHUNK_BOUNDARY_31_32_COVERAGE=EQUIVALENT_OR_STRONGER
DUPLICATE_JAVA_SEMANTIC_OWNER=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
TEST_TOOL_CHANGE=NO
EXECUTION_COHORT_RECONCILIATION=COMPLETE
TEST002_CLOSABLE=NO
```

The next slice should be a single read-only investigation, not nine package
micro-slices:

**TEST002-A5 — classify all 134 remaining pre-policy Java survivors outside
`execution` by real contract, identify all remaining migration/SPLIT batches,
and determine whether TEST002 can close after those bounded implementations.**

A5 should cover `runtime`, `parser`, `semantic`, `cli`, `conformance`,
`lsp`, `analysis`, `lexer`, and `documentation` together, while grouping
implementation candidates by coherent contract rather than package count.
