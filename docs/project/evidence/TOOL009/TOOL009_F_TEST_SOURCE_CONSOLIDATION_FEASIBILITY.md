# TOOL009-F — Test Source Consolidation Feasibility Checkpoint

## Investigation identity

- Owning work item: [TOOL009-F / #694](https://github.com/guillermomolina/protos/issues/694)
- Phase-A intake: [TOOL009-F-A / #695](https://github.com/guillermomolina/protos/issues/695)
- Investigated Protos revision: `2b3a88389da7228caed231a90b14091cf2841115`
- Investigation type: read-only architecture/corpus investigation
- Builds/tests/programs executed: none

This checkpoint records the bounded findings established by the investigation supplied for TOOL009-F. It does **not** complete the repository-wide inventory required by TOOL009-F-A and does not authorize implementation.

## Established feasibility result

```text
TOOL009_TEST_FILE_CONSOLIDATION=FEASIBLE

CURRENT_MODEL=ONE_PHYSICAL_SOURCE_CAN_DECLARE_MULTIPLE_LOGICAL_TESTS
PHYSICAL_FILE_IS_NOT_LOGICAL_CASE_IDENTITY=CONFIRMED
DISCOVERY_MULTI_TEST_SUPPORT=ALREADY_IMPLEMENTED
EXECUTION_MULTI_TEST_SUPPORT=ALREADY_IMPLEMENTED
SCHEDULING_GRANULARITY=LOGICAL_TEST
REPORTING_GRANULARITY=LOGICAL_TEST

NEW_RUNTIME_MECHANISM_REQUIRED=NO
NEW_TEST_MODEL_REQUIRED=NO
TEST_TOOL_ARCHITECTURE_CHANGE_REQUIRED=NO

PRIMARY_WORK=CORPUS_SOURCE_CONSOLIDATION
MANIFEST_REWRITE_REQUIRED=YES
SEMANTIC_REGROUPING_REQUIRED=YES
```

The current TOOL009 architecture already supports the intended physical consolidation. A source may declare multiple named Tests while preserving independent Logical Cases.

## Discovery and execution model

The investigation established the following current execution model:

1. `Discovery.declarationSignature()` traverses the source-local `tests` array and includes all declared Test names in the declaration signature.
2. `LogicalCasePlan.build()` expands those declarations into one independent Logical Case per selector while retaining declaration position.
3. `LogicalCaseRunner` flattens and schedules the Logical Cases independently.
4. During execution, `resolveSelectedTest()` rematerializes the source, validates the complete declaration signature, resolves the selected Test, and executes only that Test.

Therefore a physical source such as:

```text
float/arithmetic.protos

tests: [
    Test("add", ...),
    Test("subtract", ...),
    Test("multiply", ...),
    Test("divide", ...)
]
```

continues to represent four independently scheduled and executed Logical Cases rather than one composite Case.

## Identity boundary

The investigation confirmed the TOOL009 separation:

```text
physical source
    -> declaration
    -> N named Tests
    -> N Logical Cases
```

The physical source path is a locator/source association, not the Logical Case identity itself.

This finding removes the need for any new runtime, Test model, or Test Tool architecture solely to support source consolidation.

It does **not** establish that every existing test source may be moved or merged without path-sensitive repository consequences. That remains explicit follow-up work.

## Corpus observation

The investigation reported the main production manifest at the investigated revision as:

```text
PRODUCTION_SOURCES=817
SUITE_NATIVE_SOURCES=817
```

It also observed substantial inherited one-test-per-file fragmentation, including representative families such as:

```text
float/add.protos
float/subtract.protos
float/multiply.protos
float/divide.protos
...

package-tool/lock/primitive-decimal.protos
package-tool/lock/primitive-digest.protos
package-tool/lock/primitive-method.protos
...

crypto/sha256/abc.protos
crypto/sha256/a55.protos
crypto/sha256/a56.protos
crypto/sha256/a64.protos
...
```

These examples establish the consolidation problem, but they are not a complete TOOL009-F-A source inventory.

In particular, this checkpoint does **not** yet establish:

```text
IN_SCOPE_SOURCE_FILES
IN_SCOPE_LOGICAL_CASES
SINGLE_CASE_SOURCE_FILES
MULTI_CASE_SOURCE_FILES
ORDINARY_TEST_SOURCES
SPECIAL_EXECUTION_TEST_SOURCES
AUXILIARY_NON_TEST_PROTOS_SOURCES
```

Those remain owned by TOOL009-F-A / #695.

## Manifest consequence

The current manifest remains source-oriented.

Consolidating several physical sources therefore requires manifest reconciliation. For example, conceptually:

```text
float/add.protos        suite-native -
float/subtract.protos   suite-native -
float/multiply.protos   suite-native -
float/divide.protos     suite-native -
```

may become:

```text
float/arithmetic.protos suite-native -
```

while discovery expands that one source into multiple Logical Cases.

The exact production rewrite must be derived from the completed inventory and grouping work; this checkpoint does not prescribe a concrete file taxonomy.

## Grouping direction

The investigation found that an arbitrary numeric rule such as "N tests per file" would not be an appropriate organizing principle.

The proposed direction is semantic/behavioral cohesion. Representative candidate groupings identified during the investigation included:

```text
float/arithmetic.protos
float/special-values.protos
integer/division.protos
integer/remainder.protos
package-tool/lock/primitives.protos
```

These are examples demonstrating plausible grouping style, not an accepted repository-wide target map.

TOOL009-F-B remains responsible for the systematic thematic classification.

## Explicit non-conclusions

This checkpoint does not establish that consolidation is mechanical for every source.

The investigation explicitly identified categories requiring separate bounded analysis, including sources with:

- specialized fixtures or capabilities;
- Actor/Group/Process Snapshot execution;
- project-tree CaseAuthority;
- parser-negative behavior where source validity itself may matter;
- external physical-path coupling.

Accordingly:

- TOOL009-F-C still owns specialized execution-family constraints.
- TOOL009-F-D still owns physical-path coupling.
- TOOL009-F-E still owns the final target grouping map.
- TOOL009-F-F still owns final reconciliation and implementation slicing.

## Reusable handoff

Subsequent TOOL009-F phases should treat the following as established unless direct current-repository evidence contradicts it:

```text
ONE_SOURCE_MANY_LOGICAL_CASES=SUPPORTED
DISCOVERY_MULTI_TEST=SUPPORTED
EXECUTION_MULTI_TEST=SUPPORTED
SCHEDULING_UNIT=LOGICAL_CASE
REPORTING_UNIT=LOGICAL_CASE
PHYSICAL_FILE_IS_NOT_LOGICAL_CASE_IDENTITY=CONFIRMED
NEW_RUNTIME_MECHANISM_REQUIRED=NO
NEW_TEST_MODEL_REQUIRED=NO
TEST_TOOL_ARCHITECTURE_CHANGE_REQUIRED=NO
MANIFEST_REWRITE_REQUIRED=YES
SEMANTIC_REGROUPING_REQUIRED=YES
```

They should not reinvestigate this architecture as part of the normal phased workflow.

## Validation state

This was investigation-only work at exact Protos revision `2b3a88389da7228caed231a90b14091cf2841115`.

- No builds were run.
- No tests were run.
- No programs were run.
- No Protos repository files were modified.
- No implementation readiness conclusion for TOOL009-F as a whole is claimed.

## TOOL009-F-A completed inventory

A subsequent mechanical Phase-A inventory at the same Protos revision
`2b3a88389da7228caed231a90b14091cf2841115` completed the repository-wide
source/Logical Case accounting required by #695.

The authoritative scope is the 20 leaves declared by
`protos/tools/test/RepositorySuite.protos`, reconciled through their registered
corpora and plan-loader inputs.

```text
IN_SCOPE_SOURCE_FILES=1230
IN_SCOPE_LOGICAL_CASES=1258

ZERO_CASE_SOURCE_FILES=0
SINGLE_CASE_SOURCE_FILES=1222
MULTI_CASE_SOURCE_FILES=8

ORDINARY_TEST_SOURCES=895
SPECIAL_EXECUTION_TEST_SOURCES=335
  ACTOR_TEST_SOURCES=11
  GROUP_TEST_SOURCES=10
  PROCESS_SNAPSHOT_TEST_SOURCES=15
  PACKAGE_TEST_SOURCES=299

AUXILIARY_NON_TEST_PROTOS_SOURCES=206

CASES_PER_SOURCE_MIN=1
CASES_PER_SOURCE_MAX=15
CASES_PER_SOURCE_MEDIAN=1
```

The earlier `817/817` observation remains correct for the main
`conformance/manifest.tsv`, but it is not the complete TOOL009-F scope. The
complete inventory contains 413 additional Test sources and 427 additional
Logical Cases across scoped Actor/Group/Process Snapshot corpora, the seven
repository-explicit library corpora, package TOML, and the Package Tool corpora.

Eight registered sources already own more than one Logical Case. Seven do so
through multiple Test declarations. A second, orthogonal source/Case separation
was also confirmed in `package-tool/execution-plan`:
`fixtures/f2e3b-build-v2-error.protos` declares one Test but is referenced by
15 project-tree manifest rows with distinct project identities, producing 15
independently scheduled/reported Logical Cases. Phase B and later phases must
therefore not infer Case cardinality from Test-declaration count alone for
project-tree corpora.

The inventory also classified 206 `.protos` files under `protos/tests/` as
auxiliary non-Test sources. They include Actor/Group bootstrap modules,
JUnit-owned integration/component/resource/parser/CLI/package fixtures, and
project-tree data. They are outside the consolidation source inventory. Two
resolution-input files were found with no repository reference and five
content-identity files were outside the registered manifest but referenced by a
JUnit owner; these were flagged for owner attention, not resolved by Phase A.

Mechanical verification corrected two earlier exploration hazards:

- four manifests have no comment/header row, so unconditional first-row skipping
  silently loses a real Case;
- Package Tool contains additional `lock-file/`, `metadata-publication/`, and
  top-level fixture sources that must be accounted for when reconciling all
  `.protos` files, even though they are not registered Test sources.

Phase A did not propose thematic consolidation, audit physical-path coupling, or
reinvestigate the already-established multi-Test architecture. Those remain
follow-up work under TOOL009-F.

## Current coordination consequence

```text
TOOL009F_ARCHITECTURAL_FEASIBILITY=ESTABLISHED
TOOL009F_PHASE_A=COMPLETE
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

IN_SCOPE_SOURCE_FILES=1230
IN_SCOPE_LOGICAL_CASES=1258
ZERO_CASE_SOURCE_FILES=0
SINGLE_CASE_SOURCE_FILES=1222
MULTI_CASE_SOURCE_FILES=8
ORDINARY_TEST_SOURCES=895
SPECIAL_EXECUTION_TEST_SOURCES=335
AUXILIARY_NON_TEST_PROTOS_SOURCES=206

PHASE_B_INPUT=ESTABLISHED
TOOL009F_IMPLEMENTATION_READY=NOT_YET_ESTABLISHED
```

TOOL009-F-A / #695 may close as complete. The next bounded work is Phase B:
derive the semantic/thematic grouping classification from this inventory without
repeating Phase A or the architectural feasibility investigation.


## TOOL009-F-B bounded semantic grouping checkpoint

Phase B / #696 was completed as a bounded semantic-classification investigation
against the unchanged Protos revision
`2b3a88389da7228caed231a90b14091cf2841115`.

The Phase-A inventory remained authoritative because current `main` was still
identical to that revision when Phase B was performed:

```text
CURRENT_SOURCE_FILES=1230
CURRENT_LOGICAL_CASES=1258
SINGLE_CASE_SOURCE_FILES=1222
EXISTING_MULTI_CASE_SOURCE_FILES=8
```

The investigation deliberately stopped short of a second exhaustive per-source
scan. Phase A already established the mechanical universe; Phase B's useful
question was the semantic boundary for consolidation. A repository-wide exact
count of every proposed target source was judged unnecessary for the next
specialized-execution audit and is deferred to the final grouping/reconciliation
phases after specialized and physical-path constraints are known.

### Established grouping policy

Physical Test sources should be consolidated by coherent behavioral contract,
not by an arbitrary Test-count target, directory minimization, or one-file-per-
subsystem rules.

Positive grouping signals include:

- the same operation family or prototype behavior;
- the same acceptance/error boundary;
- the same parser or surface construct;
- the same Standard Library API surface;
- the same lifecycle/state-transition family; and
- the same Package Tool operation family.

Anti-grouping signals include materially different contracts, different
ExecutionRequirements, distinct fixture/bootstrap models, unusually large Test
bodies, project-tree authority differences, or a result that would create a
human-unmaintainable mega-file.

Logical Cases remain independent. Consolidation means moving multiple existing
named Tests into fewer physical sources; it does not mean combining Test bodies
or weakening Case identity.

### Ordinary semantic families established

Representative ordinary boundaries established from the current corpus include:

- Integer: basic arithmetic, integer division/mod/remainder, floating-result
  division, and receiver/prototype behavior rather than one `integer.protos`;
- Float: arithmetic, special values/overflow/underflow, and domain/receiver
  rejection;
- Boolean: conditional selection, lazy and/or, unary not, and
  `ifTrueIfFalse` behavior;
- Collections: Array, Map, IdentityMap, Set, IdentitySet, and Range grouped by
  operation/contract rather than one collections mega-file;
- Control: `ensure` normal transfer, suspension, and cancellation families;
  `while` ordinary behavior, validation/error transfer, suspension, and
  cancellation;
- Reflection: parent, has-slot, slot-names, remove-slot, close, freeze, and
  slot-value families;
- Text/encoding: Bytes operations, Encoding families, TextReader readText versus
  readLine/lifecycle, and TextWriter lifecycle/encoding/argument behavior;
- JSON: constructors, parser, encoder, event parser, event writer, text adapters,
  and large/deep final stress families; and
- Network: IpAddress and IpEndpoint behavioral surfaces remain distinct.

The explicit library corpora show the same pattern: URI naturally divides into
parse/format/resolve, CSV into parse/encode/row-parser/text-adapter families, CLI
into specification/parse/help/closure families, Integer Math into gcd-lcm,
factorial and power families, and SHA-256 into vector/boundary/ownership-error
families.

These boundaries are semantic candidates, not implementation-ready path moves.
Phase D still owns physical-path coupling.

### Specialized families handed to Phase C

Actor, Group, Process Snapshot, Package Tool, and project-tree-backed corpora
show coherent semantic groupings, but Phase B does not claim physical
consolidation safety for them.

Representative semantic families include:

- Actor: current/identity, spawn, request/message ordering, state/lifecycle, and
  transfer validation;
- Group: acquisition/identity, routing/request behavior, transfer, and stopped
  member behavior;
- Process Snapshot: args, environment, iteration, and snapshot identity;
- Package Tool version: ReleaseVersion parsing/precedence, constraints, fresh
  selection, and retained selection;
- Package Tool lock: primitives, header, quoted strings/tokenization, node refs,
  body records/errors, and canonical writer/order behavior; and
- other Package Tool corpora according to their existing operation/fixture
  vocabulary.

Those groups are classified conceptually as
`SEMANTIC_GROUPING_CANDIDATE` / `SPECIAL_CONSTRAINT_PENDING` until #697 audits
the specialized execution and authority contracts.

The project-tree multiplicity finding from Phase A remains a hard warning:
`package-tool/execution-plan/fixtures/f2e3b-build-v2-error.protos` declares one
Test but backs 15 independently scheduled Logical Cases through distinct project
identities. No specialized grouping decision may infer Logical Case cardinality
from raw `Test(...)` declaration count.

### Phase-B result and intentional deferral

```text
TOOL009F_PHASE_B=COMPLETE_BOUNDED
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

CURRENT_SOURCE_FILES=1230
CURRENT_LOGICAL_CASES=1258
EXISTING_MULTI_CASE_SOURCE_FILES=8

SEMANTIC_GROUPING_CRITERIA=ESTABLISHED
ORDINARY_GROUPING_BOUNDARIES=ESTABLISHED
MEGA_FILE_POLICY=ESTABLISHED

ACTOR_SEMANTIC_GROUPING=ESTABLISHED_PENDING_SPECIAL_AUDIT
GROUP_SEMANTIC_GROUPING=ESTABLISHED_PENDING_SPECIAL_AUDIT
PROCESS_SNAPSHOT_SEMANTIC_GROUPING=ESTABLISHED_PENDING_SPECIAL_AUDIT
PACKAGE_SEMANTIC_GROUPING=ESTABLISHED_PENDING_SPECIAL_AUDIT

LOGICAL_CASES_PRESERVED_BY_PROPOSAL=YES
CROSS_EXECUTION_REQUIREMENT_GROUPS=0

EXACT_REPOSITORY_WIDE_FAMILY_COUNTS=DEFERRED_TO_FINAL_GROUPING_RECONCILIATION
SPECIALIZED_EXECUTION_AUDIT_REQUIRED=YES
PHYSICAL_PATH_AUDIT_REQUIRED=YES

IMPLEMENTATION_PERFORMED=NO
TOOL009F_IMPLEMENTATION_READY=NO
PHASE_C_INPUT=ESTABLISHED
NEXT_PHASE=TOOL009-F-C/#697-specialized-execution-constraints
```

The deferred exact family/source-reduction counts are not evidence of an
implementation-ready map. They should be calculated only after Phase C and Phase
D have eliminated or constrained semantic candidates, avoiding a second
repository-wide enumeration whose result would immediately need reconciliation.


## TOOL009-F-C specialized execution-family checkpoint

Phase C / #697 audited the seven specialized execution/bootstrap families at the
unchanged Protos revision
`2b3a88389da7228caed231a90b14091cf2841115`.

The investigation consumed the Phase-A inventory and Phase-B semantic boundaries
without reopening them. It established that multiple independent Tests may share
one physical source under the current suite-native architecture while preserving
discovery, selector rematerialization, fresh Process per Logical Case,
ExecutionRequirement routing, and the existing bootstrap/resolver facilities.

Established family classification:

```text
PROCESS_SNAPSHOT=SAFE_TO_GROUP
ACTOR=SAFE_TO_GROUP
GROUP=SAFE_TO_GROUP
PACKAGE_TOML=SAFE_TO_GROUP
PACKAGE_CASE_OUTCOMES=SAFE_TO_GROUP
PACKAGE_PROJECT_TREE=GROUP_WITH_CONSTRAINTS
OVERLAY_BOOTSTRAP=SAFE_TO_GROUP
```

The only specialized execution constraint is Package Tool project-tree
authority. Its CaseAuthority granularity is the source CaseSpec/project
projection rather than the Test selector. A physical source can therefore be
consolidated only when the resulting:

```text
Test selectors × project-authority CaseSpecs
```

matrix is exactly the intended Logical Case matrix. Distinct project identities
that reuse one fixture path must remain distinct source associations. No
architecture redesign is required.

Actor and Group retain their existing exact `workers` bootstrap overlays; this
does not require their Test sources to remain one-Test-per-file.

Phase-C result:

```text
TOOL009F_PHASE_C=COMPLETE
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

SPECIAL_EXECUTION_FAMILIES=7
SAFE_TO_GROUP=6
GROUP_WITH_CONSTRAINTS=1
RETAIN_PHYSICAL_SEPARATION=0
ARCHITECTURAL_BLOCKERS=0

CASE_IDENTITY_PRESERVED_BY_ALLOWED_GROUPING=YES
FRESH_PROCESS_PER_CASE_PRESERVED=YES
PROJECT_TREE_MULTI_IDENTITY_RECONCILED=YES

NEW_TEST_TOOL_ARCHITECTURE_REQUIRED=NO
SEMANTIC_GROUPING_REOPENED=NO
PHYSICAL_PATH_AUDIT_PERFORMED=NO
IMPLEMENTATION_PERFORMED=NO

PHASE_D_INPUT=ESTABLISHED
TOOL009F_IMPLEMENTATION_READY=NO
```

Phase C was investigation-only. No Protos files were modified and no builds,
tests, or programs were run.


## TOOL009-F-D physical-path coupling checkpoint

Phase D / #698 completed the static physical-source coupling audit at the same
unchanged Protos revision
`2b3a88389da7228caed231a90b14091cf2841115`,
version `0.3.77-SNAPSHOT`. `origin/main` had not advanced from the Phase-C
baseline.

The audit confirmed that physical source identity is normally locator/plan data,
not Logical Case identity. Consolidation therefore generally requires bounded
manifest/plan/reference rewrites rather than Test Tool architecture changes.

The material coupling mechanisms are:

- manifest/TestPlan source paths carried through `CaseSpec`,
  `sourceAssociation`, discovery, and rematerialization;
- corpus-root confinement in
  `ProtosTestToolFileSelectionFacility.resolveAuthorizedSource()`;
- exact repository-explicit library membership in
  `protos/tools/test/RepositoryCorpusPlans.protos`;
- direct-file relative-import semantics, which are potentially path-sensitive
  but were not found to block the established consolidation candidates;
- Actor/Group exact `workers` overlays at
  `protos/tests/conformance/actor/modules/workers.protos` and
  `protos/tests/conformance/group/modules/workers.protos`;
- Package Tool project-tree fixture authority, where `fixtures/<fixture>` and
  the separate project identity are both semantically consumed; and
- bounded Java/Protos/tooling assertions that name current fixture/source paths.

Per-family physical classification:

```text
PROCESS_SNAPSHOT=SAFE_WITH_REWRITE
ACTOR=GROUP_WITH_PATH_CONSTRAINTS
GROUP=GROUP_WITH_PATH_CONSTRAINTS
PACKAGE_TOML=SAFE_WITH_REWRITE
PACKAGE_CASE_OUTCOMES=SAFE_WITH_REWRITE
PACKAGE_PROJECT_TREE=GROUP_WITH_PATH_CONSTRAINTS
OVERLAY_BOOTSTRAP=GROUP_WITH_PATH_CONSTRAINTS
```

No Test candidate was found whose current filename itself is semantically
required, and no candidate requires permanent one-Test-per-file separation.

The project-tree constraint remains the decisive special case. For example,
`protos/tests/package-tool/execution-plan/manifest.tsv` references
`f2e3b-build-v2-error.protos` through 15 distinct project identities. Its
current matrix is:

```text
15 project-authority CaseSpecs × 1 Test selector = 15 Logical Cases
```

Any future consolidated project-tree source containing N selectors and attached
to M project-authority CaseSpecs is valid only when all intended M × N
combinations are exactly the desired Test Plan. Manifest rows must not be
collapsed merely because they share a source fixture.

Implementation-boundary obligations established by Phase D include:

- rewrite affected manifest/source rows rather than retaining obsolete source
  aliases;
- update repository-explicit library plans and their exact-membership tests;
- update bounded Java/Protos/tooling references to renamed or merged sources;
- keep resulting sources inside their registered corpus roots;
- retain Actor/Group workers overlays unless the host overlay wiring is
  deliberately and separately rewritten;
- preserve Package project-tree authority/cases trees and CaseSpec
  multiplicity; and
- never merge across incompatible corpus, ExecutionRequirement, namespace, or
  project-authority boundaries.

Phase-D result, mapped to #698's coordination vocabulary:

```text
TOOL009F_PHASE_D=COMPLETE
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

SEMANTIC_GROUPS_AUDITED=7

SAFE_TO_MOVE_FAMILIES=0
MOVE_WITH_REFERENCE_UPDATES_FAMILIES=3
PATH_SENSITIVE_FAMILIES=4
BLOCKED_FAMILIES=0
LOGICAL_CASE_IDENTITY_BLOCKERS=0

MANIFEST_COUPLINGS=6
FILENAME_COUPLINGS=5
DIRECTORY_COUPLINGS=7
IMPORT_MODULE_COUPLINGS=1
FIXTURE_COUPLINGS=1
OBSERVABLE_PATH_COUPLINGS=1
TOOLING_COUPLINGS=7

PROJECT_TREE_AUTHORITY_CONSTRAINT_PRESERVED=YES
ALL_ALLOWED_GROUPS_PRESERVE_LOGICAL_CASE_MATRIX=YES
NEW_TEST_TOOL_ARCHITECTURE_REQUIRED=NO

SEMANTIC_GROUPING_REOPENED=NO
IMPLEMENTATION_PERFORMED=NO

PHASE_E_INPUT=ESTABLISHED
TOOL009F_IMPLEMENTATION_READY=NO
```

The remaining work is intentionally Phase E rather than another physical-path
investigation. Phase B deliberately deferred the exact repository-wide target
source map; Phase E must now assign every established semantic family to exact
target source path(s), applying the Phase-C execution constraints and Phase-D
path constraints, and explicitly prove every proposed project-tree
selector × authority matrix.

Phase D was investigation-only. No Protos repository files were modified and no
builds, tests, or programs were run.


## Phase-E baseline contradiction and Phase-A inventory correction

Phase E (`TOOL009-F-E / #699`) rechecked the already-recorded discovery model
against the Phase-A case-count checkpoint before constructing the exact target
map. That reconciliation found a concrete inconsistency in the Phase-A
inventory, at the same unchanged Protos revision:

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
VERSION=0.3.77-SNAPSHOT
```

The authoritative discovery path is:

```text
Discovery.declarationSignatureFromModule(module)
    -> module.slotValue("tests")
    -> Discovery.declarationSignature(tests)

LogicalCasePlan.build(sourceAssociation, signature)
    -> one Logical Case per selector in that source-local tests Array
```

Therefore a `Test(...)` value created inside the body Closure of another Test
is ordinary runtime data for that outer Test. It is not part of the module's
source-local `tests` declaration Array and is not independently discoverable as
a Logical Case.

Phase A had mechanically counted textual/nested `Test(...)` constructions as
additional Logical Cases in six ordinary sources:

```text
protos/tests/conformance/library/test/test-value-fresh-frozen.protos
    recorded 3 -> actual 1   delta -2

protos/tests/conformance/library/test/test-invalid-name.protos
    recorded 3 -> actual 1   delta -2

protos/tests/conformance/library/test/test-value-surface.protos
    recorded 2 -> actual 1   delta -1

protos/tests/conformance/library/test/test-invocation-exact-result.protos
    recorded 2 -> actual 1   delta -1

protos/tests/conformance/library/test/test-invocation-exact-error.protos
    recorded 2 -> actual 1   delta -1

protos/tests/conformance/library/test/test-body-validation-deferred.protos
    recorded 2 -> actual 1   delta -1
```

The combined overcount is exactly eight Logical Cases.

Two genuine multi-Case sources remain:

1. `protos/tests/conformance/control/future-detach-removed-semantics.protos`
   declares seven entries directly in its module `tests` Array.
2. `protos/tests/package-tool/execution-plan/fixtures/f2e3b-build-v2-error.protos`
   declares one Test selector but is referenced by 15 distinct project-tree
   authorities, producing 15 independently scheduled Logical Cases.

The corrected current baseline is therefore:

```text
CURRENT_TEST_SOURCE_FILES=1230
CURRENT_LOGICAL_CASES=1250

SINGLE_CASE_SOURCE_FILES=1228
MULTI_CASE_SOURCE_FILES=2

ORDINARY_TEST_SOURCES=895
ORDINARY_LOGICAL_CASES=901

SPECIAL_EXECUTION_TEST_SOURCES=335
SPECIAL_EXECUTION_LOGICAL_CASES=349
```

Execution-requirement reconciliation:

```text
ordinary          895 sources / 901 cases
process-snapshot   15 sources /  15 cases
actor              11 sources /  11 cases
group              10 sources /  10 cases
package           299 sources / 313 cases
                               -----------
                                  1250 cases
```

This correction supersedes the earlier Phase-A numeric claims:

```text
IN_SCOPE_LOGICAL_CASES=1258
SINGLE_CASE_SOURCE_FILES=1222
MULTI_CASE_SOURCE_FILES=8
ORDINARY_LOGICAL_CASES=909
```

wherever those values are repeated in this checkpoint or inherited by later
phase summaries.

The qualitative conclusions of Phases B-D are not reopened by this correction:

- semantic grouping remains based on coherent behavioral contracts;
- specialized execution families remain groupable under the established constraints;
- project-tree consolidation still requires exact selector × authority matrix preservation;
- physical-path rewrites remain implementation obligations rather than Logical Case identity blockers; and
- no new Test Tool architecture or language/library decision is introduced by this correction.

However, any Phase-E target map and any later before/after reconciliation must
use the corrected 1,250-case baseline. Phase E cannot truthfully establish
`CURRENT_LOGICAL_CASES=1258` or `PROPOSED_LOGICAL_CASES=1258`.

Coordination consequence:

```text
TOOL009F_PHASE_A=REOPENED_FOR_BOUNDED_INVENTORY_CORRECTION
TOOL009F_PHASE_E=BLOCKED_ON_CORRECTED_PHASE_A_CHECKPOINT
TOOL009F_PHASE_F=BLOCKED
```

The next bounded work is investigation-only: repair the Phase-A inventory using
the actual discovery declaration boundary, reconcile the exact per-source
inventory/counts, and republish the corrected Phase-A checkpoint. It must not
redo Phase-B semantic grouping, Phase-C execution-family analysis, or Phase-D
path-coupling analysis.

No Protos repository files were modified and no builds, tests, benchmarks, or
programs were run as part of this correction.
