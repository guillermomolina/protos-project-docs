# D185 — Test Tool logical Case identity, discovery, and exact-selection contract

Status: **RATIFIED — Candidate C′ selected**

Allocated: **2026-10-05**

Explicit project-owner approval: **2026-10-05**

Decision issue: `guillermomolina/protos#799`

Parent work item: `TOOL009-G / guillermomolina/protos#798`

Primary consumer: `TOOL009-G / guillermomolina/protos#798`

Secondary consumer: `BUG016 / guillermomolina/protos#797`

Decision baseline: `guillermomolina/protos@564dc97aacb593826011a8876554d69dd6529faa`

Nature: implementation-independent Test Tool identity/discovery/selection contract.

Normative language effect: **none**.

## Decision

D185 selects **Candidate C′ — versioned structured Case reference + opaque
transport + separate display**.

A logical Case has an authority-preserving identity formed from:

~~~text
CaseIdentity =
    authoritative logical SourceIdentity
    + D153 local selector
~~~

The Test Tool exposes that identity through one canonical, deterministic,
versioned textual `CaseRef`. Consumers MUST treat the CaseRef as opaque: they may
store, compare, partition and pass it back to the Test Tool, but they do not
derive execution authority by parsing or reconstructing its components.

Human presentation is a separate non-authoritative surface:

~~~text
Case identity
    !=
CaseRef transport syntax
    !=
display reference
~~~

This prevents the current presentation form or the current internal
`sourceAssociation` representation from becoming an accidental permanent public
grammar.

## Approval provenance

The project owner explicitly approved D185-C′ in the active decision interaction
on 2026-10-05:

> apruebo C'.

The antecedent was explicit and singular: D185-C′ as recommended by the complete
D185 investigation.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
D185_SELECTED_CANDIDATE=
    C_PRIME_VERSIONED_STRUCTURED_CASE_REFERENCE_WITH_OPAQUE_TRANSPORT
~~~

## Fixed upstream authority

D185 preserves the already-ratified D152 and D153 architecture.

D152 continues to define:

~~~text
LOGICAL_UNIT=NAMED_INDEPENDENTLY_EXECUTABLE_CASE
FILE_IS_CASE_ID=NO
SOURCE_CAN_HAVE_N_CASES=YES
DISCOVERY_BEFORE_SCHEDULING=YES_INITIAL
PLAN_SHAPE=FLAT_INERT_CASE_PLAN
JOBS=GLOBAL_LOGICAL_CASE_CAPACITY
SCHEDULER=WORK_CONSERVING
REPORT_ORDER=LOGICAL_PLAN_ORDER
COMPLETION_ORDER_IS_SEMANTIC=NO
ISOLATION_DEFAULT=FRESH_SEMANTIC_PROCESS_PER_CASE
STDOUT_STDERR=CASE_PRIVATE
~~~

D153 continues to define:

~~~text
DECLARATION_SHAPE=FINITE_CANONICALLY_ORDERED_LOGICAL_ENTRIES
LOCAL_CASE_SELECTOR_KIND=NONEMPTY_STRING
LOCAL_CASE_SELECTOR_UNIQUE_WITHIN_SOURCE=YES
CASEPLAN_RETAINS_INERT_SELECTOR=YES
CASEPLAN_RETAINS_SOURCE_ASSOCIATION=YES
EXECUTION_PROCESS=FRESH_SEMANTIC_PROCESS_PER_CASE
EXECUTION_DECLARATION_SIGNATURE_MUST_MATCH_DISCOVERY=YES
SELECTED_BODY_RESOLUTION=EXACTLY_ONE_BY_LOCAL_SELECTOR
~~~

D185 fills the one boundary D153 deliberately left open: the external/global
logical Case reference and discovery/exact-selection contract.

## Case identity

The semantic identity needed by the Test Tool is:

~~~text
CaseIdentity =
    (
        authoritative logical SourceIdentity,
        D153 LocalSelector
    )
~~~

The source component is Test Tool authority, not a user-supplied host pathname.

The following are explicitly **not** part of Case identity:

~~~text
absolute physical host path
declaration position
D153 declaration signature
ExecutionRequirement
resource reservation/catalog state
Case execution worker/process
jobs lane
shard index
run/attempt identity
completion order
live Test/body Closure identity
body/content hash
~~~

A CaseRef never creates execution authority from user-controlled fields.

The authoritative rule is:

~~~text
CaseRef
    -> exact lookup in the already-authoritative discovered CasePlan
    -> existing CasePlan entry
~~~

and never:

~~~text
CaseRef fields
    -> synthesize a new sourceAssociation/execution request
~~~

This preserves the existing ExecutionRequirement, resource/catalog,
CaseAuthority and source-association owners.

## Stability boundary

D185 does not create a persistent historical test-lineage institution.

### Physical source movement

