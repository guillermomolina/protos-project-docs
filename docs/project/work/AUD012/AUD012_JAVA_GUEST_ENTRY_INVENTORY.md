# AUD012 — Java guest-entry inventory

Status: **IN_PROGRESS — CURRENT INVENTORY / CLASSIFICATION**

Nature: durable non-normative audit evidence

Owner: `AUD012` / `guillermomolina/protos#541`

Source repository: `guillermomolina/protos`

Audited source revision: `6ccd8b91ca5446958841db125990e4e0756c3dd0`

Audit date: 2026-09-17

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Project-owner approval selected by this record: **NO**

## Purpose

Record the current Java guest-entry inventory required by AUD012 without changing
`guillermomolina/protos` and without silently turning pattern matches into design
conclusions.

The inventory separates four different things that can look similar in a text
search:

1. ordinary source guest execution from Java;
2. deliberate unhosted/bootstrap Java execution;
3. compiler/lowering/runtime component execution whose direct target is itself
   the subject of the test or implementation;
4. compile-only or textual architecture evidence that does not execute guest
   code at all.

The distinction is required by the AUD012 owner correction: direct compiler use
is not globally invalid. Hosted/production-representative execution uses the
canonical host/public-parse boundary; deliberately unhosted Java harnesses may
retain direct compiler entry when the exception is explicit and justified;
compiler/backend/compile-only tests may use the compiler directly.

`TEST002` separately owns whether retained Java tests whose primary contract is
ordinary Protos-observable behavior should later migrate to ordinary Protos /
TOOL002. An AUD012 path classification therefore does not settle permanent test
ownership.

## Counting baseline

At the audited source revision, exact GitHub code search for:

```text
"new ProtosSourceCompiler()" path:src/test/java
```

returns:

```text
RAW_TEST_COMPILER_CONSTRUCTOR_FILES=44
SEARCH_INCOMPLETE_RESULTS=NO
```

That count is a discovery count, not an execution-defect count. It contains one
textual architecture-guard match and compile-only uses, and it does not contain
all equivalent guest-entry mechanisms.

A separate exact search for:

```text
"ProtosExecution.createCallTarget" path:src/test/java
```

returns:

```text
DIRECT_LOWERED_TARGET_TEST_FILES=19
SEARCH_INCOMPLETE_RESULTS=NO
```

Those nineteen are predominantly compiler/lowering/runtime component tests, not
ordinary source-entry harnesses.

`ProtosSourceFileLoader` provides another equivalent path: it compiles a source
file and returns a `CallTarget`; callers may then invoke that target directly.
That family is inventoried separately below.

A final broad post-remediation rescan is still required before AUD012 closure,
so this record deliberately says **current inventory**, not final zero-delta
proof.

## Classification vocabulary used here

AUD012 primary path classifications remain the Issue-defined values:

```text
CANONICAL_EXECUTION_REQUIRED
DIRECT_COMPILER_INTENTIONAL
COMPILE_ONLY_INTENTIONAL
PRODUCTIVE_RUNTIME_INTERNAL
LEGACY_FALLBACK_TO_REMOVE
NEEDS_SEPARATE_DESIGN_REVIEW
```

Two additional columns in this record do not create new AUD012 outcome values:

- `JAVA_OWNED` — current evidence requires or strongly justifies Java-level
  observation/runtime/bootstrap/component control;
- `TEST002_REVIEW_REQUIRED` — TEST002 must decide eventual Java-vs-Protos
  ownership, including possible `SPLIT`; this is not itself an AUD012 path
  defect.

`DIRECT_COMPILER_INTENTIONAL` means only that the **current execution route** is
an intentional unhosted/component-level route while that Java harness exists.
It does not mean the test must remain Java forever.

## `src/main/java` inventory

