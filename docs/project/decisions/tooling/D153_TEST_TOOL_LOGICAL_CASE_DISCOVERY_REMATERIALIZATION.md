# D153 — Test Tool logical Case discovery and fresh-Process rematerialization

Status: **RATIFIED — Candidate B′ selected**

Allocated: **2026-09-18**

Explicit project-owner approval: **2026-09-18**

Decision issue: `guillermomolina/protos#596`

Upstream architecture: D152 / `guillermomolina/protos#595`

Decision baseline: `guillermomolina/protos@64b21418f19501ff472d03286367b1535fabb3a4`

Nature: implementation-independent Test Tool discovery/rematerialization contract.

Normative language effect: **none**.

## Decision

D153 selects **Candidate B′ — explicit module declaration + authority-free
discovery + symbolic fresh-Process rematerialization**.

The Test Tool discovers suite-native logical Cases by evaluating an ordinary
Protos test source as an **imported module** under a declaration phase that does
not receive Case execution bootstrap authority.

The source intentionally exposes one explicit authoring-owned declaration root.
That declaration yields a finite canonically ordered set of logical entries.

Each entry has a stable non-empty String selector that is unique within the
source declaration.

Discovery may create live body Closures locally, but the CasePlan retains only
inert planning data. No live body Closure crosses the discovery boundary or a
semantic Process boundary.

For execution, the selected source is reconstructed in a fresh semantic Process,
the declaration is rebuilt before Case execution authority is provisioned, the
reconstructed declaration signature is checked against discovery, and exactly
one selected body is resolved by its stable local selector and invoked.

Conceptually:

```text
DISCOVERY
    Tool discovery domain
        |
        | ordinary import
        v
    test source module
        |
        | no Case execution bootstrap authority
        v
    explicit declaration root
        |
        +-- selector A -> live body A
        +-- selector B -> live body B
        +-- selector C -> live body C
        |
        v
    inert declaration signature / CasePlan

    live discovery module + bodies discarded


EXECUTION OF B
    fresh semantic Process
        |
        v
    Tool-owned execution driver
        |
        | import same source before Case authority
        v
    fresh test module instance
        |
        v
    reconstruct declaration
        |
        | signature must match discovery
        v
    resolve exactly selector B
        |
        v
    provision B execution environment
        |
        v
    invoke B only
```

D153 selects this architectural contract, not the public `Test`/`Suite`
authoring spelling or one particular host/runtime carrier.

## Approval provenance

The project owner explicitly approved D153-B′ on 2026-09-18 after the complete
comparative decision packet had been published in the active Issue.

The exact approval was:

> apruebo B′

The antecedent was explicit and singular: D153-B′ — explicit module declaration,
authority-free discovery and symbolic fresh-Process rematerialization.

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Existing Protos mechanisms reused

D153 does not add language semantics to obtain this architecture.

The current language already establishes:

- a module instance is the ordinary module `moduleContext`;
- top-level bindings are local slots of that module instance;
- `import(specifier)` returns that ordinary module instance;
- Closures created during module evaluation capture the module context through
  ordinary lexical capture;
- Closure capture retains the genuine lexical context by reference rather than
  snapshotting slot values;
- imported modules do not automatically receive the RootActor's local
  `process` capability;
- imported modules do not automatically receive the default bootstrap
  `filesystem` capability;
- the standalone CLI-local `print` convenience is not a Core ambient binding;
- mutable module instances and module caches are Actor-local.

D153 uses those existing boundaries. It does not redefine module, Closure,
Process, import, filesystem or CLI semantics.

## Declaration phase

### Ordinary module evaluation

Suite-native discovery executes ordinary Protos declaration code.

```text
DISCOVERY_EXECUTES_DECLARATION_CODE=YES
```

D153 deliberately does **not** select static source/AST extraction.

The test source is evaluated as an imported module, not as the RootActor's normal
Case execution entry.

### Authority-free relative to Case execution

While the declaration signature is materialized, the test source is not supplied
the normal Case execution bootstrap authority.

Initially:

```text
DISCOVERY_BOOTSTRAP_PROCESS_CAPABILITY=NO
DISCOVERY_DEFAULT_FILESYSTEM_CAPABILITY=NO
DISCOVERY_CLI_PRINT_CAPABILITY=NO
CASE_EXECUTION_ENVIRONMENT_DURING_DECLARATION=NO
```

