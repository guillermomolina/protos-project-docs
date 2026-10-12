# PERF040-A/B/C — Published canonical primitive numeric arithmetic

**Date:** 2026-10-12. **Owner:** [PERF040/#862](https://github.com/guillermomolina/protos/issues/862). **Repository:** `guillermomolina/protos`. **Published product commit:** [`18f71b2a664c56ac808e2f508b6369984de02280`](https://github.com/guillermomolina/protos/commit/18f71b2a664c56ac808e2f508b6369984de02280), `0.3.342-SNAPSHOT`. **Before:** `1fc6899bbc6e18359bd84d2357bd63e78523b53c`, `0.3.341-SNAPSHOT`.

This is non-normative implementation and validation evidence. It does not redefine D013 or the existing D196/D197 binary64, integer, Fraction, or numeric-family semantics. It does not assert a benchmark speedup or compiled-code allocation result.

## Published implementation

The product commit changes seven files (+414/-10): `pom.xml`, `CHANGELOG.md`, `src/main/java/com/guillermomolina/protos/execution/ProtosPrimitiveNumericSendFastPathsBase.java`, `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`, and the three `ProtosI091Primitive{CarrierPropagation,FloatSend,IntegerSend}Test.java` test files.

- **PERF040-A/B:** Separate `doLL` and overflow-generalized `doLLOvf` specializations for the canonical Integer `+`, `-`, and `*` operations. Ordinary signed-64 arithmetic stays primitive; exact overflow uses the selected numeric normalization boundary. Division retains the previously governed exact-quotient/fallback behavior.
- **PERF040-C:** Independently specialize Float-receiver `double/double` and `double/long` arithmetic for `+`, `-`, `*`, `/`; reuse the existing Integer-receiver `long/double` path. Native `double` operators preserve binary64 signed zero, NaN, infinities, and division-by-zero behavior without introducing Float objects for the calculation.
- **Admission and observation:** Preserve exact canonical Closure, Prelude, entered Context, method-home lookup, and stability Assumption selection. Non-canonical selections, large operands, Fraction, incompatible arguments, and observational guest-value materialization remain subject to existing generic/semantic paths. No new normative specification revision was published by this change.

## Human-reported validation

The maintainer reports:

```text
PRODUCT_COMMIT=18f71b2a664c56ac808e2f508b6369984de02280
PRODUCT_VERSION=0.3.342-SNAPSHOT
PRODUCT_PUSH_MAIN=PASS
FOCAL_INTEGER_FLOAT_TESTS=PASS
POLYMORPHIC_TEST_NEWLINE_FIX=PASS
INTEGRATED_MAKE_TEST=PASS
STAGED_GIT_DIFF_CHECK=PASS
TESTS_AFTER_POM_CHANGE=NO
```

The first focused Maven execution had one lexical-error test fixture (`\\n` instead of a newline) while the other 18 tests passed. The fixture was corrected and that exact failing test subsequently passed. The maintainer then reported the complete integrated `make test` PASS for the final substantive code. Only `pom.xml` and `CHANGELOG.md` were updated afterward, immediately before commit/push; the previously passing tests were not rerun after metadata-only finalization. These are maintainer reports, not assistant-executed tests or independent CI attestations.

## Performance gate still open

**No new PERF040 benchmark, JFR allocation measurement, machine-code inspection, or post-change graph capture is reported.** Functional validation proves neither steady-state improvement nor absence of allocation in a particular compiled graph. The new Integer arithmetic node's `doLL`/`doLLOvf` transitions, including fast arithmetic after one overflow, need direct performance evidence.

The current [`guillermomolina/protos-benchmarks`](https://github.com/guillermomolina/protos-benchmarks) revision-independent driver is `truffle/measure_protos.py`. Its available cases include `primitive-integer-add` (one small sum per `run`) and `integer-loop` (repeated small sums); both can measure the published product commit without changing the measurement scripts. Compare on the same host, same clean harness, same `canonical` surface and unchanged reference policy (`primitive-integer-add`: warmup 50, steady 10, sample calls 10,000, steady-only admission) against a clean product checkout at the immediate parent `1fc6899bbc6e18359bd84d2357bd63e78523b53c`. Exact harness SHA, CPU, timing-policy identity and per-run validity are mandatory.

These existing cases do **not** separately isolate subtraction, multiplication, overflow-generalized arithmetic, or numeric type alternation at one hot call site. Further targeted benchmark workloads, if justified after the focused baseline, are a separate benchmark-harness task and cannot be claimed from the existing `primitive-integer-add` result. GraalJS/GraalPy baselines need not be rerun merely to measure this Protos-only delta.

**Scope state:** PERF040-A/B/C implementation published and human-validated; PERF040 overall remains open pending revision-pinned benchmark/graph acceptance. The closed I091 implementation scope and the independently open I090/D198 Complex API gate must not be conflated with PERF040's measured-performance acceptance.