| Surface | Mechanism | AUD012 classification | Current evidence |
|---|---|---|---|
| `ProtosLanguage` | `compileBytecode(...)` from Truffle `parse(...)` | `PRODUCTIVE_RUNTIME_INTERNAL` | Registered-language compiler/backend implementation. |
| `ProtosSourceFileLoader` | `compiler.compile(source)` returning `CallTarget` | `COMPILE_ONLY_INTENTIONAL` | Loader compiles only; caller owns execution. |
| `ProtosCoreBootstrap` | repeated `sourceLoader.load(coreFile).call(bootstrapActivation)` | `PRODUCTIVE_RUNTIME_INTERNAL` | Independent Core bootstrap constructs the language prelude before ordinary hosted Process execution exists. |
| `ProtosModuleRuntime` hosted route | Process execution host + public parse | `PRODUCTIVE_RUNTIME_INTERNAL` | Canonical production route. |
| `ProtosModuleRuntime` unhosted route | `compiler.compile(source).call(activation)` | `DIRECT_COMPILER_INTENTIONAL` | Source comment and A4B3 guard explicitly retain it only for deliberately unhosted/non-Process Java semantic harnesses. |
| `ProtosModuleRuntime` Bytecode import preparation | `compileBytecode(...)` | `PRODUCTIVE_RUNTIME_INTERNAL` | Backend-private module-child-root preparation inside current language context. |
| `ProtosCanonicalInitialModuleExecution` hosted route | Process host + public parse + RootTask | `PRODUCTIVE_RUNTIME_INTERNAL` | Canonical production route. |
| `ProtosCanonicalInitialModuleExecution` unhosted route | direct compiler target + RootTask | `DIRECT_COMPILER_INTENTIONAL` | Source comment and A4B3 guard say deliberately unhosted Java harness only; not production driver. |
| `ProtosProcessSnapshotExecution` | direct compiler/call | `DIRECT_COMPILER_INTENTIONAL` | D135 deterministic Process-snapshot Test Tool execution binding; fresh Process/arguments/environment fixture per invocation. |
| `ProtosPolyglotExecutionContext.execute*` | entered Context + `parsePublic(...)` + RootTask | `CANONICAL_EXECUTION_REQUIRED` **satisfied** | Canonical hosted execution boundary. |
| `ProtosPolyglotExecutionContext.evaluatePersistent` | entered Context + public parse + target call | `CANONICAL_EXECUTION_REQUIRED` **satisfied** | Persistent REPL activation remains inside owning entered Process Context. |
| closure/Bytecode task/I/O/semantic-root execution plans | direct invocation of already-prepared targets | `PRODUCTIVE_RUNTIME_INTERNAL` | Runtime-internal continuation/task/component execution, not a second top-level guest-entry architecture. |

Current main-source conclusion:

```text
PRODUCTION_PRIMARY_DIRECT_COMPILER_ARCHITECTURE=NO
HOSTED_PROCESS_ENTRY=CANONICAL_PUBLIC_PARSE
INTENTIONAL_UNHOSTED_MAIN_FALLBACKS=YES
MAIN_LEGACY_FALLBACK_TO_REMOVE=NOT_ESTABLISHED
```

## Exact raw test compiler-constructor inventory

### Compile-only or textual/non-execution matches

| Test | AUD012 classification | Ownership | Reason |
|---|---|---|---|
| `ProtosModuleSourceIdentityTest` | `COMPILE_ONLY_INTENTIONAL` | `JAVA_OWNED` | Compiles a neutral `ProtosModuleSource` only to inspect root/source identity; does not call the target. |
| `ProtosTestToolManifestPlanTest` | `COMPILE_ONLY_INTENTIONAL` | `JAVA_OWNED` | Compile-before-policy/tooling evidence; constructor hit is not ordinary guest execution. |
| `ProtosA4B3ProductionEntryArchitectureTest` | not an executing occurrence | `JAVA_OWNED` | Source-inspection architecture guard contains compiler spelling as text. |

### Compiler/backend/runtime/bootstrap/architecture tests

The following current direct-entry uses are justified by Java-level component,
representation, runtime, bootstrap, scheduler, host-integration, or architecture
evidence. They are not TEST002 migration candidates merely because they execute
Protos source from Java.

