# LIB022-0 — Runtime introspection investigation and owner selection of Candidate B

**Date:** 2026-10-18  
**Product:** [guillermomolina/protos](https://github.com/guillermomolina/protos), observed `63a178e3ea19af1caf3e2e43e947130c5bcfc792` (`0.3.375-SNAPSHOT`)  
**Work item:** [LIB022 / #921](https://github.com/guillermomolina/protos/issues/921)  
**Public contract decision:** [D205 / #922](https://github.com/guillermomolina/protos/issues/922)  
**Document role:** durable investigation and owner-selection evidence; non-normative; no product code or specification change

## Approval provenance and exact scope

In the active owner interaction of 2026-10-18, after receiving the LIB022-0 comparative investigation and its expressly unapproved recommendation, the project owner explicitly stated **"apruebo candidato B"**. This is approval of *Candidate B* as the architectural decomposition described in the investigation, not approval of example names, signatures, result formats, errors, new host authority, or implementation. There is no claim of a previously existing separate owner vote.

### Owner-selected architectural boundary — B

1. A **small, portable, read-only runtime metadata surface**, with real runtime/context information rather than invented optimizer/function-compiled status.
2. **Separate, explicitly opt-in, implementation-dependent value/callable/bytecode/physical-tree diagnostics**. Its formats are not automatically public compatibility promises.
3. **Existing developer/host tooling** remains the owner of compiler compilation events, inlining/bailouts/graphs, privileged heap operations and other host-controlled diagnostics; do not duplicate it in ordinary guest libraries.
4. **Pay-as-you-grow** is mandatory: a program not using introspection must not pay new unavoidable startup, allocation, frame, instrumentation, metadata retention, source reconstruction, JIT, Task/Actor/Process, or steady-state costs.
5. **NORMAL remains minimal; DEBUG retains its established semantics**. Introspection must not cause implicit profile changes, frame materialization on ordinary calls, or global diagnostic subscriptions.
6. Preserve current Context/Actor/Process/Task/continuation ownership, ordinary Protos error rules, existing foreign `std:interop` authority, and JVM/Native truthfulness. No unbounded Java reflection, mutable raw Truffle nodes, or ambient privilege to guest code.

### Explicitly not selected by the owner

- The example module names `std:runtime` and `std:inspect`.
- The example operations `info()`, `describe(value)`, `callableInfo(callable)`, `bytecode(callable)`, `truffleTree(callable)`.
- Any precise result object schema, omission/null/unsupported policy, error prototype, source metadata disclosure, or wire/diagnostic format version.
- Claims of per-callable `isCompiled()`, heap dumps, RSS, stack capture or compiler IR as portable guest APIs.
- A mandatory full inspector in the first implementation, new Java bridge design, Native Image feature parity, or actual implementation authorization.

D205 owns the independent exact public contract decision under GITHUB010/GITHUB021 and remains unresolved. Candidate B itself must not be reopened merely because these finer decisions are pending.

## Investigation summary

The original investigation compared four decomposition models:
- **A:** minimal portable runtime information only.
- **B (owner selected):** stable runtime information plus optional implementation-specific diagnostics, while reusing external/host tooling for compiler and privileged operations.
- **C:** use developer tools only, with no guest API.
- **D:** unified stateful diagnostic provider / session subsystem.

B was preferred because it combines the ordinary-program metadata use case and intentionally invoked structural inspection without requiring a permanent instrumentation framework. D imposes premature state and authority complexity; C fails to serve the small guest metadata use case; A cannot serve the interpreter-inspection motivation alone.

### Upstream evidence (specific source revisions)

**GraalPy:** [`GraalPythonModuleBuiltins.java`](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/GraalPythonModuleBuiltins.java). It includes `get_graalvm_version`, `get_jdk_version`, `is_native`, `is_forced_uncached_interpreter`, `which`, `dis`, `dump_truffle_ast`, `get_storage_strategy`, `dump_heap`, `get_current_rss`, `tdebug` and a compiler-bailout test hook. `dis` uses `PCode.toDisassembledString(quickened)`; AST printing uses `NodeUtil.printTreeToString`; storage strategy exposes the concrete representation class. [`PythonOptions.java`](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/PythonOptions.java) defaults `EnableDebuggingBuiltins` to false, and the module disables multiple debugging operations when the option is off. The user-reported distinct Python module named `graalvm` was **not independently identified**; do not equate its identity or public exports with `__graalpython__`.

**GraalJS:** [`DebugBuiltins.java`](https://github.com/oracle/graaljs/blob/bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/builtins/DebugBuiltins.java) implements `dumpFunctionTree(fn)` using a JS function call target, `RootCallTarget.getRootNode()`, and `NodeUtil.printTree`. Other debug builtins expose shapes, backing classes and array representations; `neverPartOfCompilation` performs a deliberate bailout and should not be conflated with inspection. [`JSContextOptions.java`](https://github.com/oracle/graaljs/blob/bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSContextOptions.java) defines `js.debug-builtin` as internal, non-API and off by default. [`JSRealm.java`](https://github.com/oracle/graaljs/blob/bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSRealm.java) implements `Graal.isGraalRuntime()` via the runtime name; it does **not** establish that any particular function is compiled.

**Broader comparison:** public, but implementation-dependent bytecode and runtime inspection in CPython/Ruby; hosted JVM/JFR diagnostics; and GraalVM's Polyglot/Truffle mechanisms illustrate why inspectable representations must be distinguished from stable language semantics and host instrumentation. Full revision-specific API availability in Protos' pinned GraalVM version requires follow-up validation, not extrapolation from upstream `master`.

### Current Protos evidence at observed product revision

- `pom.xml` declares `graalvm.version=25.4.4.1.1`; `ProtosPolyglotRuntimeHost` owns the shared `Engine` and Process-scoped `Context`s.
- `ProtosLanguageContext` and `ProtosExecutionMode` select immutable NORMAL/DEBUG execution; `ProtosLanguage` exposes the corresponding context option. DEBUG does not mean always-on compilation tracing.
- `ProtosSemanticBytecodeRootNode` owns generated semantic Bytecode DSL roots, with selective `RootTag` emission and no equivalence between a physical Truffle root and every semantic activation.
- `ProtosClosureExecutionPlan` retains callable plan/physical-root substrate. An inspector should project safe snapshots, not expose live compiler objects.
- `ProtosDiagnosticTraceCapture` already reconstructs failure traces, including semantic source roots, inlined callbacks and resumed terminal caller frames on error paths. Do not duplicate this as eager, ordinary-call `stack()` machinery.
- `ProtosDiagnosticRootIdentity`, `tools/truffle_compilation_gate.py`, and `tools/truffle_root_diagnostic.py` already support compilation root identity, targeted compilation and graph/trace evidence. These are developer tools, not currently portable guest-library operations.
- `protos/lib/interop.protos` supplies a deliberately narrow explicit foreign operation surface. Inspection of Protos objects must not grant arbitrary foreign-host access.
- Governing normative surfaces include `spec/semantics/CALLABLES.md`, `MODULES.md`, `VALUES_AND_COLLECTIONS.md`, `ERRORS.md`, `EXECUTION_AND_CONTROL.md` and `spec/concurrency/ACTORS.md`, `PARALLEL_EXECUTION.md`.

### Known capability placement after architectural selection

| Capability | Selected home | Notes |
|---|---|---|
| Runtime/language/engine metadata | Small portable guest module candidate | Actual public contract pending D205 |
| Representation/Closure metadata | Optional diagnostic inspector candidate | Physical output unstable; no privileged handles |
| Bytecode instructions and Truffle root tree | Optional inspector or deferred tools | Must distinguish source/semantic/physical/compiler representations |
| Compilation events, inlining, bailouts, Graal IR/BGV | Existing host/developer tooling | Avoid duplicate live collection |
| Heap dumping and privileged process diagnostics | Existing host tooling | No import-time filesystem privilege |
| Failure stack and debugger | Existing Protos trace/DAP | No redundant frame-retention path |
| Per-callable compiled status | Deferred | Availability of optimizer is not compiled state |

## GITHUB021 invariant/delta check for selected B

| Explicit selected invariant | Status in recorded recommendation |
|---|---|
| Small portable metadata vs opt-in diagnostics | Preserved |
| Developer-tool boundary for privileged compiler/heap actions | Preserved |
| No cost for non-users; NORMAL minimal | Preserved as acceptance criterion, not benchmark proof |
| No new hidden authority or cross-context leakage | Preserved as required behavior, adapter design pending |
| Honest compiler/Native status | Preserved; exact support requires checking |
| No premature public API ratification | Preserved: D205 remains pending |

No existing owner-approved invariant is intentionally reopened by the high-level selection. D205 must repeat this check against any later exact proposal.

## Future work and validation

D205 investigates and obtains explicit owner approval of module names, exact operations, result contracts, errors and compatibility. Only then should LIB022 group implementation by actual cohesive change surfaces (initial runtime metadata, optional inspector as justified). A new PLAT issue is conditional on proving a separate independently blocking lifecycle/authority decision. No new Ixxx issue is appropriate for research; future implementation work uses its proper work family and repository governance.

No shell commands, tests, benchmarks or patches were executed for LIB022-0. No product files, normative specification, or agent policy changed. This record documents research and approval scope only; **it is not an implementation or performance validation report**.

```text
WORK_ITEM=LIB022-0
TYPE=INVESTIGATION
OWNER_SELECTION=Candidate_B
OWNER_SELECTION_DATE=2026-10-18
APPROVED_ARCHITECTURE=SMALL_PORTABLE_METADATA__SEPARATE_OPTIONAL_DIAGNOSTICS__HOST_COMPILER_TOOLS
PUBLIC_API_APPROVED=NO
PENDING_DECISION=D205/#922
IMPLEMENTATION_AUTHORIZED=NO
PRODUCT_REVISION=63a178e3ea19af1caf3e2e43e947130c5bcfc792
UPSTREAM_GRAALPY_REVISION=7e7182b1036d56f8e89ecc9c7a86cee292180aa6
UPSTREAM_GRAALJS_REVISION=bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d
TESTS_EXECUTED=NO
CODE_MODIFIED=NO
```
