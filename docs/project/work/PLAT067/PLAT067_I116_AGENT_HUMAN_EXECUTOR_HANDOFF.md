# PLAT067 / D208 / I116 — Agent & Human Executor Handoff

**Type:** self-contained, product-implementation handoff, **prepared but BLOCKED** until exact ratification of [PLAT067-B](https://github.com/guillermomolina/protos/issues/933) and [D208](https://github.com/guillermomolina/protos/issues/934).
**Implementation issue:** [I116/#935](https://github.com/guillermomolina/protos/issues/935).
**Product repo:** `guillermomolina/protos` (with two “mo”). The agent may edit but may not execute tests/builds/version/publication. **Project docs repo** `guillermomolina/protos-project-docs` (with two “mo”) is authorized for agent commit/push on this task only; that does not authorize product repo publication.
**Research baseline:** product `27cb31d20e517d8377e58963ea595e5ec16bfd15`; use **actual local HEAD** when coding, not the research SHA. The full evidence and actual per-file changes are [here](../../evidence/PLAT067/PLAT067_ONE_STREAM_RESEARCH_2026_10_19.md).

## Next slice — I116-A (IMPLEMENTATION, not investigation; currently blocked)

**Gate:** verify that PLAT067 Candidate B and the exact D208 public contract choice are explicitly owner-approved and durably ratified. Without this gate, **STOP before editing product source**. The current conversation authorized documentation and issue coordination, not those two design approvals. The implementation is already researched: do not open new PLAT research or browse other language repositories.

**Fixed goal:** only one semantic Bytecode DSL stream; all independently proven former NORMAL fast paths active for every Context; Truffle source/tags/probes on demand; DAP as host Engine service, not a debug guest mode; minimum Builder oracle unchanged for literal and optimized Boolean witness. No invented callback frame for optimized-out calls. No changes to D013 selection, to normative Closure semantics, to one-generator I111, or to C-prime/Task helpers.

**Scope I116-A:** remove `ProtosExecutionMode` and all propagation through `ProtosLanguage`, `ProtosLanguageContext`, `ProtosPolyglotRuntimeHost`, `ProtosSourceCompiler`, `ProtosModuleRuntime`, `ProtosStandaloneHostedSession`, `CanonicalToBytecodeLowerer`, `ProtosBytecodeClosureExecutionPlan`, `ProtosClosureExecutionPlan`, `ProtosClosureCallPreparation`, test helpers and consumer tests. Only then delete the enum. Preserve language/Context projection and the existing B5 ABI; remove precisely eight profile guards at research-HEAD positions ~1250/1533/2138/3239/3947/3993/8812/10506. Implement *the selected D208 metadata/option contract* in the same coherent publication. Complete debugger/RootTag/source/negative regression acceptance before versioning. I116-B, if needed, adjusts only passive source/tags and tooling acceptance, without new semantic instructions.

### Peer facts — already researched and sufficient (no further browsing)

- GraalPy one `PBytecodeDSLRootNode` includes optional `@Instrumentation` `TraceOrProfileCall` and `TraceLine`; `RootNodeCompiler` emits from one compiler: [root](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java), [compiler](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java).
- Truffle Bytecode DSL: `BytecodeConfig.DEFAULT` omits optional tags/metadata; `WITH_SOURCE` and `newConfigBuilder().addTag(...)` materialize lazily with `BytecodeRootNodes.update`; upgrades are monotonic, may invalidate compiled code and shift BCIs; parser replay deterministic/idempotent; `BytecodeLocation.update()` translates. `@Instrumentation` must have transparent stack effect and cannot stand in for semantic instructions: [UserGuide](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/docs/bytecode_dsl/UserGuide.md), [InstrumentationTutorial](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/src/com.oracle.truffle.api.bytecode.test/src/com/oracle/truffle/api/bytecode/test/examples/InstrumentationTutorial.java).
- SimpleLanguage shows a single DSL generator, tag instrumentation and selective root tags: [SLBytecodeRootNode](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java). GraalJS offers *AST* on-demand materialization; do not treat as equivalent Bytecode DSL lowering.
- Protos already has **one** `@GenerateBytecode` in `ProtosBytecodeRootNode`, with `enableTagInstrumentation=true`, `enableRootTagging=false`, `enableRootBodyTagging=false`, and explicit semantic `RootTag`. Preserve it. Truffle version **25.4.4.1.1**.

### Acceptance and independent oracles

1. Cold/default bytecode: `() => { 1 }` and `() => { true.ifTrue() { 1 } }` must have identical optimized two-root semantic topology and independent handwritten-Builder executable signature under both ordinary and DAP hosts; no `PrepareSendOne` or missing callback root fabricated by debugger. Same source provenance where real statement survives.
2. Retain terminal Closure, Boolean control, pure scalar, single-param and required-prefix optimizations with all old *non-mode* proof gates. Reject unsafe folds for effects/dynamic dispatch/capture/suspension/ReturnHome/default/rest.
3. Update `ProtosI101ProfileSpecializedLoweringTest`, `ProtosPerf032ExecutionModeTest`, `ProtosRuntimeMetadataEmbeddingTest`, and existing I101 independent oracle classes, source/scopes/DAP, I096/I107/I112/I115 and Task/C-prime regressions; no blanket deletion of tests.
4. Reparse: stable true RootTag, root count/order, RootCallTarget identity, BytecodeLocation correctness; only optional source/tag/probe instructions added; no new physical semantic operations or debug mode.
5. DAP: real loopback endpoint, `PROTOS_DEBUG_READY`, after-prepare/after-first-call attach within a DAP-enabled host, correct breakpoint/source/scope on **surviving** operations, exact lexical ABSENT vs `null` values; normal host still does not start DAP.
6. Preserve independent `protos.Inspect` authority, std:inspect API, public spec per selected D208, 5-slot B5 call header, ReturnHome/ensure/Error, Actors, Tasks, I/O C-prime yield/resume and Native Image when applicable.

### Agent / Human Executor rules

AGENT: inspect `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, `spec/AGENTS.md`, local source/tests and the two ratified decisions. Local repo already HEAD. Apply Python patches rather than unified diff and `grep` rather than `rg`; no `git fetch/pull/reset/checkout` or product git write, tests, builds or benchmark. Source edits may temporarily be uncompilable **within the same cohesive I116-A patch**; do not ask for build until the batch series is fully coherent. Preserve concurrent edits.

HUMAN: after the agent finishes a coherent code/test edit, run `git diff --check`, relevant static/focal checks, then `make check` and `make test` as required by actual affected-file policy. No tests need repeating if no code changes. Separate DAP and native checks as needed. **Only after tests PASS** amend `pom.xml` and `CHANGELOG.md` immediately before exact-path add/commit/push; run **no tests after these metadata edits**. The agent may afterward coordinate issues and publish revision-bound evidence in docs.

## First editing batch — Python mechanical front-end conversion

**Use this batch ONLY after BOTH approval/ratification gates pass, while preparing the complete I116-A implementation.** It intentionally updates only the parse/source/module front-end; the same I116-A patch **must** migrate lowerer constructors, Closure plans, Context and DAP before compile/test. The script verifies exact current HEAD text before changing; STOP if assertions fail (concurrent edits), inspect and adapt, never overwrite unknown changes. Use this in the **product repository root**:

```bash
python3 - <<'PY'
from pathlib import Path

base = Path("src/main/java/com/guillermomolina/protos")
patches = {
    base / "execution/host/embedding/ProtosSourceCompiler.java": [
        (
            "     * I101-A (PLAT058-D): compiles {@code source} under the immutable physical compilation\n"
            "     * {@code profile} of the Context that owns the resulting executable. The caller resolves that\n"
            "     * profile at its language-entry boundary; the compiler never guesses it.\n",
            "     * Compile using the unique proof-optimized semantic Bytecode DSL emission stream.\n",
        ),
        (
            "RootCallTarget compileBytecode(\n            Source source, ProtosLanguage language, ProtosExecutionMode profile) {",
            "RootCallTarget compileBytecode(Source source, ProtosLanguage language) {",
        ),
        ('        Objects.requireNonNull(profile, "profile");\n', ""),
        (
            "new CanonicalToBytecodeLowerer(language, source, bindingAnalysis, profile)",
            "new CanonicalToBytecodeLowerer(language, source, bindingAnalysis)",
        ),
    ],
    base / "execution/host/embedding/ProtosLanguage.java": [
        (
            "        ProtosExecutionMode profile = ProtosLanguageContext.current().executionMode();\n",
            "",
        ),
        (
            "this, sourceCompiler.compileBytecode(request.getSource(), this, profile))",
            "this, sourceCompiler.compileBytecode(request.getSource(), this))",
        ),
    ],
    base / "execution/modules/ProtosModuleRuntime.java": [
        (
            "languageContext.materializeModuleSource(source),\n"
            "                languageContext.languageForRuntime(),\n"
            "                languageContext.executionMode());",
            "languageContext.materializeModuleSource(source),\n"
            "                languageContext.languageForRuntime());",
        ),
    ],
}
for path, replacements in patches.items():
    content = path.read_text()
    updated = content
    for old, new in replacements:
        if updated.count(old) != 1:
            raise RuntimeError(f"STOP: expected exactly one matching span in {path}: {old[:90]!r}")
        updated = updated.replace(old, new, 1)
    if updated != content:
        path.write_text(updated)
    print(f"EDITED {path}")
PY
```

Do not run compilation after this **partial** batch. The follow-up agent edit batch removes lowerer `profile` constructor+all eight proof gates and migrates Context/Closure/host contracts and tests; the full I116-A patch must be coherent before a human validation request. Verify scopes/source after I116-B or earlier if all tests already pass.

### Handoff state

`PLAT067_APPROVAL=PENDING`
`D208_APPROVAL=PENDING`
`I116_STATUS=BLOCKED`
`PRODUCT_WRITES=NONE`
`TESTS=NOT_RUN`
`GITHUB_PRODUCT_PUBLICATION=HUMAN_ONLY`

This document does not authorize the agent to reclassify the design as approved.
