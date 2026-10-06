# AUD013-B3 and final closure evidence

## Status

```text
WORK_ITEM=AUD013/#581
SLICE=AUD013-B3
TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PUBLISHED_REVISION=b4e9a9814befb7685fc57483a8eac5bf5c1aada7
SUBJECT=AUD013-B3: adopt current Protos forms in remaining maintained sources
RESULT=PASS
PARENT_CLOSURE_READY=YES
NEXT_SLICE=NONE
```

The maintainer reports that the final B3 candidate was published after all
required local tests passed and `git diff --check` was clean. No exact aggregate
test count is asserted here because the handoff supplied the pass result but not
an exact suite count.

## Audit coverage

AUD013-A established the activation baseline and complete adoption manifest.
The source audit then covered the maintained repository in one integrated
audit cycle, partitioned only for reviewability:

```text
AUD013-B1  protos/tests/library/**/*.protos
AUD013-B2  protos/lib/**/*.protos
AUD013-B3  every remaining maintained .protos surface discovered from the tree
```

B3 inventoried the remaining source from the repository tree rather than using
a hardcoded directory-only list.

```text
TOTAL_PROTOS_FILES=726
B1_B2_EXCLUDED=75
B3_FILES_INVENTORIED=651
FINAL_RESCAN_UNCLASSIFIED=0
```

The B3 inventory covered tooling, conformance, tooling tests, package-tool tests
and fixtures, maintained examples/tutorials/benchmarks, test resources, negative
fixtures and all other remaining `.protos` surfaces discovered by the sweep.
Generated `target/test-classes/**` copies were classified as generated rather
than independently modernized.

## B3 final published delta

The final published B3 commit is one commit ahead of its publication base
`b2b132af1338e474857a0e1c9012f2c32f56e869` and changes 21 `.protos` files:

```text
protos/tests/conformance/execution-context/capture-by-reference-and-late-nearer-creation-retargeting.protos
protos/tests/conformance/library/json/final-deep-stress.protos
protos/tests/conformance/library/json/final-streaming-roundtrip.protos
protos/tests/conformance/library/json/text-adapter-reader.protos
protos/tests/conformance/library/json/text-adapter-writer.protos
protos/tests/package-tool/content-identity/record-shape.protos
protos/tests/package-tool/content-identity/varuint.protos
protos/tests/package-tool/execution-plan/fixtures/registry-leaf-v2-cases.protos
protos/tests/package-tool/execution-plan/fixtures/workspace.protos
protos/tests/package-tool/toml-syntax/array-of-tables.protos
protos/tests/tooling/tool002-e1a-package-toml-manifest-plan.protos
protos/tests/tooling/tool002-lib011-options-adoption.protos
protos/tests/tooling/tool004-b-progress-boundary.protos
protos/tests/tooling/tool009-discovery-logical-case-plan.protos
protos/tools/package/ContentIdentity.protos
protos/tools/test/FileSelection.protos
protos/tools/test/LogicalCaseRunner.protos
protos/tools/test/Main.protos
protos/tools/test/Manifest.protos
protos/tools/test/Progress.protos
protos/tools/test/ResourceRequirements.protos
```

The final delta applies only already-ratified current-Protos forms. It introduces
no new syntax, semantics, public API, test institution, formatter policy or
unrelated cleanup.

## Active-manifest outcomes

### TEST003 / LIB016 Assertions

B3 migrated genuine assertion duplication where the legacy helper or inline
shape had only the TEST003-proven assertion contract.

Notable retained/migrated outcomes include:

- the local `require` / `reject` helpers in
  `protos/tests/tooling/tool002-lib011-options-adoption.protos` were removed;
- nine uses became `Assertions.require(...)`;
- five `require(reject(...))` cases became `Assertions.signals(Error, ...)` only
  where any Error was intentionally accepted and no Error identity/category was
  observed;
- additional inline `ifFalse -> Error().signal()` assertions in maintained test
  source were migrated where their conditions were Boolean assertions and the
  Error itself was not part of the oracle;
- aggregating `ok` helpers, fake-executor guards, Error-identity/category tests
  and direct domain validation were retained as distinct/canonical behavior.

The ten `Error.handle` sites embedded in maintained executable Protos source
strings produced by `protos/tools/test/Runner.protos` are explicitly classified:

```text
RUNNER_GENERATED_SOURCE_ERROR_HANDLE_SITES=10
RUNNER_GENERATED_SOURCE_CLASSIFICATION=KEEP_DISTINCT_OR_STILL_CANONICAL
```

They deliberately observe Error identity, parent/category, freshness or handler
behavior, or use Error handling as a negative capability probe. They are not
semantically equivalent to `Assertions.signals`. These source strings are
maintained executable source, not generated-build or historical out-of-scope
material.

### D142 / I049 Map absence

All discovered Map/IdentityMap absence candidates were classified. Two genuine
absence-fallback sites in `ResourceRequirements.protos` adopted `atIfAbsent`.
Membership, duplicate checking, schema/presence validation, get-or-insert,
conditional removal and `containsKey` conformance remained canonical where
presence itself is semantically observed.