| Test | AUD012 classification | Ownership / rationale |
|---|---|---|
| `ProtosModuleRuntimeTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — resolver identity, Actor-local module cache, cycles, retry and host-failure mapping. |
| `ProtosCoreBootstrapTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — Core bootstrap/representation evidence. |
| `ProtosSourceCompilerTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — compiler is the component under test. |
| `ProtosMatchExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — retained host/runtime matching-error coverage after ordinary semantics moved to Protos tests. |
| `ProtosMapMatchExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — represented/runtime match coverage. |
| `ProtosGuardMatchExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — host/runtime guard-error evidence. |
| `ProtosArrayMatchExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — represented/runtime match coverage. |
| `ProtosParallelExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — scheduler/concurrency/runtime evidence; individual helper routing remains audit-visible. |
| `ProtosStandardMapProtocolTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — explicitly Java-side represented close/freeze lifecycle; ordinary open-state semantics live in Protos tests. |
| `ProtosJsonParserModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — explicitly Java-side implementation stress for deep nesting and large materialization. |
| `ProtosJsonDataModelModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — explicitly tests Actor graph-transfer/runtime boundary and mutation-state preservation. |
| `ProtosStandardArrayFactoryTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — represented-value/materialization/lifecycle evidence. |
| `ProtosStandardPathProtocolTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — represented Path/runtime component evidence. |
| `ProtosPolymorphicInvocationTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — invocation/lowering/host-identity runtime evidence. |
| `ProtosExactExecutionFacilityTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — execution-facility/scheduler evidence. |
| `ProtosIdentityMapConformanceTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — exact bootstrap/represented runtime-family evidence. |
| `ProtosA4B3ModuleProcessHostingTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — architecture evidence explicitly distinguishes hosted public parse from unhosted staging. |
| `CanonicalMapConstructionExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — canonical/backend construction execution. |
| `ProtosStandardLibraryModuleResolverTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — resolver/host boundary. |
| `ProtosParallelPolyglotContextRoutingTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — owns real Process Context placement, exact `ProtosLanguageContext` routing and physical multi-carrier evidence. |
| `ProtosPerf006B6A6A1TaskOwnedClosureDispatchTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — Bytecode/task-owned closure runtime evidence. |
| `ProtosMatchBytecodeExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — compiler/Bytecode backend evidence. |
| `ProtosArrayConformanceCompletionTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — source Javadoc explicitly retains Java only for represented lifecycle and host-visible callback control transfer. |
| `ProtosNetworkingIpAddressesModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — source Javadoc says Protos fixtures own observable IP semantics; Java owns exact local export-surface observation unavailable to guest code. |
| `ProtosNetworkingIpEndpointsModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — same exact export-surface/bootstrap rationale as IP addresses. |
| `ProtosCollectionsSetMutationModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — constructs/closes/freezes represented Map/IdentityMap state directly and observes guest behavior across that host-established lifecycle. |
| `ProtosCollectionsArrayAlgorithmsModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — injects native callback and wrong-source host object to prove failure occurs before callback. |
| `ProtosCollectionsArrayReduceSortModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — same host-injected callback/fail-before-callback boundary for reduce/sort. |

### Current direct route, but TEST002 ownership review required

These tests may legitimately use a deliberately unhosted direct route **while
retained as Java harnesses**, but their long-term Java ownership contains enough
ordinary Protos-visible semantics that TEST002 must perform its fresh
`MIGRATE_TO_PROTOS | KEEP_JAVA_HOST_RUNTIME | KEEP_JAVA_BOOTSTRAP | SPLIT`
classification.

| Test | AUD012 path classification | TEST002 relation |
|---|---|---|
| `ProtosTomlParserModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — ordinary TOML semantic coverage plus adversarial/stress concerns; AUD012 A/B evidence specifically rejects mechanically replacing this direct path. |
| `ProtosTomlEncoderModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — ordinary encoder semantics plus deep implementation-stress cases. |
| `ProtosTomlDataModelModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — ordinary constructor/error semantics mixed with Actor-local/module-transfer runtime evidence. |
| `ProtosInvalidSuperExecutionTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — source-visible ordering/error semantics mixed with exact prototype identity and direct runtime invocation evidence. |
| `ProtosCsvModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — public/export semantics mixed with Actor-local module cache and fresh parser-state identity. |
| `ProtosCollectionsSetModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — represented Map-family parent and Actor-local import caching are Java/runtime relevant, while part of the surface is ordinary library behavior. |
| `ProtosUriModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — current class identifies itself as real-std conformance and primarily observes the library surface. |
| `ProtosCommandLineModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — source identifies it as a Protos-language semantic harness. |
| `ProtosMathIntegerModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — closed-surface real-std semantic conformance. |
| `ProtosCommandLineResultModelTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — substantial guest-visible result-model semantics with Java representation inspection. |
| `ProtosCommandLineSpecModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — command-line specification/module semantics require ownership review. |
| `ProtosTomlClosureConformanceTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — guest semantic conformance requires ownership review. |
| `ProtosCollectionsSetAlgebraModuleTest` | `DIRECT_COMPILER_INTENTIONAL` | `TEST002_REVIEW_REQUIRED` — ordinary collection algebra is a TEST002 ownership candidate unless a Java-only boundary is demonstrated. |

This accounts for all 44 exact raw constructor-search files: 3
compile-only/textual matches, 28 currently Java-owned direct/component uses, and
13 tests whose current direct route is intentional but whose permanent ownership
must be reviewed by TEST002.

The 28/13 split is an AUD012 audit classification, not a TEST002 resolution.
TEST002 may later split a class or discover stronger Java-only evidence.