This is not a claim that discovery code is mathematically pure or a hostile-code
security sandbox.

Declaration code may perform ordinary local computation, allocate/mutate its own
module-local objects, create body Closures and import ordinary modules according
to the supplied resolver domain.

Hard CPU/memory/time containment of malicious or non-terminating discovery code
is outside D153.

### Explicit declaration root

Discovery does not scan arbitrary program structure.

```text
DECLARATION_ROOT=EXPLICIT_AUTHORING_OWNED_PROTOCOL
ARBITRARY_MODULE_SLOT_SCAN=NO
TEST_PREFIX_SCAN=NO
AMBIENT_GLOBAL_REGISTRY=NO
```

The source intentionally exposes exactly the authoring-owned declaration
boundary required by the future testing library.

D153 does not select the public spelling of that boundary.

The declaration root may later be represented by `Suite`, an ordinary object,
an ordered collection of `Test` values, a dedicated top-level authoring slot, or
another ordinary library representation selected by LIB018.

## Logical selector and identity

Each logical declaration entry has one stable local selector:

```text
LOCAL_CASE_SELECTOR_KIND=NONEMPTY_STRING
LOCAL_CASE_SELECTOR_UNIQUE_WITHIN_SOURCE=YES
```

That selector identifies the logical entry **within one source declaration**.

It is:

- not a live Closure identity;
- not the physical file path;
- not the whole externally visible CaseId by itself.

The replacement Test Tool combines stable source/logical namespace information
with the local selector to produce collision-free logical Case identity.

D153 does not select the final public CaseId string serialization.

## Declaration shape

The initial declaration is finite and canonically ordered.

```text
DECLARATION_SHAPE=FINITE_CANONICALLY_ORDERED_LOGICAL_ENTRIES
```

Canonical declaration order provides stable planning/presentation evidence and a
stable declaration signature.

This does not make completion order semantic.

## Inert discovery projection

Discovery may hold ordinary live values temporarily while the declaration module
exists.

The durable CasePlan does not.

```text
DISCOVERY_LIVE_BODY_MAY_EXIST_LOCALLY=YES
LIVE_BODY_TRANSFERRED_TO_CASEPLAN=NO
LIVE_BODY_TRANSFERRED_ACROSS_PROCESS=NO

CASEPLAN_RETAINS_INERT_SELECTOR=YES
CASEPLAN_RETAINS_SOURCE_ASSOCIATION=YES
PHYSICAL_FILE_IS_LOGICAL_CASE_ID=NO
```

Only inert planning-affecting metadata leaves discovery.

If later LIB018 features introduce planning-affecting metadata, that metadata
must also be representable inertly and participate in the declaration signature
where necessary.

## Discovery-state isolation

One source's discovery must not depend on mutable module state leaked from
another source's discovery.

```text
MUTABLE_DISCOVERY_MODULE_STATE_SHARED_ACROSS_SOURCES=NO
```

A fresh Actor-local module-cache domain per discovered source is a natural
implementation strategy under current module semantics, but D153 does not freeze
that exact carrier if another implementation provides the same observable
isolation.

Immutable parsed/compiled artifacts may remain physically shared when that
sharing is semantically unobservable.

## Fresh-Process rematerialization

D153 preserves D152's selected isolation:

```text
EXECUTION_PROCESS=FRESH_SEMANTIC_PROCESS_PER_CASE
```

For each selected Case, execution:

1. starts the selected Case's fresh semantic Process;
2. reconstructs/imports the source declaration before supplying Case execution
   environment authority;
3. obtains the same explicit declaration root;
4. reconstructs the declaration signature;
5. validates that signature against the authoritative discovery result;
6. resolves exactly one body by the selected local selector;
7. provisions the selected Case execution environment;
8. invokes only that body.

```text
EXECUTION_REIMPORTS_SOURCE=YES
EXECUTION_RECONSTRUCTS_DECLARATION_BEFORE_AUTHORITY=YES
SELECTED_BODY_RESOLUTION=EXACTLY_ONE_BY_LOCAL_SELECTOR
UNRELATED_BODY_INVOCATION=NO
```