### D143 / I050 multiple-slot creation

Thirteen fixed-prefix standard-Array extraction runs adopted multiple-slot
creation. Candidate pairs whose source could not be proven to own standard Array
indexed state were retained rather than rewritten speculatively.

### D147 / I045 recognizes

Seven discarded probe idioms adopted the ratified String/Integer/Float/Array
recognition forms where the probe's sole purpose was the exact semantic-family
check and the surrounding failure behavior was preserved. Domain validation,
slot-presence/reflection checks and stricter protocol checks remained distinct.

### D144 / I048 null control

The final semantic review inspected all 11 proposed `ifNotNull` migrations and
reverted every one to the original explicit canonical-null identity check.

```text
IFNOTNULL_SITES_REVIEWED=11
IFNOTNULL_MIGRATIONS_RETAINED=0
D144_ADOPTION_CHECKED=YES
```

The reason is semantic, not stylistic: the receivers did not have contracts that
excluded a closer ordinary `ifNotNull` slot, so replacing non-overridable
`=== null` branching with ordinary selector dispatch would have widened
observable behavior. All 11 are therefore
`KEEP_DISTINCT_OR_STILL_CANONICAL`.

## Retained fixtures and distinct forms

The integrated sweep retained intentional source when modernization would alter
what the source is meant to prove. This includes:

- Error handler/signaling/ensure/cancellation conformance;
- `containsKey` Map/IdentityMap semantics tests;
- `parent() === ...` reflection/identity tests;
- explicit null-behavior oracles and parser/EOF/initialization state;
- negative syntax/error-subject fixtures;
- aggregating test helpers whose result is not a simple assertion;
- production domain validation and get-or-insert/update logic;
- maintained executable Runner-generated source whose Error handling is itself
  semantically observed.

No unresolved candidate required `ROUTE_TO_EXISTING_OWNER` or
`ROUTE_TO_NEW_DESIGN_OR_LIBRARY_WORK` at final closure.

## Concurrency reconciliation

B3 began from `22bdcdc455cdaff3c4509e96555f28623d6254b7`. During the sweep AUD007-B2
advanced `main` to `b2b132af1338e474857a0e1c9012f2c32f56e869`. That concurrent commit
changed `Makefile`, `pom.xml`, `CHANGELOG.md`, `scripts/**` and Java/DAP tests,
with no intersection with the B3 Protos-owned paths. The B3 publication was then
made directly on top of that current head as
`b4e9a9814befb7685fc57483a8eac5bf5c1aada7`.

## Validation

Maintainer-provided final evidence:

```text
FULL_LOCAL_REQUIRED_TESTS=PASS
GIT_DIFF_CHECK=PASS
PUBLICATION=PUSHED
FINAL_RESCAN_UNCLASSIFIED=0
```

The semantic review that reverted the 11 `ifNotNull` candidates changed code,
so the earlier validation was correctly discarded and the maintainer reran the
required tests on the final published candidate before publication.

## Parent closure criteria

```text
ACTIVATION_BASELINE=RECORDED
ADOPTION_MANIFEST=COMPLETE
MAINTAINED_PROTOS_SOURCE_SURFACES=AUDITED
BASELINE_OBLIGATIONS=CHECKED_IN_SINGLE_INTEGRATED_PASS
SEMANTICS_PRESERVING_MIGRATIONS=COMPLETE
INTENTIONAL_LEGACY_FIXTURES=JUSTIFIED
UNRESOLVED_DESIGN_GAPS=ROUTED_NOT_INVENTED
TEST003_ASSERTION_ADOPTION=CONSUMED
DOCUMENTATION_MODEL_ADOPTION=CHECKED_IF_RATIFIED_AT_BASELINE
OTHER_RATIFIED_ADOPTION_OBLIGATIONS=CHECKED
COVERAGE=EQUIVALENT_OR_STRONGER
FULL_REQUIRED_VALIDATION=PASS
FINAL_RESCAN=NO_UNCLASSIFIED_LEGACY_PATTERNS
CURRENT_PROTOS_BASELINE=ESTABLISHED
```

Interpretation of `BASELINE_OBLIGATIONS=CHECKED_IN_SINGLE_INTEGRATED_PASS`: the
AUD013 audit cycle reviewed all active manifest dimensions together while
walking each source partition; it did not run an independent repository-wide
campaign per feature. A/B1/B2/B3 are reviewable slices of that one integrated
audit cycle, not separate feature-specific audits.

Already-completed owner work consumed rather than repeated includes I046
`fail`, TOOL009/LIB018 Test authoring/execution, DOC008 source documentation,
I041 matching, I078 `whileTrue`, AUD009-owned removals/relocations and the
superseded D179 C3 path.

## Final outcome

```text
AUD013_B3=COMPLETE
AUD013_PARENT_CLOSURE_READY=YES
FINAL_RESCAN_UNCLASSIFIED=0
CURRENT_PROTOS_BASELINE=ESTABLISHED
ISSUE_581_CLOSE_AS=COMPLETED
NEXT_SLICE=NONE
```
