# AUD012 — Direct Java guest-execution current-state audit

Status: **IN_PROGRESS — INITIAL CURRENT-STATE AUDIT**

Nature: durable non-normative audit evidence and remediation-routing input

Owner: `AUD012` / `guillermomolina/protos#541`

Source repository: `guillermomolina/protos`

Audited source revision: `6ccd8b91ca5446958841db125990e4e0756c3dd0`

Audit date: 2026-09-17

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Project-owner approval selected by this record: **NO**

## Purpose

Reconstruct the real current state of AUD012 from the current `main` revision,
the live AUD012 Issue, current Java production and test code, existing tests,
ratified architecture/decisions, and durable project evidence before making any
repository change.

AUD012 audits Java guest-entry paths. It is not a blanket prohibition on
`ProtosSourceCompiler`. The audit must distinguish canonical hosted execution
from compiler/backend internals, deliberately bounded unhosted/bootstrap paths,
compile-only uses, and legacy ordinary Java semantic harnesses that require
remediation or routing.

This record captures the initial state audit only. It does not settle a new
semantic or architectural choice and it does not authorize removal of any
mechanism whose retention/removal would cross the project approval gate.

## AUD012 objective

AUD012 owns the exhaustive inventory and classification of Java uses involving
at least:

- `new ProtosSourceCompiler(...)`;
- direct `compile(...).call(...)` or equivalent guest execution;
- `compileBytecode(...)` when execution ownership is relevant;
- direct `CallTarget.call(...)` on Protos guest code;
- branches that deliberately bypass hosted `Process` /
  `PolyglotExecutionContext` execution;
- Java helpers that build `ProtosActivation` state and execute guest source.

The required primary classification vocabulary is:

```text
CANONICAL_EXECUTION_REQUIRED
DIRECT_COMPILER_INTENTIONAL
COMPILE_ONLY_INTENTIONAL
PRODUCTIVE_RUNTIME_INTERNAL
LEGACY_FALLBACK_TO_REMOVE
NEEDS_SEPARATE_DESIGN_REVIEW
```

For test code, AUD012 additionally records whether Java ownership is justified or
whether the test family belongs to TEST002's later Java-to-Protos migration
audit. AUD012 does not perform that migration.

## Explicit boundaries

AUD012 does not authorize:

- a blanket ban on `ProtosSourceCompiler`;
- a Protos-visible semantic change merely to simplify the audit;
- replacement of compiler/backend tests with canonical guest execution when that
  would stop testing the intended component;
- assuming every `src/main` compiler use is production-reachable;
- reopening TEST002 migration work inside AUD012;
- rebuilding the retired global semantic-test ownership registry;
- architectural redesign without the ordinary `Dxxx` / `PLATxxx` approval gate.

## Architectural authority

### PLAT001 and I026-A4B3

PLAT001 ratifies the current runtime hosting topology: one multithread Truffle
`Context` per hosted Protos Process, with Engine sharing permitted across Process
Contexts. Context/Engine objects are implementation placement, not semantic
Process/Actor identity.

I026-A4B3 owns the production-driver cutover and retirement of direct
compiler/call entry as a separate primary **production** architecture.

Current architecture guards already enforce that production Process creators
host their Process before guest entry and that hosted module / initial-module
routes use the owning execution host and public parse.

This means AUD012 is not starting from a state where ordinary production entry
is known to rely on direct compiler execution. Its remaining work is primarily
classification of runtime internals, bounded unhosted/bootstrap exceptions, and
Java test harnesses.

### D135

D135 ratifies the Process-snapshot Test Tool execution model. Fresh execution
attempts own fresh bootstrap state, and the concrete Java mechanism remains
behind the execution binding.

`ProtosProcessSnapshotExecution` is therefore assessed against D135 rather than
against the ordinary production Process-entry rule.

### TEST002

TEST002 owns the future audit/migration of legacy Java/JUnit tests whose primary
contract is ordinary Protos-observable behavior.

