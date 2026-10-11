# I092 — Native Boolean fallback boundary: local compiled-graph A/B and ablation

**Date:** 2026-10-11.  
**Owner:** [I092 / guillermomolina/protos#870](https://github.com/guillermomolina/protos/issues/870).  
**Published product revision:** [`d8692398cc8fa9dc1e968bf073a86d78159bcdb3`](https://github.com/guillermomolina/protos/commit/d8692398cc8fa9dc1e968bf073a86d78159bcdb3), `0.3.333-SNAPSHOT`.  
**Controlled preceding HEAD:** [`1c414e08fefc2378b828bba7be7d263ea32e10ce`](https://github.com/guillermomolina/protos/commit/1c414e08fefc2378b828bba7be7d263ea32e10ce).  
**Status:** Published implementation; maintainer reports `git diff --check` clean and all local tests PASS. Local JVM compilation outputs and semantic-transition checks supplied by the maintainer. This record neither certifies unobserved raw test logs nor claims completion of the separate benchmark-harness acceptance gate.

## Published changes

The published product commit, named `I092: keep the unobserved generic fallback of native Boolean.ifTrue out of the compiled Boolean-only path`, modifies exactly:

- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`: puts `prepareNativeIfTrueCallbackFallback(caller, supplied)` behind `@TruffleBoundary`, keeping the compact B-prime admission compilable and the exact ordinary callback-call preparation available when admission fails.
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`: adds per-site `CanonicalBooleanMissProfile` to four `TryCompactCanonicalBoolean*` operations; the initial noncanonical miss invalidates and records observation.
- `src/test/java/com/guillermomolina/protos/execution/ProtosI092CompactBooleanTest.java`: adds dynamic-receiver canonical/override transition and once-only unselected callback-producer coverage.
- `pom.xml`, `CHANGELOG.md`: published `0.3.333-SNAPSHOT` metadata.

The source record is confirmed by the exact published commit; no independent local tests or Graal runs were performed by the recording agent.

## Local diagnostic: dynamic Boolean receiver

The maintainer executed a standalone diagnostic directly from the Protos packaged JVM, **not** via `guillermomolina/protos-benchmarks`:

```protos
run: (a) => {
    a.ifTrue(() => { 1 })
}
n: 0
(() => { n < 400 }).whileTrue(() => {
    run(true)
    run(false)
    n = n + 1
})
print(n)
```

The diagnostic identified the source-specific Closure root `protos-root:28ac5c46d3c578da` and applied `CompileOnly`, `CompileImmediately=true`, synchronous Truffle compilation, `TraceCompilation=true`, `TraceMethodExpansion=truffleTier`, and Graal BGV dumping. The candidate yielded **four** Tier-2 `opt done` events and **four** BGV files, along with deoptimization/invalidations. Output was furnished from local paths under `target/i092-local-dynamic-loop/` and `target/i092-head-reference/`; these local raw files were **not** committed to this record.

### Equivalent source / controlled variants

The maintainer constructed a reference from `git archive HEAD` of `1c414e08...`, compiled that reference and the two isolated variants in separate local `target/` directories. The same program, root selector, Java launcher and compilation flags were used. These figures are the **first number of the Graal `TraceCompilation |IR A/B` pair**; they have not been independently re-exported as final `After TruffleTier` BGV histograms.

| Compiled attempt | Baseline HEAD | PROFILE_ONLY | BOUNDARY_ONLY | Full patch |
| --- | ---: | ---: | ---: | ---: |
| 1 | 8454 | 8454 | 5079 | 5078 |
| 2 | 8535 | 8535 | 5160 | 5159 |
| 3 | 8544 | 8544 | 5169 | 5168 |
| 4 | 8608 | 8608 | 5233 | 5232 |
| **Mean** | **8535.25** | **8535.25** | **5160.25** | **5159.25** |

- Full patch reduces the displayed IR by **3376 nodes at every corresponding compiled attempt** (approximately 39.6% of the four-attempt baseline mean).
- `BOUNDARY_ONLY` reproduces nearly all of that reduction; `PROFILE_ONLY` reproduces the baseline first-IR figures. The full patch has **one fewer** first-IR node per attempt than `BOUNDARY_ONLY`.
- Full-patch emitted code sizes: **58549, 60783, 60318, 62060** bytes; baseline: **87593, 90248, 89814, 91438**. This is a reduction in emitted code size, **not** a measured latency or allocation rate.
- The same pattern of `uncommon trap` and `Profiled Argument Types` deoptimizations/invalidations appeared before and after the patch. These traces do not prove the miss profile causes or cures recompilation.

### Expansion-tree attribution

Local `TraceMethodExpansion=truffleTier` reports first-attempt inclusive subtree figures:

| Selected method / subtree | HEAD | Full patch |
| --- | ---: | ---: |
| `TryCompactCanonicalBooleanOne.perform` | 3574 | 1107 |
| `nativeIfTrueCallback` | 3562 | 1098 |
| `prepareInlineLiteralCall` | 2443 | absent from matched expansion |
| `compactCanonicalIfTrue` | 1081 | 1080 |

The measured difference is **principally explained by isolating generic callback-call preparation behind `@TruffleBoundary`**, not by the new per-site miss profile. The expansion rows are nested/inclusive and **must not be summed or equated to surviving final IR node ownership**. The trace search found no matching `PrepareSend` / `PrepareStructuredBooleanCall` expansion rows in either variant; this is **not** a final-BGV control-flow proof that no cold generic dispatch nodes survive.

## Dynamic receiver transition after compilation

The maintainer then warmed the same `run(a)` root with canonical `true` and `false`, invoked a custom receiver with `ifTrue: (callback) => { 99 }`, and returned to canonical receivers.

For **both** `BOUNDARY_ONLY` and **FULL_PATCH**, the result was:

```text
COMPILATIONS_DONE=5
COMPILATIONS_FAILED=0
EXPECTED=['I092_WARM_COMPLETE', '41', 'true', '99', '41', 'true']
OBSERVED=['I092_WARM_COMPLETE', '41', 'true', '99', '41', 'true']
SEMANTICS_PASS=True
```

Both variants reported the same invalidation sequence: `uncommon trap`, two `Profiled Argument Types`, two `validRootAssumption local tags updated`, and a final `uncommon trap`. This is a **bounded JVM transition check**, not proof of every alias/override/continuation/tooling scenario; independent focal and integrated tests were reported PASS by the maintainer.

## Interpretation and limits

1. **Supported:** the published fallback boundary yields a reproducible, substantial reduction in the trace's first IR number for the one dynamic Boolean site tested. The ablation specifically localizes the dominant saving to the boundary. Canonical/custom/canonical behavior survived real compilation.
2. **Not supported:** equating the trace `IR A/B` field with the benchmark harness's **selected `After TruffleTier` final BGV node count**; claiming a 39.6% speedup; claiming Boolean control now compiles as compactly as GraalJS/GraalPy.
3. **Not inferred:** the miss profile is useless, must be removed, or explains the invalidations. The full patch is the published owner-selected state; no removal is authorized by this record.
4. **Distinct historical captures:** `primitive-if-true` on earlier **different sources/revisions** was 5193 (original PERF041) then 2173 (earlier I092) selected `After TruffleTier` nodes; those are **not** directly comparable with the dynamic receiver standalone diagnostic's first IR figures or a new `run: () => { 1 }` test.
5. **Unresolved structural attribution:** callback Closure materialization, `PreparedInlineLiteralCall` / B-prime admission, frame/locals, `TryFinally` and exceptional/deopt `FrameState` may still account for significant cost. No exact partition of final nodes has been verified.

**Disposition:** preserve the published implementation and this evidence. PERF041/#863 is already closed; do not reopen it or manufacture a new formal PERF item. I092 remains open until its existing final graph-acceptance/closure criteria are independently reconciled. A separate lightweight `run: () => { 1 }` source-root vs peer comparison is suitable under existing [PERF032/#831](https://github.com/guillermomolina/protos/issues/831), without changing product or using `protos-benchmarks`; it is **not executed or claimed complete** here.

```text
PRODUCT_REVISION=d8692398cc8fa9dc1e968bf073a86d78159bcdb3
PREPATCH_HEAD=1c414e08fefc2378b828bba7be7d263ea32e10ce
PRODUCT_PUBLICATION=CONFIRMED
MAINTAINER_GIT_DIFF_CHECK=CLEAN_REPORTED
MAINTAINER_LOCAL_TESTS=PASS_REPORTED
LOCAL_DIAGNOSTIC_VARIANTS=4
LOCAL_COMPILED_ATTEMPTS_PER_VARIANT=4
DYNAMIC_TRANSITION_COMPILED_ATTEMPTS_PER_VARIANT=5
DYNAMIC_TRANSITION_SEMANTICS=PASS_REPORTED
RAW_LOCAL_BGV_COMMITTED=NO
SELECTED_AFTER_TRUFFLE_TIER_PEER_PARITY=NOT_MEASURED
SPEEDUP=NOT_MEASURED
I092_COMPLETION=OPEN
```
