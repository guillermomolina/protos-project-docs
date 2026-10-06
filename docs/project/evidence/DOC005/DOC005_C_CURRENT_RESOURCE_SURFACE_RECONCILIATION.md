# DOC005-C — Current Test Tool Resource-Surface Reconciliation

## Identity

```text
WORK_ITEM=DOC005
ISSUE=guillermomolina/protos#448
SLICE=DOC005-C
SLICE_TYPE=INVESTIGATION_RECONCILIATION
STATUS=COMPLETE_SUPERSEDED
PROTOS_REVISION=fc9aca90f479051964d8d56c11a2778386789671
PROTOS_REVISION_SUBJECT=DOC005-F: document Test Tool exact file-backed focal selection
RECONCILIATION_DATE=2026-10-06
PRODUCT_MUTATIONS=NONE
BUILDS_TESTS_PROGRAMS_EXECUTED=NONE
```

This is durable non-normative evidence for the current-product reconciliation
of DOC005-C. It does not define Test Tool semantics and it does not resurrect
historical TOOL002 resource architecture.

## Result

```text
DOC005_C_RECONCILIATION=COMPLETE
DOC005_C_CURRENT_CLASSIFICATION=C_SUPERSEDED_BY_D152

PUBLIC_GENERIC_RESOURCE_CATALOG=NO
PUBLIC_D077_RESOURCEFUL_CASE_EXECUTION=NO
PRODUCTION_RESOURCE_REQUIREMENTS_FILES=0
PUBLIC_PROVIDER_REGISTRY=EMPTY

D152_GENERIC_RESOURCE_CATALOG_NO_INITIAL=PRESERVED
REPOSITORY_EXECUTION_LOGICAL_ONLY=YES
GLOBAL_PRODUCTION_LEGACY_CASES=0
PUBLIC_D108_REPOSITORY_EXECUTION=NO

RESOURCE_REQUIREMENTS_PARSE_JOIN_REACHABLE=YES
RESOURCE_REQUIREMENTS_EXECUTION_REACHABLE=NO
RESOURCE_BINDING_PUBLIC_PRODUCTION_REACHABLE=NO
RESOURCE_RESERVATION_PUBLIC_PRODUCTION_REACHABLE=NO
RESOURCE_PROVIDER_PUBLIC_PRODUCTION_REACHABLE=NO

DOC005_RESOURCE_GUIDE_STATUS=PARTIALLY_STALE
DOC005_C_IMPLEMENTATION_REQUIRED=NO
DOC005_E_RELEASED=YES
NEW_PRODUCT_ISSUE_REQUIRED=NO
NEXT_TECHNICAL_SLICE=NONE
NEXT_DOC005_SLICE=DOC005-E
NEXT_DOC005_SLICE_TYPE=IMPLEMENTATION_DOCUMENTATION_AND_CLOSURE
NEW_DOC005_SUBISSUE_REQUIRED=NO
```

## Governing architecture

D152 / `guillermomolina/protos#595` is closed and ratified with Candidate C,
`LOCAL_FIRST_FLAT_CASE_PLAN`.

The selected architecture explicitly retains:

```text
CASE_SPECIFIC_ENVIRONMENT=YES_WHEN_DEMONSTRATED
```

while deferring:

```text
GENERIC_RESOURCE_CATALOG=NO_INITIAL
REMOTE_HA_WORKER_MODEL=NO_INITIAL
RETRY_ATTEMPT_MODEL=NO_INITIAL
```

D152 distinguishes demonstrated case-specific execution environment/authority
needs from the not-demonstrated generic scarce-resource
requirements/catalog/provider/locality scheduler institution.

D153 / `guillermomolina/protos#596` subsequently ratified authority-free
logical Case discovery plus symbolic fresh-Process rematerialization. It
preserves later explicit provisioning of a selected Case environment, but it
does not select the historical D077/ResourceCatalog mechanism as that
environment protocol.

TOOL009 / `guillermomolina/protos#600` completed the production migration:

```text
REPOSITORY_EXECUTION_LOGICAL_ONLY=YES
GLOBAL_PRODUCTION_LEGACY_CASES=0
```

The old D108 repository-production branch therefore no longer owns current
`protos test` execution.

## Current Main.protos behavior

Current `protos/tools/test/Main.protos` still parses
`--resource-catalog PATH` and still applies
`ResourceRequirements.loadFromCorpus(...)` to `manifest` and
`package-toml` plans.

That retained parsing/join is not equivalent to current resourceful execution.

Before logical discovery/execution, every repository production Case must be
`suite-native`, and Main fails closed when:

```text
Manifest.caseRequirements(spec).size() != 0
```

The current production path then executes through:

```text
LogicalCaseRunner.run(...)
LogicalCaseDispatch.logicalCaseExecutorAsync(...)
```

and does not call:

```text
Runner.runD108WithResources(...)
```

The retained public-cutover guard
`ProtosTestToolI8D5CPublicCutoverTest` explicitly requires zero
`Runner.runD108WithResources(` occurrences in `Main.protos`.

Therefore the current reachability boundary is:

```text
ResourceRequirements discovery/parsing/attachment = reachable
suite-native D077 resourceful execution           = not reachable
ResourceBinding/ResourceReservation production    = not reachable
D108 repository production                         = not reachable
```

## Resource catalog and provider state

`Main.protos` can acquire and parse a supplied resource catalog, but the
resulting `resourceCatalog` value is not consumed by the logical production
scheduler.

The historical resource runner and resource binding/reservation modules remain
in the repository as lower-level retained machinery and focused evidence. Their
continued existence is not a current public repository-execution contract.

The host resource execution scope is installed with:

```text
ProtosTestResourceProviderRegistry.empty()
```

and no current public installation populates a provider registration set.

A syntactically valid catalog entry naming a provider such as
`device/gpu` therefore does not establish an end-to-end public provider-backed
Case execution path.

## Production sidecar inventory

A complete, non-truncated recursive tree read of
`guillermomolina/protos@fc9aca90f479051964d8d56c11a2778386789671`
found:

```text
resource-requirements.toml=0
```

This preserves the AUD014 finding that no current production corpus declares
D077 resource requirements through the historical sidecar model.

## ExecutionRequirementId and CaseAuthority boundary

Current production does have explicit execution environment/authority
mechanisms.

Repository Suite leaves select a finite `ExecutionRequirementId`, and the
logical dispatcher resolves that identifier to an installed
`logicalCaseExecutionAsync` binding at execution time.

Some Package Tool corpora additionally carry a D133 project-tree
`CaseAuthority` descriptor that provisions `projectTreeFilesystem` for the
same fresh Process as the selected logical Case.

These are current product mechanisms, but they are not a semantic replacement
for:

```text
ResourceCatalog
provider/profile
capacity
shared/exclusive requirements
ResourceReservation
```

DOC005 must not present them as the generic-resource model under another name.

## Documentation drift

The maintained `docs/guide/tools/test-tool.md` resource-status section is now
partially stale.

Still current:

- `ResourceRequirements.loadFromCorpus(...)` is applied to relevant
  manifest-backed plans.
- true sidecar absence preserves a plan without attached D077 requirements.
- present sidecars still pass through the retained acquisition/schema/join
  rules.

No longer current:

- describing the join as feeding D108 repository production scheduling;
- saying DOC005-C should publish a complete resource-backed user recipe;
- saying resource requirements/catalog/provider/profile are the next
  user-facing documentation layer;
- saying TOOL006 currently enables a resource-backed command path;
- saying repository-wide suite/corpus routing remains pending under TOOL005.

TOOL005 / `guillermomolina/protos#468` and TOOL009 are complete.

## DOC005 lifecycle consequence

The historical DOC005-C scope cannot force the product to preserve or restore
the former generic resource architecture.

DOC005-C therefore closes by reconciliation, not by implementation:

```text
DOC005_C=SUPERSEDED_BY_D152
RESOURCE_AWARE_USER_RECIPE=DO_NOT_PUBLISH
DOC005_C_IMPLEMENTATION=NOT_REQUIRED
```

No product issue should be opened merely to make the old documentation slice
implementable.

DOC005-E is now released and should be the one remaining implementation slice
for the parent. It should, in one documentation-only publication:

1. reconcile the stale resource-status prose against D152/TOOL009;
2. remove the promise of a current generic resource-backed recipe;
3. preserve only current product behavior;
4. reconcile stale TOOL005 status/navigation claims;
5. perform the final DOC005 current-surface/link/example consistency review;
6. close DOC005/#448 when the documentation validation required by the work item
   passes.

No additional DOC005-C research slice, DOC005-C2, product TOOL issue, or
intermediate closure-review slice is required unless that implementation finds
new concrete contradictory product evidence.

## Materially inspected product evidence

The reconciliation materially inspected current versions of:

```text
protos/tools/test/Main.protos
protos/tools/test/LogicalCaseRunner.protos
protos/tools/test/LogicalCaseDispatch.protos
protos/tools/test/Manifest.protos
protos/tools/test/ResourceRequirements.protos
protos/tools/test/ResourceCatalog.protos
protos/tools/test/ResourceBinding.protos
protos/tools/test/ResourceReservation.protos
protos/tools/test/Runner.protos
protos/tools/test/Options.protos
protos/tools/test/RepositorySuite.protos

src/main/java/com/guillermomolina/protos/cli/
  ProtosTestToolAsyncExecutionScope.java
  ProtosTestExecutionRequirementRegistry.java

src/main/java/com/guillermomolina/protos/execution/
  ProtosTestResourceExecutionScope.java
  ProtosTestResourceProviderRegistry.java
  ProtosTestResourcefulExecutionFacility.java

src/test/java/com/guillermomolina/protos/cli/
  ProtosTestToolI8D5CPublicCutoverTest.java

docs/guide/tools/test-tool.md
```

Live coordination materially reconciled:

```text
DOC005  #448
TOOL005 #468
TOOL006 #473
AUD014  #594
D152    #595
D153    #596
TOOL009 #600
```

## Validation provenance

This reconciliation was read-only against the product repository. It executed
no builds, tests, programs, Maven, Make, validators, or state-changing product
Git operations.

The durable evidence publication itself is a bounded
`guillermomolina/protos-project-docs` repository-content publication explicitly
authorized by the project owner in the active interaction.

A local `git diff --check` was not executed by this publication path. The
published commit must therefore not be represented as having maintainer-reported
or locally executed validation that did not occur. The final DOC005-E product
documentation implementation retains the normal documentation validation gate.

## Final state

```text
DOC005_C_RECONCILIATION=COMPLETE
PUBLIC_GENERIC_RESOURCE_CATALOG=NO
PUBLIC_D077_RESOURCEFUL_CASE_EXECUTION=NO
PRODUCTION_RESOURCE_REQUIREMENTS_FILES=0
PUBLIC_PROVIDER_REGISTRY=EMPTY
D152_GENERIC_RESOURCE_CATALOG_NO_INITIAL=PRESERVED
DOC005_RESOURCE_GUIDE_STATUS=PARTIALLY_STALE
DOC005_C_IMPLEMENTATION_REQUIRED=NO
DOC005_E_RELEASED=YES
NEW_PRODUCT_ISSUE_REQUIRED=NO
```