TEST002 explicitly rejects rebuilding the retired semantic-ownership
registry/guard model. AUD012 may classify and route test families to TEST002 but
must not perform that migration or recreate a global ownership registry.

### Repository test-placement guidance

Current repository guidance prefers observable behavior to be tested at the
highest useful level. Ordinary Protos/TOOL002 coverage is preferred where it can
faithfully express the contract; Java/JUnit remains appropriate for Java,
Truffle, compiler/backend, runtime, scheduler, native/host integration, and
independent bootstrap evidence.

## Current `src/main/java` inventory

The current exact direct `new ProtosSourceCompiler` inventory in `src/main/java`
contains these five surfaces:

| Surface | Current role | AUD012 initial classification | Notes |
|---|---|---|---|
| `ProtosLanguage` | registered language parse/compiler backend | `PRODUCTIVE_RUNTIME_INTERNAL` | `parse(...)` delegates to `compileBytecode(...)`; compiler is the implementation component |
| `ProtosSourceFileLoader` | source-file loader returning a `CallTarget` | `COMPILE_ONLY_INTENTIONAL` | compilation/load infrastructure; execution ownership belongs to caller |
| `ProtosModuleRuntime` | module runtime | `PRODUCTIVE_RUNTIME_INTERNAL` plus bounded direct fallback | hosted Process route uses execution host/public parse; direct compiler path exists only when no hosted Process owns the activation |
| `ProtosCanonicalInitialModuleExecution` | canonical initial-module execution | bounded direct fallback | hosted route uses execution host/public parse; direct branch is documented and tested as unhosted staging only |
| `ProtosProcessSnapshotExecution` | D135 Process-snapshot execution binding | `DIRECT_COMPILER_INTENTIONAL` | fresh snapshot execution owned by D135/Test Tool binding, not ordinary production Process entry |

### Module-runtime fallback assessment

`ProtosModuleRuntime` and `ProtosCanonicalInitialModuleExecution` both make the
hosted/unhosted distinction explicitly.

When a semantic Process has an execution host, execution enters that host and
uses `ProtosLanguageContext.current().parsePublic(...)` on the materialized
module source.

The direct compiler branch is retained only for unhosted/staging Java harnesses.
Current A4B3 architecture tests explicitly protect this distinction.

Initial result:

```text
MODULE_RUNTIME_DIRECT_FALLBACK=INTENTIONAL_UNHOSTED_STAGING
INITIAL_MODULE_DIRECT_FALLBACK=INTENTIONAL_UNHOSTED_STAGING
LEGACY_FALLBACK_TO_REMOVE=NOT_ESTABLISHED
```

### Process-snapshot assessment

`ProtosProcessSnapshotExecution.execute(...)` still performs direct
`new ProtosSourceCompiler().compile(source).call(activation)`.

Current evidence ties this path directly to the D135 Process-snapshot Test Tool
lane. Its focused test proves fresh equivalent bootstrap on repeated execution
and the snapshot-identity conformance case.

Initial result:

```text
PROCESS_SNAPSHOT_EXECUTION=DIRECT_COMPILER_INTENTIONAL
AUTHORITY=D135
JAVA_OWNERSHIP=HOST_RUNTIME_BOOTSTRAP
```

No current evidence justifies mechanically replacing this path with the ordinary
production Process-hosted entry boundary.

## Direct `CallTarget.call(...)` in runtime internals

A textual `.call(...)` occurrence is not by itself a separate guest-entry
architecture.

Current runtime examples include:

- closure execution plans;
- semantic Bytecode helper roots;
- Bytecode task execution;
- Bytecode I/O operation/release execution;
- context-local Bytecode closure plans;
- `ProtosPolyglotExecutionContext`.

These paths execute already-prepared guest targets as part of the runtime.
`ProtosPolyglotExecutionContext` is specifically canonical: it enters the
Polyglot Context, performs `parsePublic(...)`, and invokes the resulting target.

