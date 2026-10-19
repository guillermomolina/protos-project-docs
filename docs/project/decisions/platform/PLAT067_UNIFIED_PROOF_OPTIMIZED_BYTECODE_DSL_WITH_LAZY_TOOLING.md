# PLAT067 — Unified proof-optimized Bytecode DSL with on-demand Truffle tooling (Candidate B)

**Status:** RATIFIED — exact project-owner approval **“ratifico ambas decisiones”**, 2026-10-19, in direct response to the explicitly enumerated **PLAT067-B** and **D208-A** selections.
**Decision Issue:** [PLAT067/#933](https://github.com/guillermomolina/protos/issues/933).
**Semantic companion:** [D208/#934](https://github.com/guillermomolina/protos/issues/934); [language-domain ratification](../language/D208_RUNTIME_METADATA_WITHOUT_EXECUTION_MODE.md).
**Implementation owner:** [I116/#935](https://github.com/guillermomolina/protos/issues/935), implementation not yet executed.
**Source basis:** [guillermomolina/protos@27cb31d20e517d8377e58963ea595e5ec16bfd15](https://github.com/guillermomolina/protos/commit/27cb31d20e517d8377e58963ea595e5ec16bfd15), read-only inspected baseline for architecture research, **not an implementation checkout requirement**.
**Research:** [PLAT067 complete code/peer/trade-off evidence](../../evidence/PLAT067/PLAT067_ONE_STREAM_RESEARCH_2026_10_19.md); [I116 agent and Human Executor implementation handoff](../../work/PLAT067/PLAT067_I116_AGENT_HUMAN_EXECUTOR_HANDOFF.md).
**Role:** concrete non-normative host/Truffle architecture. Observable runtime-metadata schema is exclusively governed by D208 and eventual ratified `spec/` changes.

## Selection and scope of exact owner approval

The owner had been presented the two precise choices:

- **PLAT067-B:** one always-proof-optimized semantic Bytecode DSL emission stream, former I101 NORMAL minimum for every Context; Truffle tags/source/`@Instrumentation` only on demand, no second DEBUG semantic lowering, with debugger observation limited to surviving executable sites and no fictitious callback frames; optional tooling upgrade monotonic, DAP remains a real Engine/host service.
- **D208-A:** remove the public `std:runtime.info().executionMode` member and retire the `protos.Debug` language option (see companion normative-design record), without dropping external DAP support.

The owner responded: **“ratifico ambas decisiones.”** This is explicit approval of **B and A exactly as proposed**, not delegation to pick another architecture, assert missing test results, or change unrelated guest semantics.

## Selected architecture: one physical semantic stream

1. Keep exactly one generated Truffle Bytecode DSL interpreter `ProtosBytecodeRootNode` (I111), with `enableTagInstrumentation=true`, `enableRootTagging=false`, `enableRootBodyTagging=false`, and manual `RootTag` only for genuine guest-semantic roots/inline callback regions. Auxiliary Task/C-prime/I/O roots are not guest frames.
2. Keep one `CanonicalToBytecodeLowerer` and its proof-driven Builder emission, with **no `ProtosExecutionMode` parameter, Context-owned compilation profile or profile-specific semantic/root-topology output**. Former NORMAL fast paths are considered for all ordinary/DAP-enabled Contexts whenever their *independent semantic proofs* pass, not forced unconditionally and not disabled for debugging.
3. Preserve `BytecodeConfig.DEFAULT` and lazy source/tag/probe generation through retained deterministic/idempotent builder replay and `BytecodeRootNodes.update` or API-equivalent path. Upgrades are **monotonic**, can invalidate compiled code and change BCIs, and use `BytecodeLocation.update` when reusing locations. Do not promise that disabling instrumentation restores the original physical instructions.
4. Treat DAP as a **host Engine tooling service**; retain `ProtosPolyglotRuntimeHost.openDebug`, actual Graal DAP options, loopback/ephemeral endpoint, `PROTOS_DEBUG_READY`, CLI and external debugger behavior. Do not claim that late DAP network attachment can retrofit a formerly non-DAP Engine.
5. Preserve correct NodeLibrary lexical scope projection of actual live B5 frames, frame-native bindings, inline regions/captures and their ABSENT semantics; nothing fabricates a guest frame or CallTarget eliminated by sound proof-based lowering.
6. Preserve all existing guest contracts: D013 ordinary call-slot semantics; ReturnHome/unwind/ensure/errors; D179 lexical membership; PLAT060 B5 universal five-slot ordinary call ABI and B5+ extensions; PLAT014 continuation, Task/Actor/Process/IO C-prime; Context/language identity and projection; standard and host `Inspect` authority. No new semantic `@Operation` or instrumentation-only success-path tax is authorized just to emulate DEBUG.

### All eight code-identified profile gates

The previous product revision has conditional gates in `CanonicalToBytecodeLowerer.java` at **approximately** 1250 (constant body), 1533 (scalar locals), 2138 (unary identity), 3239 (required prefix), 3947 (module prolog authority), 3993 (terminal call), 8812 (Boolean control) and 10506 (literal Closure-call inlining). These baseline line numbers are evidence only; I116 must inspect the actual local HEAD. Delete the **profile condition**, but retain independent arity/effect/ownership/capture/suspension/ReturnHome/fallback conditions and structural Builder proofs.

## Deliberate trade-off: truthful debugging of optimized programs

Before I101 optimization, a source expression `() => { true.ifTrue() { 1 } }` could retain a real callback CallTarget; the physical NORMAL path instead provably folds it to the same bytecode/root group as `() => { 1 }` (two physical roots for module + Closure, not three). **This ratification explicitly accepts** that Truffle instrumentation cannot step into a callback whose CallTarget was never emitted. Debugging must reflect the **surviving optimized execution**: source, stepping, breakpoints and scopes for actual executable locations. No fake RootTag, ghost callback scope, de-optimizing into a second semantic instruction stream, or physical callback resurrection is allowed. DAP integration remains required, with observable limitations documented/truthfully tested.

### Native Truffle mechanism / peer proof

The Truffle [Bytecode DSL UserGuide](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/docs/bytecode_dsl/UserGuide.md) confirms optional `BytecodeConfig` instrumentation, tag/source materialization, deterministic reparse and irreversible-once-added instructions. `@Instrumentation` permits zero-child/zero-result or one-child/one-result transparent stack effects, **not** a replacement for arbitrary semantic lowering. [GraalPy's PBytecodeDSLRootNode](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) and [RootNodeCompiler](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) provide the one-compiler / optional trace instrumentation peer. [SimpleLanguage](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) provides a second one-DSL precedent. GraalJS dynamic tagged AST materialization is informative **but not a Bytecode DSL model**; TruffleRuby, Espresso, Sulong and Pkl are separate instrumentation precedents, not assumed structurally equivalent.

## GITHUB021 — exact supersession, retained invariants

| Prior owner-ratified record | What PLAT067-B **supersedes** | What PLAT067-B **retains** |
|---|---|---|
| PERF032/#831 | Immutable guest NORMAL/DEBUG Context/Engine execution mode as physical code-selection input | Same guest semantics, accurate real activation and cross-Context isolation, normal host without DAP |
| PLAT058-D/#891 | Immutable Context-owned NORMAL-vs-DEBUG *physical lowering profile* and separate DEBUG semantic/root topology | One canonical lowerer, proof-driven minimal emitted semantic bytecode, independent handwritten Builder-oracle acceptance, deterministic reparse |
| I101/#892 (implemented) | Assertions that separate physical DEBUG code is required | Every independent NORMAL admission proof, two-root minimal witness, exact effective-bytecode signatures and negative cases |
| PLAT065-A/#918; I111/#919 | No architectural reversal required | One generated interpreter, selective semantic RootTag, no auxiliary C-prime guest frames |
| PLAT060-B5/#895; I102/#896 | Remove obsolete references to profile-dependent frame transport, **not ABI semantics** | The same universal B5 (and eligible B5+ addendum) physical ABI and identity |
| D205/#922 | **Nothing** in this PLAT; D208 has independent semantic authority to revise one metadata field | All other `std:runtime` and `std:inspect` contracts subject to D208's bounded amendment |
| PLAT014/041/044/057/061/062/066 | Only wording that mechanically assumes a permanent NORMAL/DEBUG mode must be reconciled by I116 | Semantics, proofs, continuation, root identity, D013 and pay-as-you-grow remain unchanged |

Do not rewrite historical ratifications retroactively. This decision prospectively supersedes only the specifically enumerated physical-profile invariants, while historical releases/tests remain truthful for their own commits. D208 is the sole decision authority for the public contract change; normative `spec/` remains authoritative after I116 applies its approved amendment.

## I116 implementation, falsifiers and publication gates

**I116-A (coherent consolidated patch):** remove enum/profile plumbing throughout host/compiler/modules/lowerer/Closure caches/reprojection; preserve optimized path and real DAP Engine factory; rewrite obsolete split tests while retaining independent Builder-oracle proofs; apply only approved D208 metadata/option/spec amendment. **I116-B (only if needed):** targeted passive source/tag placement and DAP/scopes/exception/continuation regression hardening, no second semantic stream. Avoid micro-slices. All source edits against actual local HEAD, preserving concurrent work. Product agent edits; human executes compilation/tests and Git product publication, with version/changelog changed only after green substantive tests.

**Falsification / STOP conditions:** minimum oracle degrades; semantic proof guard removed inadvertently; two semantic streams remain; reparse changes root-group/CallTarget identity; instrumentation invents optimized-away callback; DAP loses real surviving-source scopes; B5/ReturnHome/cancellation/Task/C-prime breaks; new overhead in permanently non-instrumented paths; normative D208 migration is incomplete; implementation reveals an unresolved substantive choice. Any new substantive choice needs separate owner approval, not interpretation of this ratification.

**Evidence status at ratification:** `PLAT067_B=RATIFIED`; `D208_A=RATIFIED`; `I116_READY_FOR_HUMAN_EXECUTOR_WORK`; `PRODUCT_CHANGED=NO`; `TESTS=NOT_RUN`; `PERFORMANCE_DELTA=UNMEASURED`. No claims of tested implementation, deployed debugger, native parity or benchmark improvement.