If physical resolution changes while the authoritative logical SourceIdentity
remains unchanged, the CaseRef remains unchanged.

If the authoritative logical source association itself changes, the logical Case
has a new CaseRef and the old reference becomes stale.

### Test rename

Changing the D153 local selector changes Case identity.

~~~text
Test("before", ...)
    ->
Test("after", ...)

=> new CaseRef
~~~

### Declaration reorder

Reordering declarations does not change Case identity or CaseRef.

It may change the D153 declaration signature and the authoritative
planning/reporting order.

### Body edit

Changing the Test body while preserving authoritative SourceIdentity and local
selector does not change CaseRef.

Therefore CaseRef is not a content-addressed code identity or cache key.

### Repository revisions

A CaseRef is not promised to survive arbitrary repository revisions.

Within a supported CaseRef version:

~~~text
same authoritative logical SourceIdentity
+
same D153 local selector
    =>
same canonical CaseRef
~~~

## CaseRef transport contract

The external CaseRef is:

~~~text
VERSIONED=YES
CANONICAL=YES
DETERMINISTIC=YES
TEXTUAL_SCALAR=YES
SHELL_SAFE=YES
OPAQUE_TO_CONSUMERS=YES
REVERSIBLE_OR_EXACTLY_RESOLVABLE_BY_TEST_TOOL=YES
~~~

The payload encoding is intentionally left as bounded implementation freedom for
TOOL009-G, because consumers are forbidden to parse it. The implementation must
select one deterministic collision-free canonical V1 encoding consistent with
this record and freeze that encoding with tests when V1 is first published.

Once V1 is published, changing its canonical byte/text encoding is a
compatibility change. A later incompatible encoding must use a new CaseRef
version; an old token must never be silently reinterpreted with new semantics.

A spelling family such as `pcase1_<opaque-payload>` is compatible with this
decision, but this record does not require that exact delimiter or payload codec.

## Display reference

The Test Tool may expose a human-readable display reference, including a form
similar to the current:

~~~text
corpus:source::selector
~~~

but display is presentation metadata only.

~~~text
DISPLAY_IS_AUTHORITY=NO
DISPLAY_IS_SELECTOR=NO
DISPLAY_STABILITY_PROMISE=NO
DISPLAY_PARSE_CONTRACT=NO
~~~

The current lifecycle display is especially unsuitable as identity because
current `sourceAssociation` may carry additional authority-distinguishing data
that the display does not include.

## Authoritative discovery

D185 reuses D153 discovery; it does not add source parsing or a second registry.

Conceptually:

~~~text
normal authoritative TestPlan/corpus resolution
    ->
D153 authority-free declaration discovery
    ->
flat ordered logical CasePlan
    ->
derive CaseRef for every authoritative Case
~~~

Discovery executes ordinary D153 declaration code because D153 already selected
that architecture. It does **not** execute Test bodies.

~~~text
DISCOVERY_EXECUTES_DECLARATION_CODE=YES
DISCOVERY_EXECUTES_TEST_BODIES=NO
EXTERNAL_SOURCE_PARSER_AUTHORITY=NO
AMBIENT_CASE_REGISTRY=NO
~~~

## Machine-readable listing

Machine-readable discovery is part of the baseline contract.

The public command spelling selected by D185 is:

~~~text
protos test --list-cases
~~~

It composes with the normal Test Tool corpus/package/project source scopes.

The minimum V1 listing contract contains:

~~~text
schema/version discriminator
ordered cases array
for each Case:
    canonical CaseRef
    presentation-only display reference
~~~

A JSON representation with conceptual fields:

~~~text
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

is the selected baseline shape. TOOL009-G may add only clearly additive,
non-authoritative metadata without changing the D185 identity contract.

Baseline discovery does not need tags, fixtures, duration estimates,
dependencies, worker affinity, shard assignment, retry state, historical
statistics or body hashes.

Listing stops before scheduling or Test body execution.

## Exact selection

The public exact-selection spelling selected by D185 is:

~~~text
protos test --case CASE_REF
~~~

`--case` is repeatable to select an explicit exact set.

~~~text
protos test --case C1 --case C2 --case C3
~~~

The baseline is exact-only.

~~~text
CASE_PATTERN_FILTER_BASELINE=NO
CASE_PREFIX_MATCH_BASELINE=NO
CASE_DISPLAY_MATCH_BASELINE=NO
~~~

A future human-oriented pattern/filter surface may be added as a front-end that
projects to an exact discovered Case set. It must not replace CaseRef authority.

## Selection composition

The selected composition rule is:

~~~text
upstream repository/package/project scope
    INTERSECT
source scope from --file / --directory, when present
    INTERSECT
explicit exact --case set, when present
~~~

Existing source selectors retain their established behavior:

~~~text
multiple --file / --directory selectors
    -> existing union/dedup semantics

