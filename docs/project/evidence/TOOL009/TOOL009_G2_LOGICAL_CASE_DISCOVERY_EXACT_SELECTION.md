# TOOL009-G2 — Logical Case discovery and exact selection implementation

Date: 2026-10-05

## Identity

~~~text
WORK_ITEM=TOOL009-G
SLICE=TOOL009-G2
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=f9c8b0a3127e74865440c1f0602825e2a071a83c
COMMIT_SUBJECT=TOOL009-G2: expose logical Case discovery and exact selection
IMPLEMENTATION_VERSION=0.3.207-SNAPSHOT
DECISION_AUTHORITY=D185/guillermomolina/protos#799
PARENT_ISSUE=guillermomolina/protos#798
PRIMARY_DOWNSTREAM_CONSUMER=BUG016/guillermomolina/protos#797
~~~

This is durable non-normative implementation evidence. It does not redefine
Protos language or Standard Library semantics and does not replace the live
GitHub Issue state.

## Published implementation

At the exact product revision above, TOOL009-G2 changes:

~~~text
CHANGELOG.md
pom.xml
protos/tools/test/CaseRef.protos
protos/tools/test/CaseSelection.protos
protos/tools/test/Main.protos
protos/tools/test/Options.protos
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolCaseSelectionPublicIntegrationTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolCaseSelectionTest.java
~~~

The implementation is confined to the Test Tool and tests plus required
implementation-version publication metadata.

## D185 CaseRef V1

The published V1 CaseRef spelling is frozen by tests as:

~~~text
"v1." + lowercase_hex(
    UTF8(
        compact_std_JSON(
            [sourceAssociation, selector]
        )
    )
)
~~~

The complete inert D153 source association is encoded, including any
authority-distinguishing project-tree CaseAuthority descriptor.

The V1 implementation therefore preserves:

~~~text
CASE_IDENTITY=AUTHORITATIVE_SOURCE_IDENTITY_PLUS_D153_LOCAL_SELECTOR
PHYSICAL_ABSOLUTE_PATH_IN_IDENTITY=NO
DECLARATION_POSITION_IN_IDENTITY=NO
DECLARATION_SIGNATURE_IN_IDENTITY=NO
EXECUTION_REQUIREMENT_IN_IDENTITY=NO
SHARD_ID_IN_IDENTITY=NO
RUN_ID_IN_IDENTITY=NO
~~~

Consumers do not decode CaseRefs. Selection validates the public syntax and then
compares supplied refs exactly against CaseRefs derived from already-discovered
authoritative CasePlan entries.

~~~text
CASE_REF_CREATES_AUTHORITY=NO
CASE_REF_LOOKS_UP_EXISTING_AUTHORITY=YES
~~~

The product tests freeze both an ASCII example and a Unicode example and prove
that source-authority data omitted by the human display still changes CaseRef.

## Public discovery

The Test Tool now exposes:

~~~text
protos test --list-cases
~~~

The published machine-readable schema is:

~~~json
{
  "schema": "protos.test.cases/v1",
  "cases": [
    {
      "ref": "<canonical CaseRef>",
      "display": "<presentation only>"
    }
  ]
}
~~~

The listing is built from the normal source-scoped D153 discovery result and
preserves authoritative logical CasePlan order.

The Main boundary returns the listing before progress-state creation,
LogicalCaseRunner scheduling, fresh Case Process creation, or Test body
invocation.

~~~text
DISCOVERY_EXECUTES_DECLARATION_CODE=YES
DISCOVERY_EXECUTES_TEST_BODIES=NO
LISTING_MACHINE_READABLE=YES
LISTING_ORDER=AUTHORITATIVE_LOGICAL_CASEPLAN_ORDER
DISPLAY_IS_AUTHORITY=NO
~~~

## Exact selection

The Test Tool now also exposes repeatable:

~~~text
protos test --case CASE_REF
~~~

Selection is exact-only.

~~~text
PATTERN_SELECTION=NO
DISPLAY_SELECTION=NO
PREFIX_SELECTION=NO
~~~

The implementation retains existing CasePlan entries rather than reconstructing
them.

Requested refs are validated before scheduling. The published fail-closed cases
include:

