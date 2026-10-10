# I089-A — D196 B1 operand-first mixed Integer/Float implementation acceptance

**Date:** 2026-10-10.  
**Issue:** [I089/#867](https://github.com/guillermomolina/protos/issues/867).  
**Design authority:** [D196/#864](https://github.com/guillermomolina/protos/issues/864), owner-approved B1; [ratified decision](../../decisions/language/D196_MIXED_INTEGER_FLOAT_ARITHMETIC_PROMOTION.md).  
**PROTOS_REVISION:** `eba6250f96aa6873829f7011a3a1aee7b6969887` ([product commit](https://github.com/guillermomolina/protos/commit/eba6250f96aa6873829f7011a3a1aee7b6969887)).  
**Implementation version:** `0.3.325-SNAPSHOT`.  
**Normative specification revision:** `0.1.452` (`spec/PROTOS_SPEC_CHANGELOG.md`).

## Publication and scope

The human executor committed and pushed `I089-A: implement D196 mixed numeric promotion` to `guillermomolina/protos` on `main`. GitHub commit inspection confirmed the exact revision and all 12 changed paths, with 252 insertions and 71 deletions as reported by the human.

For standard `+`, `-`, `*`, and `/`, one Integer and one Float now use **operand-first binary64 conversion** of the Integer, followed by the IEEE arithmetic operation in original operand order. Both operand orders return Float. This is not exact mixed arithmetic followed by one final rounding. Implementation adapts the Integer/Float standard protocols and shares the existing correctly rounded Integer-to-binary64 conversion semantics, including the small-`long` fast path.

Preserved by design: current exact unbounded Integer/Integer `+`, `-`, `*`; current exact-quotient-rounding Float result for Integer/Integer `/`; Integer-only `div`, `mod`, `%`; numeric equality/ordering/hash and semantic identity; ordinary overridable lookup/guarded-send fallback. `2^1024 * 0.0` yields NaN because the Integer operand converts to infinity first.

The product commit changed:
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumericConversionProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardFloatProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIntegerProtocol.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardIntegerArithmeticTest.java`
- `protos/tests/conformance/float/arithmetic.protos`
- `protos/tests/conformance/float/domain-and-receiver-errors.protos`
- `protos/tests/conformance/integer/arithmetic-and-unary.protos`
- `protos/tests/conformance/integer/floating-division-errors.protos`
- `spec/semantics/VALUES_AND_COLLECTIONS.md`
- `spec/PROTOS_SPEC_CHANGELOG.md`
- `pom.xml`
- `CHANGELOG.md`

## Human-executor validation evidence

The maintainer reported, for the unversioned substantive candidate:
1. Focal Java numeric tests: **PASS**.
2. Selected original Protos conformance: **60/60 PASS**.
3. Additional integer/float regression selections: **26/26 PASS** (seven newly added boundary cases).
4. Integrated `make test`: **PASS**.
5. `git diff --check`: **CLEAN** after finalization.
6. Staged paths and branch synchronization: **VERIFIED**; `main -> main` push succeeded.

Tests were run by the human, not replayed by this evidence author. As required by the maintainer's version policy, tests were **not** rerun after `pom.xml` and root `CHANGELOG.md` changed. No separate Native Image, benchmark, or compiler graph measurement is claimed.

## Boundary and coordinated successors

**I089 implementation and normative publication are complete at the pinned product SHA.** This does not implement [I090/#868](https://github.com/guillermomolina/protos/issues/868) (D197 visible BigInteger/Fraction/Complex, exact-division semantics) or [I091/#869](https://github.com/guillermomolina/protos/issues/869) (PLAT056 Candidate C physical numeric carriers/Java BigInteger containment). I091's next independent I091-A work is a classified dependency census and Candidate C feasibility gate. I090 remains blocked on unresolved public `Integer.recognizes` / exact-integral-domain API semantics and I091 feasibility; no approval is inferred from these passing tests.

This is product acceptance evidence, not a new semantic/design decision. The immutable product SHA above is the authority for the implementation; this evidence file's exact publication commit is the accompanying PROJECT_RECORD_REVISION, to be recorded after successful repository write.