--file FILE
    -> all authoritative Cases associated with FILE
~~~

Case selectors form an explicit exact set.

~~~text
--file F --case C
    -> C must be an authoritative Case inside the active F scope
    -> execute C only
~~~

A CaseRef cannot escape an active repository/package/project/file/directory
authority scope.

## Exact-selection failure behavior

All CaseRef validation completes before Case scheduling.

~~~text
MALFORMED_CASE_REF=FAIL
UNSUPPORTED_CASE_REF_VERSION=FAIL
DUPLICATE_EXACT_CASE_REF=FAIL
UNKNOWN_OR_STALE_CASE_REF=FAIL
CASE_REF_OUTSIDE_ACTIVE_SCOPE=FAIL
AMBIGUOUS_OR_COLLIDING_CANONICAL_REF=TOOL_AUTHORITY_ERROR
SCHEDULE_ON_SELECTOR_FAILURE=NO
~~~

A duplicate explicit CaseRef is an input error rather than silently deduplicated;
this preserves exact-once tooling safety.

Without a persistent history database, the Test Tool does not pretend to
distinguish "never existed" from "existed in an older revision". Both fail
deterministically as not-found/stale.

## Ordering

Discovery exposes the authoritative logical CasePlan order.

Selection does not make CLI argument order semantic.

For example:

~~~text
CasePlan = C1 C2 C3
arguments = --case C3 --case C1

selected logical order = C1 C3
~~~

Physical completion remains unconstrained by that order.

~~~text
DISCOVERY_ORDER=AUTHORITATIVE_LOGICAL_CASEPLAN_ORDER
SELECTED_EXECUTION_PLAN_ORDER=RELATIVE_CASEPLAN_ORDER
REPORT_ORDER=RELATIVE_CASEPLAN_ORDER
COMPLETION_ORDER_IS_SEMANTIC=NO
~~~

## Execution and authority preservation

Case-level selection changes set membership only.

After a CaseRef resolves to an existing CasePlan entry, the current execution
pipeline remains authoritative:

~~~text
selected existing CasePlan entry
    ->
existing source association
    ->
existing ExecutionRequirement resolution
    ->
existing specialized logicalCaseExecutionAsync route
    ->
existing CaseAuthority/resource provisioning
    ->
fresh semantic Process
    ->
D153 exact rematerialization
    ->
selected Test body
~~~

Therefore:

~~~text
EXECUTION_REQUIREMENT_PRESERVED=YES
RESOURCE_CATALOG_AUTHORITY_PRESERVED=YES
CASE_AUTHORITY_PRESERVED=YES
SOURCE_ASSOCIATION_PRESERVED=YES
FRESH_PROCESS_PRESERVED=YES
CASE_PRIVATE_OUTPUT_PRESERVED=YES
JOBS_SEMANTICS_PRESERVED=YES
SCHEDULER_OWNERSHIP_PRESERVED=YES
~~~

## External exact-once partitioning

D185 intentionally separates Case identity from shard policy.

An external harness may:

~~~text
D = authoritative --list-cases result

partition D into S1 ... SM

require:
    union(S1 ... SM) == D
    Si intersection Sj == empty for i != j

invoke independent Test Tool processes with the exact CaseRefs in each Si
~~~

The harness does not parse Protos source and does not reconstruct TestPlan or
CasePlan authority.

Every child invocation validates its exact CaseRefs against its own authoritative
discovery before scheduling.

Exact-once coverage is defined relative to the same repository/configuration
snapshot and scope. D185 does not add RunId, repository revision or shard identity
to Case identity.

~~~text
SHARD_POLICY=OUT_OF_SCOPE
SHARD_INDEX_PART_OF_CASE_IDENTITY=NO
RUN_ID_PART_OF_CASE_IDENTITY=NO
TEST009_OR_GRAAL_CONCEPT_IN_PUBLIC_CONTRACT=NO
~~~

A future distributed runner may carry snapshot/run context orthogonally to
CaseRef.

## Prior-art result

The investigation compared materially different mature runner families:

- pytest node IDs and collect-only versus pattern filtering;
- JUnit Platform structured UniqueId/discovery selectors/display separation;
- GoogleTest listing/filtering and explicit sharding;
- Go hierarchical run filters and dynamic subtest limitations;
- Rust/libtest exact versus substring filtering and listing.

The transferable conclusions were:

~~~text
exact identity != human display
exact identity != pattern filter
discovery != body execution
Case identity != shard assignment
scope selection can compose above exact Case selection
~~~

JUnit Platform provides the strongest precedent for structured identity separated
from display; pytest and Rust show the value of exact selectors distinct from
human-oriented filtering; GoogleTest demonstrates that sharding policy need not
be part of test identity.

Prior art is evidence only and does not override D152/D153.

## Rejected alternatives

### A — human source-qualified logical name as public identity