## `ProtosSourceFileLoader` equivalent paths

`ProtosSourceFileLoader` compiles a file and returns a target. Direct invocation
by its callers is therefore in AUD012 scope even when no
`new ProtosSourceCompiler()` appears in the caller.

| Surface | AUD012 classification | Ownership / rationale |
|---|---|---|
| `ProtosCoreBootstrap` | `PRODUCTIVE_RUNTIME_INTERNAL` | Independent bootstrap of Core source into bootstrap activation. |
| `ProtosSourceFileLoaderTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — loader/component contract. |
| `ProtosCoreContextSourceTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — Core source/bootstrap representation. |
| `ProtosFilesystemLanguageConformanceTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — native filesystem/capability integration with real host effects. |
| `ProtosFilesystemMaturityConformanceTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — native filesystem/capability integration. |
| `ProtosFilesystemTreeSurfaceConformanceTest` | `DIRECT_COMPILER_INTENTIONAL` | `JAVA_OWNED` — host filesystem/tree integration. |

`ProtosCoreBootstrapTest` also reaches this path but is already counted in the
raw compiler-constructor inventory through other code in the same class; it is
not double-counted as an additional file.

## Direct lowered/constructed `CallTarget` component tests

Exact `ProtosExecution.createCallTarget` search finds nineteen current test
classes:

```text
ProtosExecutionTest
ProtosCallableLoweringTest
ProtosParameterBindingNodeTest
CanonicalClosureMaterializationTest
CanonicalEmptySequenceExecutionTest
ProtosArgsNodeTest
CanonicalIntrinsicExecutionTest
ProtosNonLocalReturnTest
ProtosClosureInvokerTest
CanonicalLookupExecutionTest
CanonicalObjectExecutionTest
CanonicalMemberReadExecutionTest
ExtractedClosureBindingExecutionTest
ProtosArgumentVectorNodeTest
CanonicalToTruffleLowererTest
CanonicalCompositionExecutionTest
CanonicalBareSlotMutationExecutionTest
CanonicalExplicitMemberMutationExecutionTest
ProtosPerf006B2AClosureActivationTest
```

These tests construct/lower canonical/AST/node structures in Java and execute
the resulting targets to observe backend/runtime behavior. Their classification
is:

```text
AUD012_CLASSIFICATION=DIRECT_COMPILER_INTENTIONAL
MECHANISM=INTENTIONAL_DIRECT_TARGET_COMPONENT_EXECUTION
TEST_OWNERSHIP=JAVA_OWNED
```

The AUD012 enum name contains `COMPILER`, but the relevant property here is the
intentional direct execution boundary; these files need not literally construct
`ProtosSourceCompiler`.

Additional Bytecode/backend suites such as the PERF006 Bytecode conformance
helpers and `ProtosMatchBytecodeExecutionTest` are likewise Java-owned component
execution. They remain in the final broad rescan even where their exact spelling
is not `ProtosExecution.createCallTarget`.

## Canonical retained-Java bridge

`ProtosTestExecutionSupport` is the current canonical bridge for retained Java
semantic harnesses that **must** cross a post-A4B3/B6B hosted execution boundary.
It:

- enters a shared test-only `ProtosPolyglotExecutionContext`;
- parses through `ProtosLanguageContext.current().parsePublic(...)`;
- executes via `ProtosRootTaskExecution` or invokes the target while the context
  remains entered;
- preserves the supplied Activation/Actor/execution-domain semantic state;
- explicitly does not create a second production architecture.

`ProtosTomlParserStressTest` is a current example already using this bridge after
PERF009-A.

This bridge is **not** evidence that every deliberately unhosted Java harness
must be converted to it. AUD012's TOML A/B result is direct evidence against that
mechanical rule.

## Current findings

### 1. Production architecture is already guarded

A4B3 architecture tests currently require production Process creators to host
the Process before guest execution, and explicitly protect the hosted/public
parse versus unhosted-staging distinction in module execution.

No current evidence in this audit establishes a production
`LEGACY_FALLBACK_TO_REMOVE`.

### 2. Direct Java entry has several legitimate meanings

The inventory demonstrates legitimate current direct execution for:

- compiler and lowering component tests;
- Core/bootstrap tests;
- represented lifecycle/state setup unavailable to ordinary guest tests;
- scheduler/concurrency/Truffle Context placement;
- native filesystem/host integration;
- Actor graph-transfer and identity boundaries;
- deliberate implementation stress;
- explicitly unhosted Java semantic harnesses.