No discovered live Closure is reused.

## Rematerialization consistency

Discovery is authoritative for the already-materialized CasePlan.

Execution does not silently rediscover a new plan.

The reconstructed declaration must reproduce the discovery signature relevant to
the selected Case.

Initially that signature includes at least the canonical ordered local selector
sequence.

If later planning-affecting inert metadata is selected, that metadata becomes
part of the signature as required.

```text
EXECUTION_DECLARATION_SIGNATURE_MUST_MATCH_DISCOVERY=YES
```

A mismatch is a Test Tool/rematerialization error:

```text
MISMATCH_CLASSIFICATION=TOOL_REMATERIALIZATION_ERROR
MISMATCH_INVOKES_BODY=NO
MISMATCH_MUTATES_PLAN=NO
SILENT_REDISCOVERY=NO
```

Examples include:

- a discovered selector disappears;
- an extra selector appears;
- canonical selector ordering changes when ordering is part of the signature;
- a selector becomes duplicated;
- planning-affecting metadata changes;
- the selected selector no longer resolves to exactly one valid body.

## Discovery errors

An invalid declaration fails before scheduling.

```text
DISCOVERY_ERROR_BEFORE_SCHEDULING=YES
DUPLICATE_SELECTOR=DISCOVERY_ERROR
```

A source that cannot produce a valid declaration does not manufacture one
ordinary failed test Case.

The complete initial CasePlan remains authoritative only after successful
discovery.

## Conditional and generated declarations

D153 does not require declarations to be textually static.

Ordinary declaration code may construct entries conditionally or from ordinary
inert data.

The selected rule is reproducibility, not compiler-recognizable syntax.

```text
CONDITIONAL_DECLARATION=ALLOWED_ONLY_IF_REPRODUCIBLE
GENERATED_CASES=ALLOWED_WITH_STABLE_UNIQUE_SELECTORS
```

A condition that depends on unavailable Case execution authority fails rather
than turning that authority into discovery state.

A declaration whose result differs when re-materialized fails closed before body
invocation.

D153 does not introduce a parameterization DSL, Cartesian-product framework,
argument serializer or parameterized-test value category.

## Large generated-suite stress

A straightforward B′ implementation reconstructs the declaration once for
discovery and once for every fresh Case execution.

If declaration generation is O(N), running N generated Cases may therefore spend
O(N²) aggregate declaration-enumeration work.

D153 explicitly accepts that possible cost for the current generation because
the concrete need is ordinary small/medium suite authoring, while avoiding that
cost now through compiler registration, static extraction or a second external
metadata system would impose a much larger present institution.

```text
LARGE_GENERATED_SUITE_OPTIMIZATION=DEFERRED
```

A later concrete performance need may add indexed/lazy declaration
materialization while preserving:

- stable logical selectors;
- fresh Process per Case;
- authority-free declaration phase;
- test-neutral lower execution.

## Case environment provisioning boundary

D153 selects only **when** the future Case execution environment becomes
available relative to declaration reconstruction.

```text
CASE_ENVIRONMENT_PROVISIONING_POINT=
    AFTER_DECLARATION_VALIDATION_BEFORE_BODY
```

The exact environment representation/provider/protocol remains a separate
decision.

This prevents a declaration from requiring the authority whose purpose is to
execute one selected body.

## D151 / TOOL008 compatibility

D151 remains unchanged:

```text
protos test --file FILE
    -> all authoritative logical Cases associated with FILE

FILE_IS_CASE_ID=NO
```

A suite-native source discovered under D153 may therefore own:

```text
math.protos
    -> add-zero
    -> add-positive
    -> add-overflow
```

and file focal selection naturally selects all three authoritative Cases.

A future exact logical selector such as:

```text
protos test --case CASE_ID
```

remains additive and is not selected by D153.

## Relationship to LIB018

D153 deliberately stops before the public authoring API.

It does not select:

- `Suite(...)`;
- `Test(...)`;
- a particular declaration-root slot name;
- a reflection-based slot surface;
- an Array-based surface;
- nested suites;
- body arity;
- TestContext;
- fixtures/hooks;
- parameterization API.

LIB018-0 may now resume and choose the smallest ordinary authoring model that
satisfies this ratified discovery/rematerialization boundary.