Initial rule for the remaining inventory:

```text
CALLTARGET_CALL != AUTOMATIC_DIRECT_GUEST_ENTRY_DEFECT
```

Each occurrence must be classified by who prepared the target, who owns the
execution host, and whether it creates a second top-level execution architecture.

## Current `src/test/java` assessment

The current Java test tree still contains many direct
`ProtosSourceCompiler().compile(...).call(...)` or equivalent paths.

The initial audit separates them into evidence classes rather than applying a
mechanical replacement rule.

### Clearly Java-owned / intentional families

The following are currently strongly justified as Java-owned:

| Test/family | Initial classification rationale |
|---|---|
| `ProtosSourceCompilerTest` | compiler is directly the component under test |
| `ProtosMatchBytecodeExecutionTest` | Bytecode/backend execution mechanics |
| `ProtosPerf006B6A6A1TaskOwnedClosureDispatchTest` | task-owned Bytecode/closure runtime mechanics |
| `ProtosA4B3ProductionEntryArchitectureTest` | source/architecture guard; compiler references are architecture evidence |
| `ProtosA4B3ModuleProcessHostingTest` | proves hosted versus deliberately unhosted module-entry boundary |
| `ProtosProcessSnapshotExecutionTest` | D135 host/bootstrap execution-binding evidence |
| `ProtosParallelPolyglotContextRoutingTest` | Truffle Context, carrier placement and physical parallelism evidence |
| bootstrap/Core source-loader tests | Java/bootstrap infrastructure evidence, subject to per-test confirmation |

These tests still belong in the complete AUD012 occurrence inventory. Their
presence does not imply remediation.

### TOML reference pair

The current TOML tests demonstrate why AUD012 must classify intent rather than
ban syntax mechanically.

`ProtosTomlParserStressTest` now executes retained Java semantic guest code
through `ProtosTestExecutionSupport.evaluate(...)`, which supplies an entered
Truffle Context/public-parse boundary.

`ProtosTomlParserModuleTest` still uses direct
`ProtosSourceCompiler().compile(...).call(...)`.

The AUD012 Issue already records comparative evidence showing that the canonical
bridge materially improved the stress harness while making the module harness
slower in the tested configuration. The direct module-harness path therefore
cannot be declared defective merely from its syntax.

Initial result:

```text
TOML_STRESS_TEST=CANONICAL_TEST_BRIDGE
TOML_MODULE_TEST=DIRECT_PATH_REQUIRES_INTENT_CLASSIFICATION
BLANKET_DIRECT_COMPILER_BAN=REJECTED_BY_CURRENT_EVIDENCE
```

### Likely semantic-harness candidates requiring detailed classification

Current candidates include Java tests/families for ordinary guest-visible
module/protocol behavior such as:

- URI;
- CSV;
- math/integer;
- command-line module/spec/result model;
- standard Map/Path/Array behavior;
- networking IP addresses/endpoints;
- JSON;
- TOML data model/parser/encoder;
- Collections Set/Array families;
- match/guard/Array/Map conformance.

Their current direct execution syntax is **not enough** to select remediation.

For each coherent family the next audit pass must establish:

1. what contract the Java test actually owns;
2. whether direct execution deliberately isolates that contract;
3. whether the canonical Java test bridge is required or useful;
4. whether the family predates current test-placement guidance;
5. whether the primary contract is ordinary Protos-visible semantics and should
   therefore be routed to TEST002;
6. what independent Java host/runtime/bootstrap evidence must remain.

Until that analysis is complete, the safe state is:

```text
AUD012_PATH=NEEDS_CLASSIFICATION
TEST002_RELATION=INVESTIGATE
REMOVAL_AUTHORIZED=NO
```

### Filesystem conformance families

Several filesystem conformance tests execute `.protos` cases through
`ProtosSourceFileLoader().load(...).call(...)`.

