# TEST002-A3 TOML semantic ownership migration

Date: 2026-10-06

Owner: `TEST002` / `guillermomolina/protos#538`

This is durable snapshot evidence for the published TEST002-A3 implementation.
It records the exact Protos revision and the maintainer-reported local validation
result; it is not a replacement for live GitHub coordination and is not a test
ownership registry.

## Exact publication

```text
PROTOS_REVISION=bb98d68b043b0af81385fbfbaa44d682e29244d6
SUBJECT=TEST002-A3: migrate TOML semantic tests to Protos
IMPLEMENTATION_VERSION=0.3.225-SNAPSHOT
```

The publication changed exactly the TEST002-A3-owned TOML test surfaces plus
publication metadata:

- `CHANGELOG.md`;
- `pom.xml`;
- seven new suite-native files under
  `protos/tests/conformance/library/toml/`;
- `protos/tests/conformance/manifest.tsv`;
- `ProtosTomlClosureConformanceTest.java`;
- `ProtosTomlDataModelModuleTest.java`;
- removal of `ProtosTomlEncoderModuleTest.java`;
- removal of `ProtosTomlParserModuleTest.java`.

No production TOML implementation or normative specification file changed.

## Suite-native semantic owner

The public `std:toml/TOML` semantic contracts formerly exercised through the
legacy Java/JUnit harness are now registered as ordinary suite-native Test Tool
sources:

- `data-model.protos`;
- `data-model-errors.protos`;
- `parser-positive.protos`;
- `parser-errors.protos`;
- `encoder-positive.protos`;
- `encoder-errors.protos`;
- `roundtrip.protos`.

The migration covers the legacy parser, data-model, encoder and round-trip
semantics, including the previously important edge surfaces: TOML 1.1 string
forms and controls, exact numeric/float spelling, negative zero, temporal kinds
and D104 second 60, dotted/table/inline/AoT structure, malformed-input rejection,
cycles, nested containers, shared acyclic containers, escaping and deterministic
temporal spelling.

## Java ownership retained

TEST002-A3 intentionally did not mechanically remove every TOML Java test.

`ProtosTomlClosureConformanceTest` now retains only
`publicTomlSourceKeepsTheD087AndHostRuntimeBoundaries`, which reads the physical
`protos/lib/toml/TOML.protos` source and asserts the source/host architecture
boundary.

`ProtosTomlDataModelModuleTest` now retains only:

- `importedModuleExportsExactlyLib010ASemanticConstructors`, which checks the
  exact physical module local-slot export surface; and
- `semanticDataTransfersAcrossActorsWhileModuleRemainsActorLocal`, which checks
  Actor-local module identity and host/runtime snapshot transfer behavior.

Those are materially distinct bootstrap/runtime contracts, not duplicate guest
semantic ownership.

## Validation provenance

The maintainer reported after publication:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

No separate remote-CI result is claimed by this record.

## Result

```text
TEST002_A3=COMPLETE
TOML_SEMANTIC_OWNER=PROTOS_TEST_TOOL
TOML_JAVA_DUPLICATE_SEMANTIC_OWNER=REMOVED
TOML_JAVA_SOURCE_BOUNDARY=RETAINED
TOML_JAVA_EXACT_EXPORT_SURFACE=RETAINED
TOML_JAVA_ACTOR_TRANSFER_IDENTITY=RETAINED
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
TEST_OWNERSHIP_REGISTRY_REQUIRED=NO
```

TEST002 remains open. The execution-package audit still has one pure
`MIGRATE_TO_PROTOS` residual:
`ProtosJsonParserModuleTest.java`.

The next bounded slice is **TEST002-A4 — JSON parser residual semantic ownership
reconciliation** in `guillermomolina/protos`. It should reconcile the two
remaining legacy JSON parser methods against the already suite-native JSON
corpus, add only any missing equivalent-or-stronger guest-observable assertions,
and remove the Java owner only when no materially distinct host/runtime property
remains.
