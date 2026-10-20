# PERF042 / PLAT066-C — 0.3.386 bare TOP_LEVEL → identity: three-metric measurement accepted; run → identity next

**Date:** 2026-10-20. **Kind:** maintainer-supplied execution/compilation evidence and explicit **scenario-scoped acceptance**, not a performance benchmark, whole-runtime parity proof, or design ratification.

**Product:** [`guillermomolina/protos@607973c493dcb0c5ba3a7eeba9897b045eb049d8`](https://github.com/guillermomolina/protos/commit/607973c493dcb0c5ba3a7eeba9897b045eb049d8), `0.3.386-SNAPSHOT`, GraalVM CE **25.4.4.1.1**, JVM.

**Benchmark harness:** `guillermomolina/protos-benchmarks@1b28b390b61d5a59e9b1573ea30e26306a6dde86`, GraalPy JVM dependencies **25.4.4.1.1**, inside Podman using the benchmark devcontainer image `protos-benchmarks-graalpy:25.4.4`. This benchmark run used an **untracked temporary Java driver** and files under `.work/`; no product or benchmark tracked sources were edited.

**Live work:** [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924); completed implementation [I115/#931](https://github.com/guillermomolina/protos/issues/931).

## 1. Scope and exact programs

This record concerns **bare top-level execution with a nested identity definition/call**, **not** the separately established benchmark wrapper `run → identity`.

Protos:

```protos
identity: (value) => { value }
identity(1)
```

GraalPy:

```python
def identity(value):
    return value

identity(1)
```

Two independent compiled guest roots are measured for each product: (1) the bare top-level module and (2) the real one-argument `identity` Closure/function. Their node counts are added only as a **descriptive sum of independent compilations**, never as one graph.

## 2. Measured values and provenance

| Metric | Protos 0.3.386 JVM | GraalPy | Protos minus GraalPy | Provenance / comparability |
| --- | ---: | ---: | ---: | --- |
| Bytecode DSL FULL module | 11 | 12 | -1 | Protos fresh I112 census; GraalPy prior reference |
| Bytecode DSL FULL identity | 5 | 5 | 0 | Protos fresh I112 census; GraalPy prior reference |
| **FULL descriptive sum** | **16** | **17** | **-1** | Different source compilers; DSL metric, not Graal IR |
| Tier 1 module IR | 379 | 469 | -90 | Protos fresh JVM 25.4.4.1.1 vs GraalPy **historical Native Image 25.3.4.1** |
| Tier 1 identity IR | 106 | 110 | -4 | Same Tier-1 cross-runtime limitation |
| **Tier 1 descriptive sum** | **485** | **579** | **-94 (-16.2%)** | Not controlled JVM-to-JVM; not runtime speedup |
| Tier 2 bare module IR | 264 | 286 | -22 | **Fresh** JVM 25.4.4.1.1, independent module roots, differing compilation admission |
| Tier 2 isolated identity IR | 49 | 51 | -2 | **Fresh** JVM 25.4.4.1.1, independent identity roots |
| **Tier 2 descriptive sum** | **313** | **337** | **-24 (-7.1%)** | Two separately compiled roots on each side, **not** entire runtime execution |

**`IR X/Y`**: X is Graal's `After TruffleTier`-stage IR count as emitted by `[engine] opt done`; Y is compiler-final IR. These IR counts have **not** been independently extracted/validated from the BGV phase by this documentation agent; the maintainer supplied successful compilation logs and BGV file listings. Never call an `IR` count a Bytecode DSL instruction count.

### Exact successful compilation lines, 2026-10-20

| Guest root | Tier | IR Truffle/final | CodeSize bytes | Inlining | CompId | Evidence |
| --- | ---: | --- | ---: | --- | ---: | --- |
| Protos `TOP_LEVEL` | 1 | 379/913 | 5302 | 0Y 1N | 3471 | `PERF042_RUN_COMPILED_TIER=1`, `CAPTURE=READY`, RC=0 |
| Protos `identity` | 1 | 106/125 | 487 | 0Y 0N | 3464 | `PERF042_TIER1_COMPILED_TIER=1`, `CAPTURE=READY`, RC=0 |
| Protos `TOP_LEVEL` | 2 | 264/621 | 4248 | 1Y 0N | 3485 | `PERF042_RUN_COMPILED_TIER=2`, `CAPTURE=READY`, RC=0 |
| Protos `identity` | 2 | 49/61 | 278 | 0Y 0N | 3465 | `PERF042_TIER2_COMPILED_TIER=2`, `CAPTURE=READY`, RC=0 |
| GraalPy `<bytecode <module>>` | 2 | 286/779 | 3945 | 1Y 0N | 6234 | `perf042-bare-identity-program.py`, `GRAALPY_module_RC=0`, result 1 |
| GraalPy separately compiled `identity` | 2 | 51/64 | 287 | 0Y 0N | 5812 | `perf042-identity-source.py`, `GRAALPY_identity_RC=0`, result 1 |

In the GraalPy **full bare module** run, its nested `identity` was **also** compiled as a separate `<bytecode identity>` target: CompId **6235**, Tier 2 `IR 52/89`, CodeSize **382 bytes**. This is **not** the 51-node isolated capture in the main table; do not exchange them silently.

GraalPy additionally compiled **`TopLevelExceptionHandler`**, CompId **6224**, Tier 2 `IR 637/957`, CodeSize **5814 bytes**, with `Inlined 2Y 0N`. It is a distinct compilation not included in the 337-node two-root descriptive sum. Thus 264-vs-286 does **not** establish equality of complete root-entry/exception-handling infrastructure.

### Artifacts and exact environment

- Protos capture root: `/workspaces/protos/target/perf042-i115e-386/20261020T060505/` with independent `module-tier1`, `identity-tier1`, `module-tier2`, `identity-tier2` folders, each containing `maven.log` and one target-labelled BGV. User-reported SHA-256:
  - Tier 1 module: `2a299b4bc417b6c2699be30c63b230b79fd09e0efcfb1927676f4b929e2aef9d`.
  - Tier 1 identity: `3ce635c62bc356c18397d31d8fb6b03f6cc248b07981a2733378f19e2f085c92`.
  - Tier 2 module: `4a5b3ebe83597d8a42e196192a05e3cdc6ac14b9781fec54ea09381947449f30`.
  - Tier 2 identity: `2fb3900834419f62b1f028e7feec573974edf987d3ca193d28b9968ff1c65c68`.
- Protos FULL census: `ProtosI112TerminalCallBudgetTest` successful, `I112_ROOT_COUNT=2`, FULL **11+5=16**, `I112_EXTRA_GROUP_ROOTS=0`. Protos Tier compilation tests: `ProtosPerf042IdentityBuilderOracleTest#captureCompleteRunRootAtRequestedTier` and `#captureIdentityRootAtRequestedTier`; explicit `compile(false)`/ `compile(true)`.
- GraalPy capture root: `/workspaces/protos-benchmarks/.work/perf042-graalpy-bare-20261020T064542/` (under the harness commit above); two folders `module/` and `identity/`, each with `run.log` and BGV dumps; total BGV counts **47** and **45**, including other GraalPy runtime/internal targets. No user-supplied per-file SHA-256 for the GraalPy selected BGVs was captured in this handoff.
- GraalPy temporary Java driver used `Engine.newBuilder("python")`, `MultiTier=true`, `BackgroundCompilation=false`, `TraceCompilation=true`, `TraceInlining=true`, `DynamicCompilationThresholds=false`, first/last-tier thresholds **32/96**. It warmed via **2048** `context.eval(cached Source)` calls for module and **2048** `identity.execute(1)` calls for isolated function; both returned **1**. The requested tier was confirmed by individual `opt done |Tier 2|` lines.
- The first attempt to resolve peers through the configured Nexus failed due to a Java **PKIX certificate trust** error; subsequent Maven dependency resolution through a **temporary Maven Central settings mirror** succeeded (`GRAALPY_DEPENDENCIES_RC=0`), and temporary driver compilation succeeded (`DRIVER_COMPILE_RC=0`). No TLS-verification bypass was used.

### Methodological boundaries

The Protos Tier-2 tests **force** `OptimizedCallTarget.compile(true)` on CACHED roots; the GraalPy peer reached Tier 2 through **natural repeated execution** with explicitly lowered compilation thresholds. This is a successful **matched program-shape and GraalVM generation comparison**, **not** identical warmup/admission, compilation policy, complete executable root graph, or quantitative performance evidence. The earlier historical Tier 1 GraalPy figures additionally differ by Native Image vs JVM and release. Comparisons should be annotated accordingly.

## 3. Owner acceptance and next scenario

On **2026-10-20**, after reviewing the Protos and GraalPy measurements, the maintainer explicitly stated **the bare TOP_LEVEL → identity test is satisfactory and requested moving to `run → identity`**. This accepts the **current case as a sufficiently measured checkpoint**; it does not close PERF042's overall performance goal, prove complete physical/runtime parity, approve a new implementation, ratify an architectural design, or remove any language invariant.

**Next PERF042 capture:** `run → identity` should be measured as its **own** program shape: a genuine `run` Closure/function creates and calls `identity(1)`. On GraalPy the already-existing benchmark shape is:

```python
def run():
    def identity(value):
        return value
    return identity(1)

truffleRun = run
```

Use the analogous **existing canonical Protos benchmark** from `guillermomolina/protos-benchmarks/truffle/workloads/primitive-closure-call/primitive-closure-call.protos`, not a rewritten TOP_LEVEL program or a hand-created call. Capture separately `run`, any remaining non-inlined `identity`, and framework/bootstrap targets that are materially necessary; report inlining and the one-root vs multi-root budget explicitly. Use **equivalent** GraalVM version, JVM/Native choice, guest program shape, method of Tier-2 admission and `After TruffleTier` graph extraction. No assumption that old GraalPy `run` **50** plus `identity` **52** represents bare TOP_LEVEL; that historical reference belongs to a different experiment.

**Disposition:** Keep [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) **open** for `run → identity` next. I115/#931 stays **closed**; no new I-family implementation is authorized from this measurement. Preserve PLAT066-C/#930 and PLAT066/#924 as separate open decision/research owners until their own completion gates are explicitly met. No production changes, specification changes or new tests are requested in this accepted evidence checkpoint.

## 4. Reference

- Prior product integration and test-entry-point handoff: [I115-E / 0.3.386](../I115/I115_E_386_MAIN_INTEGRATION_AND_METRICS_HANDOFF_2026_10_20.md).
- Earlier two-root baseline and historical GraalPy Tier 1 evidence: [PERF042 / I115-E1](PERF042_I115_E1_TWO_ROOT_TIER1_IR_COMPARISON_2026_10_20.md).
- Current issue state, scheduling, and any subsequent design decisions remain authoritative in `guillermomolina/protos`; this immutable record is evidence, not an alternate work-status ledger.