An authoring API is compatible with D153 only if it can expose the explicit
declaration root, stable local selectors and reconstructible bodies without
ambient global registration or Java-owned Test/Suite semantics.

## Rejected candidates

### A — capability-bearing replay

Rejected because discovery would unnecessarily receive normal execution authority
and make declaration membership/effects depend on Process, default filesystem or
other Case execution capabilities.

### C — static source/AST extraction

Rejected because it solves discovery effects by creating compiler/parser/Test
Tool knowledge of Test/Suite syntax and makes ordinary testing-library evolution
part of static language tooling.

### D — external sidecar / inert manifest

Rejected as the default suite-native authoring model because it duplicates source
and metadata and recreates the maintenance split LIB018 exists to remove.

It remains a possible explicit input format for other use cases.

### E — ambient dynamic registration

Rejected because it introduces hidden mutable registration state and non-local
declaration ownership. An explicit ordinary builder that simply returns a
declaration root converges architecturally on B′.

### F — shared discovery/execution Process

Rejected because it directly violates D152's explicitly selected fresh semantic
Process per logical Case.

### G — defer / legacy source-per-case only

Rejected because the project now has a concrete one-source/N-logical-Case need
and D152 explicitly selected that future shape.

## Comparative score

The current-AGENTS informational scorecard was:

| Candidate | Score / 60 |
| --- | ---: |
| A — full-authority replay | 44 |
| **B′ — explicit module declaration/rematerialization** | **57** |
| C — static extraction | 42 |
| D — sidecar | 50 |
| E — ambient registration | 48 |
| G — defer | 42 |

Candidate F was eliminated before scoring because it directly contradicted
ratified D152 isolation.

The score did not select the result by itself. B′ survived because it preserves
ordinary Protos authoring, exact fresh-Process isolation and deterministic
rematerialization without installing privileged testing syntax, an ambient
registry or external duplicate metadata.

## Prior-art evidence

D153 compared materially different architectures including:

- SUnit / Smalltalk / Pharo;
- Io UnitTest;
- Go `testing`;
- Rust/libtest;
- pytest;
- JUnit Platform;
- ExUnit;
- Swift Testing;
- Node's built-in test runner.

The strongest transferable precedent is the separation:

```text
stable symbolic logical identity
    !=
live executable object
```

SUnit's symbolic selector model is a particularly close historical precedent:
logical test identity can survive while the executable receiver/body is
reconstructed locally.

pytest supplies mature evidence that ordinary module evaluation can participate
in discovery, while Protos' explicit bootstrap authority permits a narrower
declaration phase than a normal test execution.

Compiler/macro registration systems such as Rust, ExUnit and Swift demonstrate
the operational strength of static identities but were rejected as
disproportionate for Protos' ordinary-library direction.

## Pay-for-what-you-need boundary

D153 initially pays for:

- one explicit declaration root;
- finite ordered logical entries;
- stable local String selectors;
- inert CasePlan projection;
- source re-materialization in each fresh Process;
- declaration consistency checking.

It does not pay for:

- Test compiler annotations/macros;
- AST Test nodes;
- ambient registration engine;
- fixture graph;
- tags/traits;
- retry;
- timeout;
- remote workers;
- resource graph;
- parameterization framework;
- persistent test database;
- plugin/engine hierarchy.

## Exact selected contract