These tests also exercise substantial Java/native filesystem backends and
capability confinement, so they have a stronger initial reason to remain
Java-owned than a purely guest-semantic module test.

They still require per-family classification before the inventory can close.

## Existing prevention coverage

Current A4B3 architecture tests already protect the production boundary:

- production Process creators must host their Process before guest entry;
- production Process creators must not retain direct compiler entry;
- hosted module and canonical initial-module routes must remain hosted/public
  parse.

AUD012's broader prevention requirement is **not yet satisfied**.

There is currently no general fail-closed rule that prevents a newly added
ordinary Java semantic test from silently introducing a raw
`compile(...).call(...)` guest-entry harness.

Therefore:

```text
PRODUCTION_ENTRY_REGRESSION_GUARD=EXISTS
ORDINARY_JAVA_SEMANTIC_DIRECT_ENTRY_GUARD=NOT_YET_ESTABLISHED
AUD012_PREVENTION_REQUIREMENT=OPEN
```

## Approval gate

AUD012 requires a mechanically testable fail-closed prevention rule while
allowing justified compiler/backend/compile-only exceptions.

TEST002 simultaneously prohibits recreating a global semantic-test ownership
registry.

The desired outcome is therefore already constrained, but the mechanism for
representing justified exceptions has not been selected.

At least these implementation-policy shapes remain possible:

1. a central file/class allowlist;
2. a local marker/annotation declaring a justified exception;
3. an explicit deliberately-unhosted test execution API/helper, while retaining
   raw compiler entry for tests where the compiler/backend itself is under test.

Selecting one of these as durable repository policy affects future test
structure and exception ownership. No option is selected by this audit.

Per the current `AGENTS.md` approval gate:

```text
AUD012_PREVENTION_POLICY=NEEDS_PROJECT_OWNER_APPROVAL
IMPLEMENTATION_BEFORE_APPROVAL=NO
```

The next inventory/classification pass should make the exception population
concrete before presenting a final prevention-design packet.

## Current closure assessment

At audited revision `6ccd8b91ca5446958841db125990e4e0756c3dd0`:

```text
JAVA_DIRECT_GUEST_ENTRY_INVENTORY=IN_PROGRESS
TEST_JAVA_OCCURRENCES=NOT_YET_EXHAUSTIVELY_CLASSIFIED
MAIN_JAVA_OCCURRENCES=INITIAL_CLASSIFICATION_COMPLETE_NOT_FINAL_RESCAN
UNJUSTIFIED_LEGACY_PATHS=NOT_YET_DETERMINED
INTENTIONAL_DIRECT_COMPILER_USES=PARTIALLY_JUSTIFIED
NEW_SEMANTIC_TEST_REGRESSION_GUARD=NOT_YET_ENFORCED
AGENTS_OR_TEST_PLACEMENT_GUIDANCE=GUIDANCE_EXISTS
FINAL_RESCAN=NOT_RUN
AUD012_CLOSABLE=NO
```

## Next smallest audit step

The next bounded step is inventory-only:

1. enumerate the complete current `src/test/java` guest-entry occurrence set;
2. remove textual false positives that do not execute guest code;
3. group occurrences by coherent test family;
4. assign each family one primary AUD012 classification;
5. assign each test family `JAVA_OWNED`, `TEST002_CANDIDATE`, or
   `NEEDS_INVESTIGATION`;
6. identify the minimal real exception set that a future fail-closed guard must
   preserve.

No code, test, configuration, or semantic change is required for that step.

## Durable-record status

This document is an initial AUD012 evidence record, not a closure record.

It records repository-dependent evidence against one exact Protos source
revision and is expected to be extended or superseded by later AUD012
inventory/final-rescan evidence as the work progresses.

It does not move live scheduling authority out of GitHub Issues and does not
constitute project-owner approval of any unresolved prevention architecture.