Rejected because it would promote current source-association/display syntax into
a compatibility grammar. Current source association already has more than one
shape and current display does not represent all authority-distinguishing data.

### B — unrelated opaque UUID / persistent registry identity

Rejected because a random/session identifier is not reusable across invocations,
while a persistent UUID requires a new registry/metadata institution with no
current requirement. A deterministic opaque token derived from the selected
structured identity converges on C′.

### C — directly public structured fields

Architecturally sound but exposes more representation than external consumers
need. C′ retains the structured semantic model while making transport opaque.

### D — name/pattern filtering only

Rejected as underengineered for exact-once partitioning: a filter may match zero,
one or many Cases after repository evolution.

### E — external generated manifest/index authority

Rejected because it creates a second authority and a source/index drift failure
mode. Machine-readable discovery is only a projection of live Test Tool
authority, never an external authority store.

## Pay-for-what-you-need boundary

D185 initially pays for:

~~~text
one canonical CaseRef derivation
one CaseRef lookup layer
one versioned machine-readable listing projection
one repeatable exact selector
fail-closed pre-scheduling validation
~~~

It deliberately does not pay for:

~~~text
persistent Case UUID registry
cross-refactor historical lineage
CaseRef database
generated authority manifest
automatic sharding
duration-aware balancing
remote worker protocol
history store
pattern language
rename aliases
retry/timeout/tags/fixtures
~~~

The versioned opaque transport is justified now because D185 creates a public
machine-consumable boundary. Publishing today's human display grammar as identity
would create an immediate compatibility dependency on internal
source-association representation.

## Future regret and escape path

The main plausible regret scenario is a future requirement for persistent test
lineage across arbitrary Test renames, source moves, source split/merge operations
and long-lived historical dashboards.

D185 V1 does not provide that lineage.

If that requirement becomes concrete, add a separate stable lineage identifier or
a new CaseRef version:

~~~text
CaseRef V1 remains V1
optional StableCaseId may be added as discovery metadata
or
CaseRef V2 may explicitly incorporate the new identity model
~~~

Never reinterpret a V1 token with V2 semantics.

## Intentionally deferred

D185 does not decide or add:

~~~text
persistent cross-refactor StableCaseId
human-oriented pattern filtering
tags
fixtures
retry
timeout
new parameterization semantics
nested Suite semantics
historical test database
automatic shard count or balancing
process pool policy
remote worker topology
run/snapshot identity
general result-file protocol
duration estimation
cache-key policy
TEST009-specific CLI
Graal/Truffle concepts
~~~

The exact canonical V1 payload codec is delegated to TOOL009-G implementation
within the constraints in this record. That implementation choice becomes frozen
compatibility behavior when V1 is first published.

## Invariant / moving-HEAD consistency

The investigation baseline and ratification review baseline are the same product
revision:

~~~text
PROTOS_REVISION=564dc97aacb593826011a8876554d69dd6529faa
COMMIT_SUBJECT=I065: thread Network grant through workspace application execution
MOVING_HEAD_DELTA=NONE
~~~

The current repository state was rechecked against the selected contract.

~~~text
D152_LOGICAL_CASE_UNIT=PASS
D152_FILE_IS_NOT_CASE_ID=PASS
D152_FLAT_INERT_CASEPLAN=PASS
D152_FRESH_PROCESS_PER_CASE=PASS
D152_JOBS_AND_SCHEDULER=UNCHANGED

D153_LOCAL_SELECTOR_AUTHORITY=PASS
D153_ORDERED_DISCOVERY_SIGNATURE=PASS
D153_EXACT_REMATERIALIZATION=PASS
D153_NO_LIVE_CLOSURE_TRANSFER=PASS

EXECUTION_REQUIREMENT_AUTHORITY=UNCHANGED
RESOURCE_CATALOG_AUTHORITY=UNCHANGED
CASE_AUTHORITY=UNCHANGED
SOURCE_ASSOCIATION_AUTHORITY=UNCHANGED

NEW_UNSURFACED_ARCHITECTURAL_CONSEQUENCE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Closure contract

~~~text
D185_STATUS=RATIFIED
D185_SELECTED_CANDIDATE=
    C_PRIME_VERSIONED_STRUCTURED_CASE_REFERENCE_WITH_OPAQUE_TRANSPORT

PROTOS_REVISION=564dc97aacb593826011a8876554d69dd6529faa
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
PROJECT_RECORD_PATH=
    docs/project/decisions/tooling/D185_TEST_TOOL_LOGICAL_CASE_IDENTITY_DISCOVERY_EXACT_SELECTION.md

SPECIFICATION_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
EXECUTABLE_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

D185 ratification authorizes TOOL009-G to implement the selected Test Tool
contract. It does not authorize unrelated Test Tool, TEST009, scheduling,
language or Standard Library changes.
