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

## Current coordination consequence

```text
TOOL009F_ARCHITECTURAL_FEASIBILITY=ESTABLISHED
TOOL009F_PHASE_A=NOT_COMPLETE
TOOL009F_IMPLEMENTATION_READY=NOT_YET_ESTABLISHED
```

The next bounded work remains TOOL009-F-A / #695: produce the complete mechanical source/Logical Case inventory without repeating the architecture investigation recorded here.
