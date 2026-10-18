# PERF042 — 0.3.376 real identity graph diagnostics published

Date: 2026-10-18
Formal owner: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871)
Classification: **published test-only diagnostic checkpoint**, not a performance optimization, stable cross-language comparison, or closure of PERF042.
Product revision: [`6ad7ab2864e8e48fc546dce5d4cd61472a026819`](https://github.com/guillermomolina/protos/commit/6ad7ab2864e8e48fc546dce5d4cd61472a026819)
Product parent: `63a178e3ea19af1caf3e2e43e947130c5bcfc792`
Product version: **`0.3.376-SNAPSHOT`**

## 1. Exact product publication

The maintainer reported pushing `6ad7ab2864e8e48fc546dce5d4cd61472a026819` to `guillermomolina/protos/main` and a clean working tree. The published commit was independently retrieved and its patch inspected. It changes **exactly three paths**:

1. `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf042IdentityBuilderOracleTest.java` — real-source-root diagnostics, retaining the original handwritten bound-argument oracle and the semantic invocation/parameter-binding assertions;
2. `pom.xml` — `0.3.375-SNAPSHOT` to `0.3.376-SNAPSHOT`;
3. `CHANGELOG.md` — the matching `0.3.376-SNAPSHOT` PERF042 entry.

**There are no `src/main/**`, normative `spec/**`, benchmark-harness, or production-runtime changes in this commit.** The locally introduced experiment `Perf042KnownIdentityCallOne` (a statically known `RootCallTarget` without normal language dispatch or fallback) was removed from the working tree before publication. The published product root therefore does **not** declare the artificial PERF042 operation.

The test-only diagnostics preserve:
- `identityBodyComparedWithBoundBuilder`: handwritten Bytecode DSL B5 argument-binding lower bound plus real Closure identity and argument-semantics checks, intentionally not asserting identical physical bytecode;
- `bareIdentityTopLevelWithoutRun`: real `identity: (value) => { value }\nidentity(1)` with exactly one top-level and one nested source root;
- `captureIdentityRootAtRequestedTier`: opt-in compilation of the real lowered `identity` Closure root;
- `captureCompleteRunRootAtRequestedTier`: opt-in compilation of the actual bare-program top-level root, independently for Tier 1 and Tier 2, using synchronous Truffle compilation and stable root identity, with BGV attribution, DEFAULT instruction enumeration and separately labelled COMPLETE instrumentation inspection.

The handwritten Builder is explicitly a **physical lower-bound oracle**, not a replacement implementation of the observable Protos Closure contract.

## 2. Human-executed validation

The maintainer provided these observations after cleaning the experiment and before version/changelog finalization:

| Gate | Reported result |
| --- | --- |
| `git diff --check` | Clean |
| `git status --short` / `git diff --stat` | Only the PERF042 test file modified, +462 lines |
| `mvn -q '-Dtest=ProtosPerf042IdentityBuilderOracleTest' test` | PASS |
| `PERF042_BOUND_ARGUMENT_LOWER_BOUND` | PASS |
| `PERF042_I104_A_BYTECODE_SHAPE` | PASS |
| `PERF042_IDENTITY_SEMANTICS` | PASS |
| `make test` | PASS, maintainer reported |
| `make check` | PASS, maintainer reported |
| Final `git push origin main` | `63a178e3..6ad7ab28`, maintainer reported and exact commit read back |
| Post-push working tree | Clean, maintainer reported |

The physical oracle printed `PERF042_BOUND_ARGUMENT_BYTECODE_EQUAL=false`; this is **not** a validation failure: the manual lower-bound Builder and real Closure have different instruction sequences and the test intentionally compares rather than demands their equality. The semantic assertions and focal test passed.

These are maintainer-reported execution results, not an independently stored test-run manifest. No agent ran or reran product tests or compilation for this documentation checkpoint. Metadata finalization did not cause the already-green tests to be rerun.

## 3. Prior local BGV investigation — separate provenance

**Important:** Before this cleanup and product publication, the maintainer supplied a local `i112-identity-graphs.tar.gz` containing Tier-1 and Tier-2 BGVs and `maven.log` files for the bare program. That capture used a **locally modified experimental worktree** and **17 warmup calls** to reach `BytecodeTier.CACHED`. The archive has **not** been added to the published product commit or this evidence repository. It is a diagnostic observation, **not a clean `6ad7ab2864e8e48fc546dce5d4cd61472a026819` post-publication graph**.

For those exact prior local captures, both `After TruffleTier` graphs were successfully compiled with `BackgroundCompilation=false`:

| Metric (local prepublication capture) | Tier 1 | Tier 2 |
| --- | ---: | ---: |
| Top-level real `identity(1)` root, `After TruffleTier` IR nodes | 4,804 | 4,970 |
| `FrameState` | 839 | 893 |
| `IfNode` | 297 | 299 |
| `InvokeWithExceptionNode` | 66 | 70 |
| Estimated source-provenance ancestry: `InvokeClosureOneTail.definitionSource` | 3,332 | 3,494 |
| Estimated source-provenance ancestry: `prepareClosureCallOneFallback` | 2,343 | 2,343 |

Counts by Java source-position ancestry **overlap and must not be summed**. They show the generic prepared-call path retained *within the rejection branch of* `definitionSource`; they do not establish that `InvokeClosureOneTail.generic` executed or that the fallback is a hot-path cost. The observed prior local graphs also had many frame states and exception-capable invokes; those counts do not by themselves prove runtime heap allocations, executed invokes, performance loss, or a semantics-preserving removable-node budget.

The bare-program top-level module root is **not the same selected root** as the historical `primitive-closure-call` benchmark's wrapper `run` Closure. The historical GraalPy figure (50 selected graph nodes, Tier 2) is from a different pinned capture; it is **not** a current matched Tier-1/GraalVM/root/warmup comparison to Protos 4,804.

Independent earlier NORMAL Bytecode DSL acceptance for I112 used 11 FULL instructions in the module and 5 in the nested Closure (16 overall), but the number of DSL instructions is not a directly comparable count of Graal IR nodes. Current `0.3.376` introduces reproducible **opt-in** diagnostic tests without claiming a new stable comparative result.

## 4. Current hypothesis and remaining PERF042 work

The source-provenance investigation suggests a specific **falsifiable** candidate: in `InvokeClosureOneTail.definitionSource`, failure of `admittedInstanceOrNull` enters `prepareAndEnter` and generic `Object.call`/prepared-call handling. This fallback machinery can remain expanded under `CachedBytecodeNode`, even when the real direct Closure specialization is selected. GraalPy's `CallUnaryMethodNode` uses explicit megamorphic `@InliningCutoff`, but the effect of analogous Truffle specialization/inlining boundaries **has not been measured** under matched conditions and is not approved as an implementation change.

Next work stays under **PERF042/#871** (OPEN; status:ready; priority unset), with no new formal Issue:
1. Attribute actual live versus cold control/data paths in the retained BGV and check which fallback guards and inlining decisions cause the 2,343-node ancestry.
2. Determine whether an inlining boundary on the truly exceptional/generic fallback can reduce retained graph size while preserving dynamic `Object.call`, ReturnHome, captures, native calls, suspension/continuations, errors, and instrumentation.
3. Compare Protos and GraalPy in the *same* program shape, warmup, tier, selected root and Graal graph phase, and keep canonical benchmark admission/performance gates separate.

**No production optimization, change of semantics, benchmark rerun, issue closure, or cross-language graph-parity claim is authorized or implied by this documentation publication.**

## 5. Sources and evidence authority

- [Pinned Protos implementation/test-only commit](https://github.com/guillermomolina/protos/commit/6ad7ab2864e8e48fc546dce5d4cd61472a026819) and its exact published `pom.xml`/`CHANGELOG.md`.
- [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), including prior [PERF042-B evidence](PERF042_B_PUBLISHED_FUSED_DEFINITION_KEYED_CALL0_CALL1_2026_10_11.md) and [PERF042-A3 retained-trace assessment](PERF042_A3_RETAINED_COMPILATION_TRACE_READBACK_AND_IMPLEMENTATION_HANDOFF_2026_10_10.md).
- Human-reported focal and integrated validation in this interaction; user-supplied prepublication Tier-1/Tier-2 logs and BGV archive.
- [GraalPy 25.4.4 source: `CallUnaryMethodNode`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java) for **source comparison only**, not matched measurement.