~~~text
MALFORMED_CASE_REF=FAIL
UNSUPPORTED_CASE_REF_VERSION=FAIL
DUPLICATE_EXACT_CASE_REF=FAIL
UNKNOWN_OR_STALE_CASE_REF=FAIL
CASE_REF_OUTSIDE_ACTIVE_SOURCE_SCOPE=FAIL
CANONICAL_CASE_REF_COLLISION=TOOL_AUTHORITY_ERROR
~~~

Repeated requested CaseRefs are rejected rather than silently deduplicated.

When callers request Cases in a different order from the CasePlan, selected
entries retain their original relative CasePlan order.

## Scope composition

Existing source selectors retain their existing behavior.

Exact Case selection is applied after source scoping:

~~~text
repository/package/project scope
    INTERSECT
existing --file / --directory source scope
    INTERSECT
explicit exact --case set
~~~

A CaseRef outside the active file scope fails before scheduling.

The same exact-selection validation is available with --list-cases, so listing a
selected subset uses the same authoritative path rather than a second discovery
authority.

## Existing execution authorities preserved

TOOL009-G2 does not change the Test Tool scheduler or Case execution ownership.

Selected Cases remain the original CasePlan entries and continue through the
existing execution route.

~~~text
D152_FLAT_LOGICAL_CASE_PLAN=PRESERVED
D153_DECLARATION_SIGNATURE=PRESERVED
D153_FRESH_PROCESS_REMATERIALIZATION=PRESERVED
EXECUTION_REQUIREMENT_AUTHORITY=PRESERVED
RESOURCE_CATALOG_AUTHORITY=PRESERVED
CASE_AUTHORITY=PRESERVED
SOURCE_ASSOCIATION=PRESERVED
JOBS_SEMANTICS=PRESERVED
LOGICAL_CASE_RUNNER_POLICY=PRESERVED
CASE_PRIVATE_OUTPUT=PRESERVED
~~~

No TEST009-, Graal- or Truffle-specific concept is added to the public Test Tool
contract.

## Regression evidence

The published implementation adds focused unit and public-integration coverage
for:

~~~text
frozen V1 CaseRef spelling
Unicode CaseRef inputs
source identity + selector identity
authority-distinguishing sourceAssociation data
display/identity separation
malformed and unsupported refs
duplicate exact refs
unknown/stale refs
CaseRef collision fail-closed behavior
listing order
listing without Test body invocation
exact one-Case selection
exact multi-Case selection
CasePlan-order preservation
--file + --case in-scope intersection
--file + --case out-of-scope rejection
--list-cases + --case
public CLI stdout/stderr/exit behavior
~~~

The project owner reported after publication:

~~~text
PRODUCT_PUBLICATION=PUSHED
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
MAINTAINER_REPORT="Todos los tests han pasado en local"
~~~

This durable record does not invent individual command output or remote CI
evidence that was not supplied in the handoff.

## TOOL009-G closure assessment

The #798 acceptance direction is satisfied at this product revision:

~~~text
AUTHORITATIVE_CASE_DISCOVERY_WITHOUT_BODY_EXECUTION=PASS
EXACT_CASE_SUBSET_SELECTION=PASS
MULTI_CASE_SOURCE_EXTERNAL_PARTITION_PRIMITIVE=PASS
INVALID_AMBIGUOUS_STALE_FAIL_CLOSED=PASS
FILE_DIRECTORY_COMPOSITION=PASS
FRESH_PROCESS_AND_AUTHORITY_PRESERVATION=PASS
JOBS_AND_DETERMINISTIC_REPORTING_PRESERVATION=PASS
D185_EXPLICIT_APPROVAL_AND_RATIFICATION=PASS
LOCAL_FULL_VALIDATION=PASS
LANGUAGE_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_SEMANTIC_CHANGE=NO
~~~

TOOL009-G itself does not own process sharding. The next consumer is BUG016,
which may now partition authoritative CaseRefs across independent Test Tool/JVM
invocations without parsing Protos test sources.

~~~text
TOOL009_G=COMPLETE
TOOL009_G_NEXT_SLICE=NONE
BUG016_TEST_TOOL_PREREQUISITE=SATISFIED
~~~
