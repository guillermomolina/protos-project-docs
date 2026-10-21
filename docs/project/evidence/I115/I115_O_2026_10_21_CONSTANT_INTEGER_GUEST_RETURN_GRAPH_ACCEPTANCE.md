# I115-O / PLAT066-C / PERF041 — certified constant Integer guest return: physical IR acceptance and semantic boundaries

**Date:** 2026-10-21. **Classification:** revision-pinned experimental implementation evidence, not a normative language decision or production-conformance approval.  
**Product:** [guillermomolina/protos@ab632560f19e1b876b9b6c8f2018f5cb86e9eb13](https://github.com/guillermomolina/protos/commit/ab632560f19e1b876b9b6c8f2018f5cb86e9eb13), branch **I115** (experimental). **Baseline:** [04c04e998d1cefd0e7c55ab335ad704607f652b2](https://github.com/guillermomolina/protos/commit/04c04e998d1cefd0e7c55ab335ad704607f652b2), I115-M/N.  
**Owners:** [I115 #931](https://github.com/guillermomolina/protos/issues/931), [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930), [PERF041 #863](https://github.com/guillermomolina/protos/issues/863). All remain **OPEN**.  
**Evidence provenance:** Maintainer-supplied I115-O report dated 2026-10-21, with source publication independently checked against the exact GitHub commit. Test outcomes and new compilation-graph totals are **maintainer-reported**; the new BGV binary files and logs were not independently redecoded in this documentation transaction. No product mainline integration or release is claimed.

## 1. Exact workload and why the next change was needed

The measured object is the independent *run* Closure of the **PERF041V variable-receiver variant**, not the original minimal PERF039 *primitive-if-true* source:

~~~protos
run: () => {
    value: true
    result: null
    value.ifTrue() {
        result = 1
    }
    result
}
~~~

The matched GraalPy 25.4.4.1.1 function is:

~~~python
def run():
    value = True
    result = None
    if value:
        result = 1
    return result

truffleRun = run
~~~

Both return 1 for these inputs. They are **not semantically interchangeable** across ordinary overridable Protos Boolean messages, Closure materialization, Context, Task, ReturnHome, effects, and host interop.

The earlier PERF041V source/graph investigation established **13 Bytecode DSL FULL instructions for both run roots**, but the I115-M/N Protos compiler graph retained a substantial return-carrier projection. The before-change final IR counts were **194 / 103** in Protos Tier 1 / Tier 2, against **127 / 62** in GraalPy. Of the Protos Tier-2 103 final IR nodes, **33 were source-attributed to the return projection** in that prior investigation. Source attribution does **not** equal a guaranteed deletion budget.

## 2. Published source change and admission

The exact product commit was independently fetched from GitHub. Its subject is:

> I115-O: store certified Integer literal returns as prepared guest constants and skip the compact return-carrier projection

The commit changes **three files**:

1. [CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/ab632560f19e1b876b9b6c8f2018f5cb86e9eb13/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java) — only production file; adds *admitExperimentalConstantIntegerReturn* and the local admission/emission state.
2. [ProtosI115OConstantIntegerReturnTest.java](https://github.com/guillermomolina/protos/blob/ab632560f19e1b876b9b6c8f2018f5cb86e9eb13/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI115OConstantIntegerReturnTest.java) — focused regression/representation/structural tests.
3. [ProtosPerf041VariableIfTrueCensusTest.java](https://github.com/guillermomolina/protos/blob/ab632560f19e1b876b9b6c8f2018f5cb86e9eb13/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf041VariableIfTrueCensusTest.java) — adds guest-Integer versus primitive-Long constant census and reports admission.

The lowerer admits the optimization only inside an already-admitted **parameterless compact I115-M/N root** when all these facts hold:

- The terminal expression is a lookup of a **root-owned binding**, resolved by **CanonicalBindingIdentity**, not a benchmark-specific variable name.
- Every assignment/creation targeting that binding is a **statement** storing a signed-64-bit Integer literal, canonical null, or canonical Boolean; at least one Integer literal store occurs.
- There are **no other reads** of that binding, and the result of a store to it is **not consumed**.
- The same scan handles nested admitted inline callback and whileTrue bodies, with consumption checked at the relevant sequence boundary.
- Failed certification leaves the prior compact/ordinary lowering and return handling in place.

For a certified target store, *guestLiteralConstant* constructs a **ProtosIntegerValue** during lowering and the builder emits **load.constant** for the already prepared guest object. The terminal lookup therefore returns a guest Integer without a primitive-to-guest conversion. Only that certified root disables **epilogProjectsReturnCarrier**, skipping **MaterializeReturnCarrier** on its return path. The lowerer's temporary certification state is saved/restored around root emission.

**No new Bytecode DSL operation, runtime cache/map, synchronization, new ABI slot, or global Integer interning was introduced.** The change does not edit ProtosBytecodeRootNode, ProtosNumericValueSupport, ProtosIntegerValue, or ProtosFrameArguments; it does not change the ordinary fallback.

## 3. Bytecode DSL census: unchanged physical instruction count

| PERF041V run census | I115-O |
| --- | ---: |
| Physical roots (module + run) | **2** |
| Module DSL FULL | **11** |
| run DSL FULL | **13** |
| Total DSL FULL | **24** |
| Guest Integer constants in run | **1** |
| Primitive Long Integer constants in run | **0** |
| Certified guest result | **Yes** |
| Added DSL operations | **0** |

The guest Integer constant feeds a store.local to the result binding. **EpilogForReturn remains a Bytecode DSL epilogue**, but the root no longer performs carrier projection. The gain comes from removing physical work within the same 13 FULL instructions, not lowering to fewer instructions.

## 4. Measured compiler IR: before, I115-O, GraalPy

The maintainer reports successful independent Tier 1 (compile(false)) and Tier 2 (compile(true)) BGV captures for the **same run root**. All IR counts below refer to **compiler StructuredGraph nodes**, not AST views, and exclude the module root.

| Compiler phase / metric | Protos I115-M/N | Protos I115-O | GraalPy 25.4.4.1.1 | I115-O saved versus Protos baseline |
| --- | ---: | ---: | ---: | ---: |
| Tier 1 — After TruffleTier | 135 | **104** | 106 | **−31 (−23.0%)** |
| Tier 1 — After low tier | 194 | **123** | 127 | **−71 (−36.6%)** |
| Tier 2 — After TruffleTier | 76 | **47** | 49 | **−29 (−38.2%)** |
| Tier 2 — After low tier | 103 | **59** | 62 | **−44 (−42.7%)** |

**Result:** I115-O has **2–4 fewer compiler IR nodes than GraalPy** in these four matched phase/tier observations (104 vs 106, 123 vs 127, 47 vs 49, 59 vs 62). This is **IR-count convergence**, not proven equivalent language semantics or a measured execution-time speedup.

The new graphs reportedly retain the **ProtosIntegerValue constant** and do not retain a **NewInstanceNode** for materializing that Integer; the reduction persists through low-tier lowering. The saved 44 final Tier-2 nodes are **not** asserted to correspond one-for-one to the prior 33 source-attributed return-projection nodes.

**Outstanding measurement:** **CodeSize for I115-O was not supplied**. Earlier CodeSize values and a faster execution time must not be inferred from IR node counts. BGV checksums, compile IDs, raw phase histograms, and matched timing data were not supplied with this I115-O report and remain unverified here. The previous baseline's exact graph provenance is separately described in the maintainer's PERF041V investigation.

## 5. Regression evidence and limits

**Maintainer-reported validation:** **41 tests counted, 0 failures, 0 errors, 1 skipped** (optional Graal capture). The PERF041V structural census reports **PASS**. New and existing focused scenarios cover:

- Signed-64 literal variants, repeat writes, alternative branches, Boolean/null stores, and nested inline callbacks/whileTrue.
- Negative admission for computed returns and arithmetic consumers, including preservation of the old return projection where required.
- Calls from another Closure, two Context instances, and host interop.
- Ordinary equality and semantic identity (including repeated Integer results), UNCACHED-to-CACHED transition, and reparse stability.
- The exact certified/unadmitted **epilogProjectsReturnCarrier** distinction.

The maintainer also reports **BUILD SUCCESS** for Tier 1 and Tier 2 capture runs, with a BGV for the selected run Closure in each. Source/test structure was independently checked in the published commit; the numerical JUnit outcome and BGV run results were not reproduced by the documentation agent. An integrated full-suite PASS beyond the supplied 41-test report is **not** independently claimed.

## 6. New physical departures versus inherited semantic debt

### 6.1 I115-O itself: two changed physical choices, no newly confirmed language-contract violation

**A — Early guest Integer materialization and physical reuse.** A known signed-64 Integer literal assigned to the certified binding becomes a guest ProtosIntegerValue constant during lowering, potentially shared by repeated invocations. The Protos numeric domain is immutable and uses **semantic value identity**; physical allocation identity is not normative numeric identity. This makes reuse **plausibly conforming**, subject to the concrete guest/host, Context and reflection contracts. The reported equality/identity cases pass. It is a departure from the previous *late carrier projection strategy*, **not by itself a violation** of the language specification.

**B — Omitted return-carrier projection.** Setting **epilogProjectsReturnCarrier = false** for a proven guest-result root avoids the generic MaterializeReturnCarrier return branch. The result remains a guest Integer; there is **no source-level change** to the caller/callee argument ABI or the global return-home rules. This is an internal path specialization, not by itself a confirmed Protos semantic violation.

**New confirmed violations identified by this evidence:** **none**. That classification is **not a proof of universal conformance**.

**Conformance to investigate before production ratification:** Context-local semantic delegation and cross-Context transfer, primitive-carrier-accepting callers versus ordinary callers, identityHashOf/IdentityMap, SmallInteger prototype recognition and parent(), interop/reflection, ReturnHome/Task/continuation behavior, and cases outside the certified subset. Two-Context and caller tests reduce but do not eliminate these concerns.

### 6.2 Inherited experimental deviations: I115-I through I115-N, not caused by O

I115-O *depends* on the earlier compact-root experiment. The following **pre-existing** deviations must not be misattributed to this slice:

- **Guest lexical authority and binding context:** compact Bytecode locals can bypass the general guest Context/slot authority and reflection behavior.
- **Callback Closure identity/activation:** admitted inline callbacks may bypass observable materialization, lexical activation, ReturnHome and other general invocation guarantees.
- **Overridable dynamic method selection:** admitted Boolean/whileTrue and numeric sends may bypass ordinary lookup/overrides, error timing, and method selection.
- **Unbounded numeric semantics:** the inherited signed-64 arithmetic lane does not generally implement required overflow promotion to arbitrary-precision Integer.
- **Error/continuation boundaries:** native Boolean admission and compact control are not universally proven to preserve guest Error, Task, cancellation, or suspension behavior.

See the dedicated [I115-I/N mechanism and semantic-deviation record](I115_IN_2026_10_21_COMPACT_WHILE_BOOLEAN_DSL_IR_AND_SEMANTIC_DEVIATIONS.md). Prior I115-H departures are **outside this bounded I115-O attribution**. Passing I115-O tests does not discharge the pre-existing deviations.

**Normative authorities not changed:** Protos spec/semantics/VALUES_AND_COLLECTIONS.md (Number value identity, numeric behavior, Boolean sends), CALLABLES.md (Closure calls and return homes), EXECUTION_AND_CONTROL.md (contexts and control), and OBJECT_MODEL.md (ordinary lookup/slots). No specification, public protocol, or ratified PLAT design is amended by this evidence record.

## 7. Disposition and next acceptance work

**I115-O implementation checkpoint:** published to the experimental **I115** branch and reported functionally validated. This record documents a real source change and substantial **measured compiler-IR reduction** while retaining 13 run DSL FULL instructions, not a proven latency/throughput gain.

**Still open:** I115/#931, PLAT066-C/#930, and PERF041/#863. No normative conformance or further design ratification is implied.

The next bounded verification gates are:

1. Complete matched **CodeSize** measurement for both tiers and preserve exact compile IDs, phase selection, raw BGVs and checksums.
2. Finish targeted **semantic conformance checks** for reusable guest constants across Context, reflection, host interop, identity and carrier-accepting callers.
3. Evaluate a subsequent source-backed slice only if the remaining IR/CodeSize or other observable costs justify it, without using inherited I115-I/N semantic exceptions as automatic approval for new ones.

**Evidence status:** PRODUCT_COMMIT=VERIFIED; FOCAL_TESTS=MAINTAINER_REPORTED; NEW_BGV_COUNTS=MAINTAINER_REPORTED; CODESIZE=PENDING; THROUGHPUT=NOT_MEASURED; NEW_CONFIRMED_SEMANTIC_VIOLATIONS=NONE_IN_REPORTED_SCOPE; FULL_PRODUCTION_CONFORMANCE=NOT_ESTABLISHED.