```text
D153_SELECTED_CANDIDATE=
    B_PRIME_EXPLICIT_MODULE_DECLARATION_REMATERIALIZATION

DISCOVERY_KIND=ORDINARY_PROTOS_MODULE_EVALUATION
DISCOVERY_TEST_SOURCE_ROLE=IMPORTED_MODULE

DISCOVERY_BOOTSTRAP_PROCESS_CAPABILITY=NO
DISCOVERY_DEFAULT_FILESYSTEM_CAPABILITY=NO
DISCOVERY_CLI_PRINT_CAPABILITY=NO
CASE_EXECUTION_ENVIRONMENT_DURING_DECLARATION=NO

DISCOVERY_SOURCE_MUTABLE_STATE_ISOLATED=YES
MUTABLE_MODULE_INSTANCE_SHARING_ACROSS_SOURCE_DISCOVERIES=NO

DECLARATION_ROOT=EXPLICIT_AUTHORING_OWNED_PROTOCOL
ARBITRARY_MODULE_SLOT_SCAN=NO
TEST_PREFIX_SCAN=NO
AMBIENT_GLOBAL_REGISTRY=NO

DECLARATION_SHAPE=FINITE_CANONICALLY_ORDERED_LOGICAL_ENTRIES
LOCAL_CASE_SELECTOR_KIND=NONEMPTY_STRING
LOCAL_CASE_SELECTOR_UNIQUE_WITHIN_SOURCE=YES

DISCOVERY_LIVE_BODY_MAY_EXIST_LOCALLY=YES
LIVE_BODY_TRANSFERRED_TO_CASEPLAN=NO
LIVE_BODY_TRANSFERRED_ACROSS_PROCESS=NO

CASEPLAN_RETAINS_INERT_SELECTOR=YES
CASEPLAN_RETAINS_SOURCE_ASSOCIATION=YES
PHYSICAL_FILE_IS_LOGICAL_CASE_ID=NO

EXECUTION_PROCESS=FRESH_SEMANTIC_PROCESS_PER_CASE
EXECUTION_REIMPORTS_SOURCE=YES
EXECUTION_RECONSTRUCTS_DECLARATION_BEFORE_AUTHORITY=YES
EXECUTION_DECLARATION_SIGNATURE_MUST_MATCH_DISCOVERY=YES

MISMATCH_CLASSIFICATION=TOOL_REMATERIALIZATION_ERROR
MISMATCH_INVOKES_BODY=NO
MISMATCH_MUTATES_PLAN=NO
SILENT_REDISCOVERY=NO

SELECTED_BODY_RESOLUTION=EXACTLY_ONE_BY_LOCAL_SELECTOR
UNRELATED_BODY_INVOCATION=NO

CASE_ENVIRONMENT_PROVISIONING_POINT=
    AFTER_DECLARATION_VALIDATION_BEFORE_BODY
CASE_ENVIRONMENT_PROTOCOL=DEFERRED_SEPARATE_DECISION

DISCOVERY_ERROR_BEFORE_SCHEDULING=YES
DUPLICATE_SELECTOR=DISCOVERY_ERROR

CONDITIONAL_DECLARATION=ALLOWED_ONLY_IF_REPRODUCIBLE
GENERATED_CASES=ALLOWED_WITH_STABLE_UNIQUE_SELECTORS
LARGE_GENERATED_SUITE_OPTIMIZATION=DEFERRED

STD_TEST_SUITE_PUBLIC_API=NOT_SELECTED_BY_D153
DECLARATION_ROOT_PUBLIC_SPELLING=NOT_SELECTED_BY_D153
BODY_INVOCATION_ARITY_OR_CONTEXT_API=NOT_SELECTED_BY_D153
FULL_CASEID_STRING_ENCODING=NOT_SELECTED_BY_D153
```

## Invariant/delta consistency

D153-B′ was checked against the explicit D152 invariants and the current Protos
baseline before durable publication.

```text
D152_LOGICAL_CASE=PASS
D152_SOURCE_CAN_HAVE_N_CASES=PASS
D152_FILE_IS_NOT_CASE_ID=PASS
D152_DISCOVERY_BEFORE_SCHEDULING=PASS
D152_FLAT_INERT_CASEPLAN=PASS
D152_FRESH_PROCESS_PER_CASE=PASS
D152_TEST_NEUTRAL_EXECUTOR=PASS
D152_EXPLICIT_CASE_ENVIRONMENT_BOUNDARY=PASS
D152_MIGRATION_INVARIANTS=UNCHANGED
D151_TOOL008_FILE_SELECTOR=PASS

NEW_UNSURFACED_ARCHITECTURAL_CONSEQUENCE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Closure contract

```text
D153_STATUS=RATIFIED
D153_SELECTED_CANDIDATE=
    B_PRIME_EXPLICIT_MODULE_DECLARATION_REMATERIALIZATION

PROTOS_REVISION=64b21418f19501ff472d03286367b1535fabb3a4
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
EXECUTABLE_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D153 ratification is governance/documentation-only. LIB018-0 may resume against
this architecture; executable implementation and the exact Case execution
environment remain separate work.
