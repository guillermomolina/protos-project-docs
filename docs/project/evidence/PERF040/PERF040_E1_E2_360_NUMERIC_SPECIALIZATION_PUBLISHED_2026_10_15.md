# PERF040-E1/E2 — Published certified numerical operations and BigInteger lanes

**Recorded:** 2026-10-15. **Formal owner:** [PERF040 #862](https://github.com/guillermomolina/protos/issues/862), OPEN/READY for remaining acceptance. **Related, independently open:** [PERF046 #890](https://github.com/guillermomolina/protos/issues/890) (numeric equality/comparison performance), [I101 #892](https://github.com/guillermomolina/protos/issues/892) (minimal canonical NORMAL lowering). **Product revision:** [`0982d1149e45c68deee13623afea6890b49f166a`](https://github.com/guillermomolina/protos/commit/0982d1149e45c68deee13623afea6890b49f166a), published on `main`, `0.3.360-SNAPSHOT`. **Immediate parent:** `cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d` (I103-A). **Published message:** `PERF040: certified per-selector numeric operations with doLL/doLLOvf and BigInteger lanes`.

**Evidence class:** published implementation, source-level architectural inspection and maintainer-reported local functional/generator validation. **Not:** an independently rerun test suite, final-tier Graal graph analysis, measured speedup, allocation proof, benchmark parity with GraalPy, or closure of PERF040 as a whole. Historical [PERF040-E GraalPy source research and implementation handoff](PERF040_E_GRAALPY_COMPLETE_NUMERIC_FALLBACK_RECONCILIATION_HANDOFF_2026_10_15.md) remains the pre-publication research snapshot, not the current product state.

## Exact publication

GitHub commit inspection confirms 24 changed files, **3,359 additions / 604 deletions**, including `pom.xml`, `CHANGELOG.md`, eight new `ProtosNumeric*Node` operations, `ProtosNumericCoreAdmission`, the shared numeric support, new PERF040 regression tests, I101/PERF025 compatibility tests and the generated-bytecode BCI PE baseline.

The checked-in POM and CHANGELOG both identify **`0.3.360-SNAPSHOT`** and explicitly describe PERF040-E1/E2.

### Specialization structure

- `CanonicalToBytecodeLowerer` emits separate Bytecode DSL `@OperationProxy` operations for `+`, `-`, `*`, `/`, `<`, `<=`, `>`, `>=`; the selector is selected during lowering, not string-compared by one universal operation at run time.
- The former `TryPrimitiveNumericSendOne`, with its generic `replaces` over numeric positive lanes, is removed. Every new operation declares certified positive specializations and a **strict negatively guarded `ordinaryMiss`**, which returns `MISS` to the preserved D013 ordinary send. A nonnumeric or unauthenticated input cannot permanently evict an admitted numeric specialization.
- For `+`, `-`, `*`, distinct `doLL(long,long) -> long` and `doIntegers(...)->long` paths execute `Math.addExact`, `subtractExact` or `multiplyExact` directly. `rewriteOn=ArithmeticException` transitions only into their own `doLLOvf` / `doIntegersOvf` exact overflow generalizations. Those may return small `long` results again; overflow no longer requires generic D013 dispatch. `replaces` here is deliberately limited to the same numeric domain.
- Authorized Java `Long` and `Double` carriers remain separate from genuine immutable Protos `SmallInteger` and `Float` representations. A carrier in a position not authorized by the lowerer never acquires numeric admission. The numeric families continue to share exact operand-order and fold-candidate guards.
- `ProtosNumericCoreAdmission` checks the frozen canonical Core graph, including actual Closure identity, owner/home and prototype relationships. A caller's `ProtosPrelude` constructs and memoizes its certificate **on first specialized need**, not during Prelude construction. The field is `@CompilationFinal` and `volatile`; first access transfers to interpreter/invalidation and a `@TruffleBoundary` synchronized publication path; subsequent reads do not retake the monitor. Negative certification is memoized and separate Preludes never share state. Whether all compiled caller/Prelude/certificate loads disappear still requires graph evidence.
- Genuine Prelude-owned `ProtosLargeIntegerValue` operands have explicit BigInteger–SmallInteger and BigInteger–BigInteger lanes for the eight selectors. Large arithmetic, D197 exact division and normalization reuse `ProtosNumericValueSupport`, not a second BigInteger engine. Mixed large/Float obeys D196 binary64 rounding in both orders, and ordering uses the existing exact comparisons, including NaN as unordered. Non-owned lookalikes, alternative Core, unauthorized carriers, error domains such as a zero divisor, and unsupported Fraction/Complex inputs stay on ordinary dispatch.
- Minimal I101 folded arithmetic/comparison roots and NORMAL/DEBUG semantics remain covered by updated Builder-oracle and PERF025 tests. `lookupGuardedFloat` was removed from its now-unused site; no new language semantics or optimizer/IR are claimed.

### Generation topology reconciliation

The maintainer reviewed the actual generated Java against **the original product HEAD baseline**, rather than accepting a previously edited local candidate. Reported classification:

| Item | Count | Explanation |
| --- | ---: | --- |
| New topology entries | 49 | 31 numeric tail-call instructions and 18 dispatcher-overflow regrouping entries |
| Removed topology entries | 21 | 3 obsolete `TRY_PRIMITIVE_NUMERIC_SEND_ONE*` entries and 18 corresponding regrouped entries |
| Proven transitions | 1,107 | Static guard report |
| Dynamic transitions | 4 | Require the Makefile's dynamic compilation check |
| RISK | 0 | Reported guard result |
| UNKNOWN | 0 | Reported guard result |

Add/Subtract/Multiply include generated `$LL`, `$LL$UNBOXED`, `$LL_OVF`, `$INTEGERS` and `$INTEGERS$UNBOXED` variants, plus rich/Float combinations. This is generated-instruction evidence of primitive specialization, **not** a proof that final-tier machine code has no boxing.

The maintainer traced the three unrelated-looking handlers `PREPARE_CLOSURE_CALL_`, `PREPARE_CLOSURE_CALL_ARGUMENTS_`, and `APPEND_INLINE_SPREAD_SUPPLIED_ARGUMENT_` to the generated `continueAt_1` overflow split; their handler topology remains represented. The older Float-only generated names were replaced by merged BigInteger/Float variants. No BCI-topology difference was attributed to I102. The accepted baseline is the generated candidate, not an unverified manual projection.

## Validation and attribution

**Human-reported, after the E1/E2 source changes:**

- `188` focal tests PASS (reported in the coordination conversation; do not infer total suite count).
- Generator BCI guard classification reported **1,107 proven / 4 dynamic / zero RISK / zero UNKNOWN**, and the candidate was reviewed before adoption. An exact final `make check-generated-bytecode-bci-pe` PASS log, including the separate dynamic compilation check, was not supplied in the coordination messages.
- Complete local `make test` PASS.
- `git diff --check` clean.
- Product commit and push completed; the commit and matching metadata were independently verified through GitHub.

The assistant **did not execute** the product build, tests or benchmarks. The maintainer's reported PASS is retained as human validation, not recast as independent CI or Graal compilation-graph evidence. Changes to `pom.xml` and `CHANGELOG.md` belong to the final human publication transaction; no post-version-bump testing is claimed.

## Scope correction — correctness of Truffle specialization, not promised speedup

**Maintainer clarification (2026-10-15):** the primary goal of this E1/E2 implementation was to make Protos' Truffle DSL specialization structure and transitions **correct by the DSL's own contracts**, using GraalPy as an upstream architectural reference. The former universal negative/generic specialization with broad `replaces` could evict viable positive numeric specializations; disjoint admission prevents this, and narrow `doLL` / `doLLOvf` rewrites generalize only overflow within the same numeric domain. Owned BigInteger lanes and the memoized Prelude certificate preserve exact canonical method-selection authority without conflating ordinary lookalikes with builtins.

This is **compiler/specialization correctness**, not necessarily evidence of previously incorrect guest-language results: the old ordinary D013 fallback could still produce the correct observable answer while failing to retain the intended specialized state. The correctness fix was not commissioned on an assumption that it would make programs faster. It may or may not change steady-state performance; no speedup, regression, or parity with GraalPy is inferred.

**Acceptance distinction:** the maintainer's green focal/integrated tests, generated-topology audit and published source commit close the **implementation and functional-validation scope of E1 and its BigInteger extension**. Any new Tier-1/Tier-2 graph or timing work is an independently scoped **performance investigation under the broader PERF040 issue**, not a retroactive gate preventing acceptance of these published correctness changes. Fraction/Complex remains separately pending within the broader previously authorized numeric-family plan.

## Remaining PERF040 acceptance

**PERF040 #862 stays OPEN/READY.** E1 and the BigInteger part of E2 are published; no need to recreate these as new Issues. Still not established: pinned Tier-1/Tier-2 Graal IR, whether a monomorphic caller's Prelude/certificate reads fold away, non-boxing and stable arithmetic after overflow in final compiled code, measured timing/allocations versus a clean controlled baseline, and the separately constrained Fraction/Complex accelerated-admission/execution semantics. Fraction/Complex standard source-backed Closures must retain observability, instrumentation, control/suspension and Error behavior; they were intentionally **not** bypassed by these BigInteger lanes.

**PERF046 #890** independently owns cross-family `==`/`!=` and broader comparison performance acceptance; this publication only adds specialized canonical `<`, `<=`, `>`, `>=` lanes and does not close PERF046. **I101 #892** remains independent for its pending cross-profile/full acceptance; PERF040 preserved its Builder-oracle tests rather than claiming I101 closure.

No new benchmark run, latency number, speedup, GraalPy parity or machine-code allocation result is asserted by this record.