Therefore a raw pattern ban would be incorrect.

### 3. TEST002 still has real work

Several current Java suites are mostly or partly ordinary Protos-visible
semantics. AUD012 can route them to TEST002, but must not decide the TEST002
migration outcome while solving guest-entry architecture.

### 4. The prevention requirement is still open

No current general fail-closed rule prevents a newly added **unclassified**
ordinary Java semantic test from silently introducing direct guest entry.

The guard must simultaneously preserve the legitimate categories above and must
not recreate TEST002's rejected global semantic-test ownership registry.

## Approval gate reached: prevention mechanism

The desired invariant is already owner-constrained:

```text
FAIL_CLOSED_ON_NEW_UNCLASSIFIED_ORDINARY_JAVA_DIRECT_GUEST_ENTRY=YES
GLOBAL_DIRECT_COMPILER_BAN=NO
JUSTIFIED_COMPILER_BACKEND_COMPILE_ONLY_DIRECT_ENTRY=ALLOWED
DELIBERATELY_UNHOSTED_JAVA_HARNESS=ALLOWED_WHEN_EXPLICITLY_JUSTIFIED
GLOBAL_SEMANTIC_TEST_OWNERSHIP_REGISTRY=NO
```

How justified exceptions are represented and mechanically enforced is a durable
test-infrastructure/architecture choice. Current `AGENTS.md` therefore requires
explicit project-owner approval before implementation.

The meaningful current alternatives are:

### A — central class/file exception allowlist

A CI architecture test keeps a list of Java files allowed to use raw direct
entry and fails on any new file.

Advantages: simple, mechanical, strongly fail-closed.

Costs: the list becomes a central ownership/exception registry and is therefore
very close to the registry model TEST002 explicitly rejected; renames and splits
create central bookkeeping.

### B — local marker / annotation

A local marker or Java annotation declares that a class intentionally owns a
direct entry exception; the architecture guard rejects unmarked direct entry.

Advantages: intent stays with the test instead of in one central list.

Costs: a comment marker is mechanically brittle; an annotation creates a new
permanent test-infrastructure institution and can degenerate into boilerplate
that merely suppresses the guard.

### C — intent-bearing unhosted execution API plus structural raw-component guard

Provide one explicit test-side API for **deliberately unhosted semantic Java
execution**. Raw compiler/CallTarget entry remains available where the
compiler/backend/bootstrap/runtime component itself is under test. An
architecture guard rejects new ordinary semantic raw entry unless it goes
through the explicit unhosted API, while the existing canonical
`ProtosTestExecutionSupport` remains the path for tests that require an entered
Context/public parse.

Advantages:

- exception intent is executable and local at the call site;
- it does not create a central semantic ownership registry;
- canonical hosted and deliberately unhosted test execution become explicit,
  distinct operations;
- compiler/backend/component tests retain direct access to what they test;
- future code review sees the reason-bearing boundary rather than an opaque
  allowlist membership.

Costs:

- establishes a durable test-infrastructure distinction;
- the architecture guard still needs a bounded, mechanically reliable way to
  distinguish raw component-test areas from ordinary semantic harnesses;
- exact API/name and guard scope must be kept small to avoid replacing one
  registry with another institution.

**Audit recommendation, pending project-owner approval: C.**

No prevention implementation is authorized by this record.

## Current closure state

At source revision `6ccd8b91ca5446958841db125990e4e0756c3dd0`:

```text
JAVA_DIRECT_GUEST_ENTRY_INVENTORY=SUBSTANTIALLY_COMPLETE_PENDING_FINAL_RESCAN
RAW_TEST_COMPILER_CONSTRUCTOR_FILES=44
DIRECT_LOWERED_TARGET_TEST_FILES=19
MAIN_JAVA_OCCURRENCES=CLASSIFIED
TEST_JAVA_OCCURRENCES=CLASSIFIED_FOR_AUD012_PATH
TEST002_OWNERSHIP_REVIEW=OPEN
UNJUSTIFIED_LEGACY_PATHS=NONE_PROVEN_YET
INTENTIONAL_DIRECT_COMPILER_USES=JUSTIFICATION_RECORDED_BY_CATEGORY
NEW_SEMANTIC_TEST_REGRESSION_GUARD=NOT_YET_ENFORCED
PREVENTION_MECHANISM=NEEDS_PROJECT_OWNER_APPROVAL
FINAL_RESCAN=NOT_RUN
AUD012_CLOSABLE=NO
```

The next implementation slice must not begin until the project owner explicitly
selects or delegates the prevention mechanism.