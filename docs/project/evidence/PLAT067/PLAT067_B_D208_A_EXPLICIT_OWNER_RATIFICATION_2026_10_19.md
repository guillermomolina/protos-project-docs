# PLAT067-B + D208-A — Explicit combined owner ratification, 2026-10-19

**Evidence type:** immutable approval provenance and bounded GITHUB021 reconciliation. This is not an implementation, test run or normative product change.

## What the owner approved

The project owner was explicitly asked to approve both separately formulated and linked recommendations after the complete PLAT067 research, created D208 and I116 issues, and prepared gated implementation handoff. The immediately preceding project status stated that **implementation required express approval of “PLAT067-B” and “D208-A”** and described:

1. **PLAT067-B:** one proof-optimized semantic Bytecode DSL stream for both ordinary and DAP-enabled Contexts; native lazy Truffle instrumentation; retain former NORMAL minimum and independent I101 Builder oracles; truthful debugger observation limited to surviving optimized sites, with no reconstructed callback CallTarget.
2. **D208-A:** change `std:runtime.info/0` from exactly five frozen own String slots to exactly **four** by removing `executionMode`; remove experimental `protos.Debug` as a guest compilation-mode option; retain real Graal DAP/CLI host service, source/scopes and independent `protos.Inspect` host authorization.

**Exact current owner statement:**

> ratifico ambas decisiones.

This is a direct, affirmative choice of **both** specified alternatives, not merely authorization to research, publish documentation or start implementation. No extra public schema, arbitrary debugger behavior or performance improvement was separately approved.

## Canonical ratified records, source evidence and implementation

- [PLAT067 B — platform architecture](../../decisions/platform/PLAT067_UNIFIED_PROOF_OPTIMIZED_BYTECODE_DSL_WITH_LAZY_TOOLING.md), owning [PLAT067/#933](https://github.com/guillermomolina/protos/issues/933).
- [D208 A — public metadata semantics](../../decisions/language/D208_RUNTIME_METADATA_WITHOUT_EXECUTION_MODE.md), owning [D208/#934](https://github.com/guillermomolina/protos/issues/934).
- [PLAT067 original source-backed research](PLAT067_ONE_STREAM_RESEARCH_2026_10_19.md), pinned [product baseline `27cb31d20e517d8377e58963ea595e5ec16bfd15`](https://github.com/guillermomolina/protos/commit/27cb31d20e517d8377e58963ea595e5ec16bfd15).
- [I116 consolidated implementation and human executor handoff](../../work/PLAT067/PLAT067_I116_AGENT_HUMAN_EXECUTOR_HANDOFF.md), [I116/#935](https://github.com/guillermomolina/protos/issues/935). Implementation is **now unblocked**, but the agent may not build/test/commit/push the product.
- [D205 historic B2+ ratification](../../decisions/language/D205_LIB022_RUNTIME_INTROSPECTION_B2_PLUS_PUBLIC_CONTRACT.md) remains an honest historical record. D208 **prospectively** supersedes only the `executionMode` field/slot-count and language `protos.Debug` guest-mode selector, not `std:inspect` or independent metadata.

## Guardrails that survive ratification

- Preserve D013, D179, B5 and B5+, PLAT014/041/044/060/061/062/065, ReturnHome, errors/ensure, Task/Actor/Process, C-prime and Context identity.
- Preserve single `@GenerateBytecode`, selective real semantic RootTag and independent I101 proof/Builder oracles; all eight old NORMAL conditions become **profile-independent, proof-guarded**, never unconditional fast paths.
- Retain DAP as separate host service; no bogus callback frame for an optimizer-eliminated call and no guarantee of re-creating a non-DAP Engine with network debugger attachment.
- `BytecodeConfig` source/tags/`@Instrumentation` upgrades are lazy, monotonic and deterministic; no second semantic debug stream.
- The changed `std:runtime.info()` result has exactly four guest String slots: `implementation`, `implementationVersion`, `engineVersion`, `platform`. Fresh, frozen, ordinary, delegated to Object, same on-demand observation and authority. No fake `NORMAL` or replacement debug capability field.
- Keep `protos.Inspect` unchanged and independent of DAP, and preserve the host DAP launch/readiness protocol.
- Update public normative `spec/semantics/MODULES.md` and spec changelog **only in approved product I116 implementation**, not in this documentation-only ratification.
- Never treat this record as test evidence: `PRODUCT_CODE_CHANGED=NO`, `TESTS_NOT_RUN=YES`, `PERFORMANCE_UNMEASURED=YES`. Implementation must validate all acceptance on actual local HEAD.

**Workflow distinction:** PLAT067 and D208 can have decision closure with their own ratified durable records; [I116/#935](https://github.com/guillermomolina/protos/issues/935) is **ready for implementation**, not implemented or closed. The project-docs repository has explicit owner permission for direct commits, but no product-repository Git authorization.
