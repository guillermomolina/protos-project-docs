# D230-C1 — Comparative research, adversarial cases and approval evidence (2026-11-01)

> **Historical, non-normative evidence.** The owner approved C1 in the active conversation with the exact words **“Apruebo C1.”** This is evidence for [D230/#1011](https://github.com/guillermomolina/protos/issues/1011), not executable conformance or proof of a speedup. Ratified non-normative decision: [D230-C1](../../decisions/language/D230_CORE_INTEGER_BITWISE_OPERATIONS_C1.md).

**Exact Protos source review:** [`73bd9fe913421ff493146b546869f9a1f0d2b34e`](https://github.com/guillermomolina/protos/commit/73bd9fe913421ff493146b546869f9a1f0d2b34e). **Research conducted:** 2026-11-01. **No Protos source/spec edits, tests or benchmarks were performed in this decision research.**

## Problem and inspected evidence

- `spec/semantics/VALUES_AND_COLLECTIONS.md` §Numeric Model fixes exact arbitrary-precision mathematical Integer, signed-64 SmallInteger/other BigInteger canonical families, strict original semantic-family recognition, and explicitly *does not yet standardize bitwise or wrapping protocols*. Its `div` rounds toward zero and `mod` has the dividend's sign; copying these to negative arithmetic right shift is incorrect.
- `spec/PROTOS_GRAMMAR.md` §§21 and Symbolic Operator Lexing retain a fixed symbolic surface. `^` is non-local return. No spare `&`, `|`, `~`, `<<`, `>>` operator budget exists without independent language design.
- `spec/semantics/OBJECT_MODEL.md` original-receiver-family validation forbids accepting an arbitrary object merely delegating to Integer; user selectors may shadow. `spec/semantics/ERRORS.md` specifies generic Error, with no special required numeric-error prototype.
- `protos/lib/core/Integer.protos`: source-backed `negated` and `_coreIntegerPercent`; `src/main/java/com/guillermomolina/protos/execution/protocols/numeric/ProtosStandardIntegerProtocol.java`: canonical install, exact selector/home/Closure provenance and validated native execution. `ProtosNumericValueSupport.java` normalizes semantic Integer family. Truffle specializations may not bypass ordinary overriding.
- `protos/lib/crypto/SHA256.protos`: `word32`, logical shift by division, right rotation by arithmetic, `xor3`/`choose`/`majority` via three 64-entry tables with sixteen two-bit extraction cycles. This is direct evidence of source complexity, NOT a performance measurement. The SHA-256 library has production consumers in `protos/tools/package/{ProjectMetadata,ContentIdentity,ResolutionInput}.protos` and source conformance tests under `protos/tests/library/crypto/sha256/`.
- `protos/lib/math/Integer.protos` owns existing Protos-first math algorithms, not bit operations. `docs/design/STANDARD_LIBRARY_IDEAS.md` §§Bit operations discusses a potential optional Bits library; it is exploratory, not ratified.
- Historical constraints: [D156/#616](https://github.com/guillermomolina/protos/issues/616) and [I052/#628](https://github.com/guillermomolina/protos/issues/628) remove fixed-width Core families; [D197/#865](https://github.com/guillermomolina/protos/issues/865) specifies the newer mathematical SmallInteger/BigInteger split; [AUD009-E3/#649](https://github.com/guillermomolina/protos/issues/649) retains both existing modules and expressly excludes the general bitwise API from *its* prior audit; [INC004/#1010](https://github.com/guillermomolina/protos/issues/1010) motivates but does not approve D230.

## Primary-source comparative precedents (at least five / three approaches)

| System | Semantic evidence | Lesson / non-transferable rule |
| --- | --- | --- |
| [Python int — language expressions](https://docs.python.org/3/reference/expressions.html#binary-bitwise-operations) | Bitwise operations on signed unlimited integers behave with infinite two's-complement sign extension; right shift negative values rounds down | Good model for exact `bitAnd/Or/Xor/Not` and right shift; Python's operator syntax is not required |
| [Java BigInteger](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/math/BigInteger.html) | `and/or/xor/not/shiftLeft/shiftRight` on unbounded signed integers; two's-complement semantics | Host-independent mathematical precedent; do **not** copy Java's host `int` shift-count bound or negative-count reverse-direction behavior |
| [JavaScript BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) and [asUintN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt/asUintN) | Separate unlimited BigInt bit semantics from explicit N-bit unsigned projection; no BigInt unsigned >>> | Finite unsigned semantics need an explicit word width, not a guessed default width or Numeric value family |
| [GNU Smalltalk Integer methods](https://www.gnu.org/software/smalltalk/manual-base/html_node/Integer_002dmath-methods.html) | `bitAnd:`, `bitOr:`, `bitXor:`, `bitInvert`, `bitShift:` message surface | Proof that message-oriented bit protocols do not need new operator grammar |
| [Rust i32/u32](https://doc.rust-lang.org/std/primitive.i32.html) | Typed fixed-width arithmetic, `rotate_left/right`, checked/wrapping shift variants | Strong alternative *for a width-bearing domain*, but conflicts with D156 if introduced as new Core families |
| [Go numeric operators](https://go.dev/ref/spec#Arithmetic_operators) | Shifts constrained to integer operands, signed vs unsigned right behavior determined by type | Demonstrates why right shift is not simply truncating mathematical division |
| [SMT-LIB fixed-size bit-vectors](https://smt-lib.org/theories-FixedSizeBitVectors.shtml) | Explicit indexed width with separate bit-vector operators | Sound model for a possible future optional width-aware bit-vector library rather than unlimited Core integers |

These are mathematical and language-level precedents. They do not authorize importing their syntactic operators, coercions, signedness categories, runtime exceptions, overflow modes, or FFI types into Protos.

## Alternatives and common 12-criterion scorecard

Options: **A** no Core change / keep private algorithms; **B** pure Protos-first optional `std:math/Bits`; **C** minimal six Core Integer messages (owner-approved C1); **D** runtime substrate + optional library front; **E** width-bearing optional word/bit-vector abstraction.

Scale **1=poor, 5=strong**; confidence **H/M/L** (high/medium/low). Each cell includes its rationale. Ratings are design judgments, not measurements. Scores must not trump qualitative overengineering constraints.

| Criterion | A | B | C1 | D | E |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness/invariants | 5H no change | 5M exact algorithms | 5M portable laws | 4M extra boundary | 5M explicit words |
| 2 Protos philosophy | 3M repeated workarounds | 5M ordinary optional module | 5M small general protocol | 3M privileged substrate | 3M new abstraction |
| 3 Present-need proportionality | 3H zero new API | 5H pay on import | 4M six generic selectors | 2M native machinery unproven | 2M word model premature |
| 4 Incremental growth | 4M future additions | 4M library extensible | 5M stable mathematical base | 3M two contracts | 4M independent word ops |
| 5 Future-option resilience | 5H no lock-in | 5H replaceable | 4M Core API commitment | 4M backend flexible | 5M separate boundary |
| 6 Scalability | 2M per-bit implementations | 2M pure loops may scale poorly | 5M semantic fast paths possible | 5M potential acceleration | 3L dependent on representation |
| 7 Conceptual simplicity | 3M scattered complexity | 4M one module | 5M six precise laws | 2M duplicate public/private rules | 2M width/overflow policy |
| 8 Portability | 5H existing exact math | 5M guest source | 5M host-independent laws | 4M machinery-specific traps | 5M explicit math |
| 9 Runtime/resource costs | 2L SHA workaround | 2L repeated math/tables | 4L potential native path | 4L potential native path | 3L representation-dependent |
| 10 Failures/operability | 3M ad hoc validation | 4M library boundary | 5M standard operand/errors | 3M extra failure layers | 4M width validation |
| 11 Deferral/reversibility | 5H no API | 5M removable module | 3M permanent selectors | 2M two contracts persist | 3M broader API |
| 12 Evidence maturity | 5H known code | 4M implementable in source | 4M strong precedents | 2L no performance proof | 3M external models |

**Qualitative red flags:** A retains demonstrated repeated low-level source work; B may have unacceptable costs at bit-intensive scale but has no measurements; C incurs permanent Core compatibility without measured performance proof; D pre-builds privileged machinery; E pre-builds width-bearing state/identity not established by present needs. The table favors considering B vs C and does not authorize any automatic numerical winner.

### Incremental gates and deferred cost

- **Smallest sufficient general bit protocol:** six operations; algebraic reductions could use fewer primitives but reintroduce repeated indirect implementations. SHA-256 alone is not proof of a need for popcount, masks, rotations or bit-vector identity in Core.
- **Pay for what is needed:** C only adds standard Integer selector/protocol semantics; no array of fixed-width types, parser ladder or global state. Its implementation must avoid cost for users not sending bit operations.
- **Grow later:** explicit-width words, wrapping/saturating arithmetic, byte-order formats, signed/unsigned projection, popcount and bit tests can be independently added outside Core without invalidating the mathematical operations.
- **Deferring C:** would require rewriting existing library algorithms and possibly application source, not a foundational runtime/type migration. This is B's strongest case.
- **Regret scenario and escape:** if future workloads overwhelmingly require a persistent width-bearing word representation, a separate ordinary width-aware domain remains possible; the six Core methods remain valid and do not force width-bearing identity. If C were removed later, public selector compatibility would break, so its scope must remain minimal.

## Adversarial falsification and future acceptance vectors (not executed)

| Case | Expected C1 result or invariant |
| --- | --- |
| `0.bitNot()` | -1 |
| `(-1).bitNot()` | 0 |
| `5.bitNot()` | -6 |
| `x.bitNot().bitNot()` | mathematical x |
| `(-1).bitAnd(42)` | 42 |
| `(-1).bitXor(5)` | -6 |
| `x.bitOr(0)`, `x.bitXor(x)`, `x.bitAnd(-1)` | x, 0, x |
| `(-3).shiftRight(1)` vs `(-3).div(2)` | -2 vs -1 |
| `1.shiftLeft(64)` | 18446744073709551616; BigInteger |
| `(-1).shiftRight(1000000)` | -1; no masking |
| `1.shiftRight(2^100)` / `(-1).shiftRight(2^100)` | 0 / -1 |
| `0.shiftLeft(2^100)` | 0; no host shift-counter overflow |
| `1.shiftRight(-1)` and `1.shiftLeft(-1)` | ordinary Error |
| Non-Integer receiver, Float/Fraction/Complex argument, delegating ordinary object, wrong arity | ordinary Error; no implicit conversion |
| `SmallInteger`/ `BigInteger` crossed both directions through AND/shift | exact canonical family determined by result, not receiver |
| Locally shadowed `bitAnd` method | ordinary selected override executed; canonical fast path must not bypass it |
| Numeric equality/hash/identity after bit operation, Actor/P transfer | unchanged contracts and destination-owned prototypes |
| JVM vs Native Image | same bit values, family, dispatch/error semantics |
| SHA-256 known vectors for empty, abc, binary 00 FF 01 FE, 55/56/64-byte lengths, multiblock | unchanged Bytes, digest and input immutability |

Do not infer that producing correct SHA-256 proves the Core protocol is necessary or fast. Benchmark independently from semantics, on the same published revision with correctness proven. Compare cold/hot time, allocation, operand width, heavy shifts and real SHA-256 throughput on JVM and Native Image when available. No constant-time/side-channel cryptography guarantee is approved.

## Scope and authority audit

- Current Protos docs inspected: root `AGENTS.md`; `AGENTS.work/DESIGN.md`, `REFERENCE.md`, `COORDINATION.md`; `spec/AGENTS.md`, `src/AGENTS.md`, `protos/AGENTS.md`; `spec/PROTOS_LANGUAGE_SPEC.md`, `PROTOS_GRAMMAR.md`, `semantics/VALUES_AND_COLLECTIONS.md`, `semantics/OBJECT_MODEL.md`, `semantics/ERRORS.md`; `docs/design/STANDARD_LIBRARY_IDEAS.md`; `protos/lib/core/Integer.protos`, `protos/lib/math/Integer.protos`, `protos/lib/crypto/SHA256.protos`; Java numeric protocol, support and guarded fast-path files; SHA-256 library tests; D156/I052/AUD009-E3/AUD011/INC004 issues.
- `spec/semantics/VALUES_AND_COLLECTIONS.md` is the canonical owner for normative bit semantics. The approved decision record is non-normative; normative publication and actual conformance belong to I135.
- All source reading was read-only. No commands/programs/tests/builds/benchmark were executed; no performance, implementation, portability or native correctness PASS is claimed.
- **OWNER_APPROVED_EXACT_C1=PASS; NO_FIXED_WIDTH_CORE_FAMILY=PASS; NO_NEW_SYMBOLIC_OPERATOR=PASS; NO_IMPLEMENTATION_IN_THIS_RECORD=PASS.**
