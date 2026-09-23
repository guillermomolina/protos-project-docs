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
