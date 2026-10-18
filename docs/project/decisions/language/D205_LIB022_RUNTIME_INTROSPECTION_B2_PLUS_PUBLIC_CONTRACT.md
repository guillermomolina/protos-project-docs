# D205 — LIB022 runtime and diagnostic introspection: Candidate B / B2+ public contract

**Status:** RATIFIED — owner-selected **Candidate B, exact B2+ public contract** on **2026-10-18**.  
**Formal decision:** [D205/#922](https://github.com/guillermomolina/protos/issues/922).  
**Parent:** [LIB022/#921](https://github.com/guillermomolina/protos/issues/921).  
**Product basis:** [guillermomolina/protos@6ad7ab2864e8e48fc546dce5d4cd61472a026819](https://github.com/guillermomolina/protos/commit/6ad7ab2864e8e48fc546dce5d4cd61472a026819), `0.3.376-SNAPSHOT`, GraalVM `25.4.4.1.1`, observed during design research, **not** an instruction to check out that revision during implementation.  
**Earlier owner selection and research:** [LIB022-0 Candidate B evidence](../../evidence/LIB022/LIB022_0_CANDIDATE_B_RESEARCH_AND_OWNER_SELECTION_2026_10_18.md) at `d2f083816a9b6772dd0dfadc9b64774fc6044b2e`.  
**Role:** durable, non-normative ratification and evidence. Public Standard Library behavior must be reconciled with the applicable normative `spec/` owners when implemented. This record is **not** evidence that any code, tests or Native Image feature already shipped.

## Exact owner-approval provenance

During the active D205 interaction on 2026-10-18, after reviewing the expanded scope and its tradeoffs, the owner explicitly stated:

> aprobada Candidate B, contrato B2+ de cinco operaciones en dos módulos, con preparación y materialización permitidas exclusivamente bajo demanda, y las garantías PAYG/NORMAL/DEBUG intactas.

Approval is **exactly** of the Candidate B decomposition and the presented B2+ five-operation/two-module contract, including its authority, failure, stability and PAYG/NORMAL/DEBUG boundaries. It is not approval to add other public methods, new runtime state, implicit DEBUG, privileged compiler/heap instrumentation, or an arbitrary Java reflection API. Previously unselected B2-min is superseded as a recommendation. This ratification does not imply successful implementation.

## Selected architecture and exact public surface

Candidate B retains three independent roles:

1. Small, portable, read-only stable **runtime metadata**;
2. Separately opted-in, implementation-dependent **on-demand diagnostic inspection**;
3. Existing host/developer tooling for compilation graphs, compiler events, heap/privileged operations.

### `std:runtime` — stable metadata

One public operation: **`info()`** (exactly zero arguments). Success returns a **fresh, frozen, ordinary Protos Object**, with Object as its delegation parent and exactly these own String slots:

| Own slot | Contract |
| --- | --- |
| `implementation` | Exactly `"Protos"` |
| `implementationVersion` | Actual packaged/distribution version, or `"development"` when version identity is unavailable; reuse existing CLI version-identity precedence |
| `engineVersion` | Nonempty opaque version string from the active Polyglot Engine, not a hand-coded `pom.xml` version and not guaranteed SemVer |
| `platform` | Exactly `"JVM"` or `"NATIVE_IMAGE"` |
| `executionMode` | Exactly `"NORMAL"` or `"DEBUG"`, read from the entered Context's immutable mode |

Each call observes the **entered Context** and returns detached values. No Engine, Context, Process, Actor, Task, RootCallTarget or mutable native handle becomes a guest value. No JVM/Native Image equality of diagnostic availability is inferred. No added `isCompiled`, `isOptimized`, `jitEnabled` or public compiler-capability promise is selected. Obtaining the implementation version must not fork the existing CLI/distribution version policy.

### `std:inspect` — opt-in diagnostic inspection

**Host authority:** a new Context-level Boolean option **`protos.Inspect`**, default `false`, fixed for that Context's lifetime. This option is distinct from `protos.Debug`: it does not select DEBUG, start DAP, grant Java/foreign interop, create instrumentation listeners or widen host/Context/Actor/Process/Task authority. Importing the inspection library under a host without authorization must fail with an ordinary Protos Error and must not elevate authority.

Exactly **four** public operations, each of arity one. Successful outputs are detached, nonempty **String** snapshots. Their human-readable text, diagnostic categories, ordering and physical Truffle details are implementation-specific and **not** cross-version stable contracts:

| Operation | Scope of the snapshot |
| --- | --- |
| `representation(value)` | Describes the value's available implementation representation or storage category; never arbitrary recursive contents, foreign members, captured values, host references, IDs, addresses or Java reflection |
| `callableInfo(closure)` | Describes available source/native Closure identity, canonical definition/source-span metadata and executable-plan/projection status; does **not** invent compiled state, force invocation, expose captured values or promise a semantic call stack |
| `bytecode(closure)` | Textually dumps available **Context-owned physical Bytecode DSL** for a source-backed Closure; does not claim a 1:1 mapping of semantic Closure, inline activation, physical root or continuation root |
| `truffleTree(closure)` | Textual **physical Truffle node-tree** description for an available Context-owned root; initial adapter prefers safe compact tree output such as `NodeUtil.printCompactTreeToString`, not raw arbitrary Java-field dump, semantic Protos AST, Graal compiler graph or full source-attribution text |

The first implementation may reuse existing `ProtosClosureValue` canonical definitions and `ProtosClosureExecutionPlan`/Context-local Bytecode projections, along with Truffle's `BytecodeRootNode.dump()`, `BytecodeNode.dump()` and compact node tree facilities. The exact Java adapter, allocation layout and formatter implementation are internal choices, **not new public contracts**.

### Explicit, on-demand preparation and materialization: approved

A requested inspection **may**:

- resolve the authorized executable projection for the entered Context;
- prepare or rematerialize that executable on demand when supported by existing mechanisms and authority;
- materialize source metadata on demand; and
- allocate formatting/snapshot storage proportional to the requested diagnostic work.

No requested operation may invoke the inspected Closure merely to obtain metadata, synthesize frames/Tasks/Actors, acquire a different Context's executable, or expose live Truffle roots or mutable handles. A native Closure need not have bytecode and may be diagnosed as unsupported by `bytecode`/`truffleTree`. An unavailable or unauthorized physical representation signals Error; no fake physical root or compiled state is invented.

**PAYG is not a cheap-inspection rule:** performing an explicitly requested diagnostic may be expensive. The hard boundary is **zero new unavoidable cost for a program that does not use the introspection capabilities**. Demanding free inspections would wrongly narrow the selected API.

**Source-only vs instrumentation reparse:** pinned Truffle `BytecodeRootNodes.ensureSourceInformation()` supports reparse for source metadata and documents that source-only updates do not invalidate compiled code; `update(BytecodeConfig)` that adds tags/instrumentation can reparse a root group, invalidate optimized code and reset feedback. The initial four operations do **not** implicitly enable extra tags, enter DEBUG or subscribe to instrumentation. Their source-rematerialization consequences must be measured/verified and reported, not assumed zero.

### Exact failures and compatibility

- Wrong arity, inappropriate argument kind, import without inspection authorization, unavailable/unsupported inspection, context-ownership rejection and failed materialization signal **fresh ordinary Protos Error** through existing rules.
- `bytecode` and `truffleTree` require an admitted source-backed Closure with a legitimately obtainable Context-local executable; native Closure or absent valid physical root is **not** a successful empty String. `callableInfo` can report an admitted native Closure's existing category without pretending it has bytecode.
- Unsupported is **never** represented by `null`, an empty String, fabricated data or Java exception escaping as a guest return value. No new inspection-specific public Error hierarchy, mandatory reason-code schema or error-message text promise.
- The `std:runtime` module path, `info/0` signature, five field names/types/meanings and frozen-object semantics are stable.
- The `std:inspect` path, four names/arities, String result type, authorization and error boundary are stable; **diagnostic text is not**. No public structured bytecode schema or formatted-output version is selected.

## Authority, lifetime, and execution ownership

Inspection receives only values available in the **current entered Context and existing Actor/Process ownership domain**. It cannot discover or access another Context/Actor/Process, transfer non-transferable values, inspect a suspended Task through a new Task identity, reopen a foreign provider, force a callback or call host APIs under ambient authority. A foreign value may be classified as opaque; `std:interop` retains sole authority for explicit foreign operations. Results are detached snapshots with no retained Context/root/source-cache reference. No mutable physical Truffle object becomes a public Protos value.

Physical Bytecode/root trees are **not** equivalent to semantic Closure activations, inline callbacks, continuation frames, interpreter-stack capture or compiler IR. Existing error-stack diagnostics (`ProtosDiagnosticTraceCapture`) and host compiler tooling (`tools/truffle_compilation_gate.py`, `tools/truffle_root_diagnostic.py`) remain the owners of those other observations. `NodeUtil.printTreeToString` can print arbitrary internal fields, while `printSourceAttributionTree` may print full source: the implementation must not indiscriminately expose those dumps.

## Mandatory pay-as-you-grow / NORMAL / DEBUG proof obligations

Programs that do not import/use introspection must gain **none** of the following solely because LIB022 exists:

- eager inspection facility initialization or an eagerly populated per-Closure/Root registry;
- new per-call dispatch metadata, permanent source sections, extra NORMAL instructions, frame materialization, Task/Actor/Process creation, universal instrumentation listeners, new compiler subscriptions, unavoidable steady-state dispatch or startup cost.

Existing `ProtosCoreBootstrap.standardModuleMembers()` eagerly constructs some module facilities. Naively adding a `createFacility()` call to that eager bootstrap is **not** acceptable proof of import laziness. Prefer the least general new lazy provisioning route consistent with existing module ownership, Actor-local instances and frozen native facilities. Do not add a universal provider framework.

NORMAL remains its own optimized physical execution profile. DEBUG retains exact established debugger semantics. Requesting inspection must not implicitly switch profiles. On-demand materialization may change the explicitly inspected root's available metadata; any effects on group roots, JIT invalidation, concurrency and later execution must be bounded and validated. Native Image support must be tested rather than claimed from API presence.

## Comparative research and GITHUB010

Evidence sources:
- [Pinned GraalVM BytecodeRootNodes.java](https://github.com/oracle/graal/blob/vm-25.4.4.1.1/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/BytecodeRootNodes.java) — source-only versus instrumentation updates; group reparse/deoptimization;
- [Pinned BytecodeRootNode.java](https://github.com/oracle/graal/blob/vm-25.4.4.1.1/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/BytecodeRootNode.java) and [BytecodeNode.java](https://github.com/oracle/graal/blob/vm-25.4.4.1.1/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/BytecodeNode.java) — dump/source/structured instruction introspection;
- [Pinned NodeUtil.java](https://github.com/oracle/graal/blob/vm-25.4.4.1.1/truffle/src/com.oracle.truffle.api/src/com/oracle/truffle/api/nodes/NodeUtil.java) — compact tree versus raw field/source printing;
- [GraalPy builtins pinned source](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/GraalPythonModuleBuiltins.java) — explicitly gated disassembly, tree/storage diagnostics;
- [GraalJS pinned Debug builtins](https://github.com/oracle/graaljs/blob/bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/builtins/DebugBuiltins.java) — opt-in physical tree inspection.

D205 research additionally compared CPython `inspect/dis`, Ruby `RubyVM::InstructionSequence`, Lua `debug`, JVM/JFR and Node/V8 Inspector. These systems support a **separate unstable diagnostic surface**; they do not establish Protos semantics or demand a huge first API.

Candidate comparison in already-approved B architecture:

| 1–5 criterion | B1 metadata only | B2-min metadata + 2 inspector ops | **B2+ 5 total ops** | B3 broad structured inspector |
| --- | ---: | ---: | ---: | ---: |
| Correctness/invariants | 5 | 5 | 4 | 3 |
| Protos alignment | 4 | 4 | 5 | 3 |
| Present proportionality | 3 | 4 | 5 | 2 |
| Incremental growth | 5 | 5 | 5 | 3 |
| Future options | 5 | 5 | 5 | 3 |
| Scalability | 5 | 5 | 4 | 3 |
| Simplicity | 5 | 4 | 4 | 2 |
| Portability | 5 | 4 | 4 | 2 |
| Runtime/resource cost | 5 | 5 | 4 | 2 |
| Failure/operability | 4 | 4 | 4 | 2 |
| Deferral/reversibility | 3 | 3 | 5 | 3 |
| Evidence maturity | 5 | 4 | 4 | 2 |

Confidence: HIGH on small stable metadata and existence of pinned Truffle introspection APIs; MEDIUM for safely integrating four opt-in operations with context-owned roots; LOW for B3's speculative structured format/Native Image parity. Scores are aids, **not** arithmetic permission to violate an invariant. B1/B2-min under-deliver existing introspection motivation; B3 overengineers structured formats, listeners and compatibility. B2+ is the smallest present-useful design while remaining incrementally extensible.

Strongest objection to B2+: root materialization, source retention and explicit inspection may perturb shared runtime execution or expose internal data. The mitigation is exact entered-Context ownership, bounded safe text formatting, authorization, on-demand-only updates and targeted integration/resource tests; a failed obligation is a blocker, not a license to broaden authority.

Future scenarios: many Actors and Contexts, large/long-lived bytecode roots, concurrent inspection, P transfer, resumed/cancelled Tasks, Native Image, alternate backends and Truffle updates. B2+ survives by offering *detached snapshots*, no cross-boundary observation, no permanent registry, and implementation-specific diagnostics. If structured consumers become necessary, add a new explicitly versioned operation later rather than silently stabilizing current text.

## GITHUB021 approved invariant / delta reconciliation

| Previously approved Candidate B invariant | B2+ design | Required implementation evidence |
| --- | --- | --- |
| Portable metadata distinct from optional internals | PRESERVED | Two modules, distinct stability |
| No privilege escalation from import | PRESERVED | Context option default false and Error on unadmitted inspection |
| No universal registry | PRESERVED | No global root/callable tracking |
| No unavoidable cost for nonusers | PRESERVED | Structure/performance proof pending |
| NORMAL stays genuinely minimal | PRESERVED | No new default bytecodes/frames; proof pending |
| DEBUG unchanged | PRESERVED | Same scopes/provenance; proof pending |
| Existing diagnostic stack authoritative | PRESERVED | No synthetic stack or Task |
| Existing compiler tooling reused | PRESERVED | No JIT graph/heap APIs in guest |
| No fabricated per-callable compilation state | PRESERVED | No `isCompiled` |
| Context/Process/Actor/Task/continuation ownership | PRESERVED | Current-Context projection/admission tests pending |
| JVM and Native Image truthfulness | PRESERVED | Separate integration tests; no silent substitutes |
| No Java reflection / raw Truffle handles | PRESERVED | Safe Strings and opaque foreign categories only |

**Material delta versus initial Candidate B architecture:** exact API paths/names/shapes, `protos.Inspect` permission, frozen metadata result, Error/unsupported semantics, unstable text outputs, and **explicitly allowed on-demand materialization and preparation**. These are included in the owner's **B2+ exact approval**; no earlier Candidate B invariant is replaced or reopened. In particular, opt-in expensive work is permitted; permanent cost charged to nonusers is not.

`DECISION_INVARIANT_CONSISTENCY=PASS` at public-contract level; executable evidence remains `NOT_VERIFIED`. No independent new PLAT decision has been shown necessary.

## Explicitly deferred

No public per-callable JIT status, compiler IR/events/bailouts, heap dump/RSS, continuously captured logical stacks/scopes, cross-Context/Actor/Process discovery, arbitrary host reflection, raw Truffle handles, or stable structured instruction serialization. Additional tags/instrumentation are **not** silently enabled by the initial operations. Such features can be added with separate proof/approval and do not require the first release to pre-build them.

## Implementation handoff and closure boundary

The planned cohesive implementation sequence is **LIB022-A** (lazy facilities and `std:runtime.info()`, host inspection option scaffolding and targeted tests) and **LIB022-B** (the complete four-operation `std:inspect`, secure Context-root adapter, tests and docs), both in `guillermomolina/protos`. They are implementation work, not permission to alter the approved contract. The current product checkout HEAD when the human begins execution remains authoritative. `pom.xml` and `CHANGELOG.md` are changed **only after tests pass**, immediately before human commit/push; no tests after those late metadata edits. Repository-product publication remains human-executed. This decision publication is documentation-only and does **not** assert product validation.

Current D205 native parent link to LIB022/#921 was recorded as unresolved due to the available GitHub coordination interface; textual parent prose is **not** verification of GitHub's native relation. Closure of D205 requires re-reading the exact published project-docs revision, live Issue state and satisfying GITHUB015 structural postconditions; until then, coordination may remain open without questioning the substantive approval.

```text
FORMAL_DECISION=D205
PARENT=LIB022/#921
SELECTED_ARCHITECTURE=CANDIDATE_B
SELECTED_PUBLIC_CONTRACT=B2_PLUS
RUNTIME_MODULE=std:runtime
RUNTIME_OPERATIONS=info/0
INSPECTION_MODULE=std:inspect
INSPECTION_OPERATIONS=representation/1,callableInfo/1,bytecode/1,truffleTree/1
INSPECTION_HOST_OPTION=protos.Inspect
INSPECTION_HOST_OPTION_DEFAULT=false
INSPECTION_RESULTS=String
RUNTIME_INFO_RESULT=FreshFrozenObject{implementation,implementationVersion,engineVersion,platform,executionMode}
UNSUPPORTED=ORDINARY_ERROR
ERROR_POLICY=FRESH_ORDINARY_ERROR
DIAGNOSTIC_TEXT_STABILITY=UNSTABLE
IMPORT_AUTHORITY_ESCALATION=NO
PAY_AS_YOU_GROW=MANDATORY
ON_DEMAND_PREPARATION_AND_MATERIALIZATION=PERMITTED
NORMAL_DEBUG_CHANGES=NONE
STRUCTURED_INSTRUCTION_FORMAT=DEFERRED
NEW_PLAT_REQUIRED=NO_CURRENTLY_PROVEN
IMPLEMENTATION_EXECUTED=NO
TESTS_EXECUTED=NO
OWNER_APPROVAL_PROVENANCE=2026-10-18_ACTIVE_INTERACTION_EXACT_B2_PLUS
GITHUB021=PASS_DESIGN_LEVEL_IMPLEMENTATION_NOT_VERIFIED
```
