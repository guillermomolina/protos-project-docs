# D230 — Core Integer bitwise operations: approved C1

**Formal decision:** D230 / [issue #1011](https://github.com/guillermomolina/protos/issues/1011).
**Owner approval:** 2026-11-01 in the active design conversation, exact response **“Apruebo C1.”** This refers to the fully presented D230-C1 proposal containing six specifically spelled named Integer messages, mathematically unbounded two's-complement behavior, strict Integer operands, and explicit deferral of width-bearing operations and new grammar.
**Selected:** C1 — standard Core `Integer` message protocol, with finite-width policy outside Core.
**Inspected Protos product revision:** [`73bd9fe913421ff493146b546869f9a1f0d2b34e`](https://github.com/guillermomolina/protos/commit/73bd9fe913421ff493146b546869f9a1f0d2b34e) (read-only investigation). Product specification, runtime, benchmarks and tests are **not** changed by this record; this record is non-normative.
**Related:** [INC004 #1010](https://github.com/guillermomolina/protos/issues/1010); [D156 #616](https://github.com/guillermomolina/protos/issues/616); [I052 #628](https://github.com/guillermomolina/protos/issues/628); [AUD009-E3 #649](https://github.com/guillermomolina/protos/issues/649).
**Research evidence:** [D230 comparative research and falsification](../../evidence/D230/D230_C1_COMPARATIVE_RESEARCH_2026_11_01.md).

## Ratified minimal Core surface

All six are **ordinary named methods on the canonical common `Integer` prototype**, reachable by ordinary delegation from semantic `SmallInteger` and `BigInteger` values. Spelling and arity **are** approved:

| Method | Arity | Exact meaning |
| --- | ---: | --- |
| `a.bitAnd(b)` | 1 | Bitwise AND in infinite sign-extended two's-complement |
| `a.bitOr(b)` | 1 | Bitwise OR in the same model |
| `a.bitXor(b)` | 1 | Bitwise exclusive OR in the same model |
| `a.bitNot()` | 0 | Complement, mathematically `-a - 1` |
| `a.shiftLeft(n)` | 1 | For `n >= 0`, exact `a * 2^n` |
| `a.shiftRight(n)` | 1 | For `n >= 0`, exact `floor(a / 2^n)` |

The bit representation is a *mathematical interpretation*, not finite storage. There is no implicit signed-64 or unsigned-64 word size, no wrapping/saturation, and no machine shift-count masking. In particular `(-3).shiftRight(1) == -2`, whereas current `(-3).div(2) == -1`. A negative count signals ordinary `Error`; it does not reverse direction. Zero counts are identity. Huge nonnegative counts are semantically valid: positive inputs eventually shift right to 0, negative inputs to -1; zero shifted left by any nonnegative count stays zero. All counts must be genuine mathematical `Integer`, even outside machine `int` range.

For all methods, the **original receiver** must be a genuine semantic Integer (`Integer.recognizes`) and arguments, including counts, must likewise be genuine semantic Integers. An ordinary object delegating to an Integer or to its prototypes is not itself an Integer. Wrong arity, invalid receiver/argument, and negative shift count signal the generic standard `Error`; no new mandatory Error family or coercion is created. Evaluation occurs normally before dispatch; valid standard computation invokes no user callback and introduces no suspension. Results are exact mathematical Integers canonically normalized to `SmallInteger` or `BigInteger` by value, including promotion/demotion at signed-64 boundaries.

The six behaviors are normal slot-lookup/message-dispatch participants. User-defined shadowing/overrides, original `this`, call evaluation, Error handling and established guarded-dispatch provenance checks remain effective. A runtime fast path may not bypass a selected user override on the strength of receiver shape or selector spelling alone. Semantics remain identical across JVM/Native/other implementations, Actor/P transfers, equality, identity, reflection and hashing. Resource constraints never authorize silently incorrect wrapping/truncation or host shift-mask results; do not promise unlimited physical memory or constant-time operation.

## Explicit non-selections / scope limits

- **No additional operators or grammar precedence.** `^` remains non-local return; `&`, `|`, `~`, `<<`, `>>` and `^` as XOR are not introduced. No arbitrary symbolic-operator change.
- **No Core fixed-width signed/unsigned families.** D156's removal of `UInt8`/`Int8` … `UInt64`/`Int64` remains in force, notwithstanding the D197 `SmallInteger`/`BigInteger` mathematical value distinction.
- **No Core finite-width logical right shift, rotate, mask, bit-vector/word value, overflow/wrapping policy, byte-order/serialization, popcount, bitLength, bitTest/set/clear, NAND/NOR/XNOR or cryptographic constant-time promise.** Such capabilities may be proposed through ordinary library/interop domains only after demonstrated need and any necessary separate design approval.
- No implicit ABI/FFI, Bytes, network, SHA-256 public API, Future/Actor/P, primitive carrier, SIMD or Graal/Truffle contract is added. Optimizations are implementation choices, not this decision.

## Owner-invariant reconciliation (GITHUB021)

| Existing owner-approved contract | C1 impact |
| --- | --- |
| D156: eight machine-width Core families removed, capability may live at explicit interop/binary boundary | **Preserved.** Six methods operate on unlimited mathematical Integer, not word families |
| D197: abstract Integer, canonical SmallInteger/BigInteger based on exact value | **Preserved.** Bit results undergo existing canonical normalization |
| D147 numeric family recognition and OBJECT_MODEL original-receiver domain | **Preserved.** No delegation-based admission, no implicit coercion |
| D013/guarded method dispatch and ordinary user selectors | **Preserved.** Only proven canonical method selection may be optimized |
| AUD009-E3: retain `std:crypto/SHA256` and `std:math/Integer`; general bitwise API previously excluded from that audit | **Preserved.** New D230 independently authorizes only six Core messages; no audit reversal |
| Fixed operator grammar and `^` non-local return | **Preserved.** No parser syntax change |
| Numeric equality/hash/identity, Actor/P transfers, resource/representation independence | **Preserved.** Exact ordinary Integer outputs, no new representation exposure |

**SELECTED_CANDIDATE=C1; OWNER_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS.** No new owner approval is inferred for deferred width/word/library or implementation performance choices.

## Necessity, comparative conclusion, strongest counterargument

`protos/lib/crypto/SHA256.protos` at the inspected product revision synthesizes 32-bit XOR/choose/majority with 64-element lookup tables and a 16-pass digit extraction, and rotates/shifts via divisions, modulo and powers. `protos/tools/package/ProjectMetadata.protos`, `ContentIdentity.protos` and `ResolutionInput.protos` consume SHA-256. This **proves source complexity** but is not a timing measurement. Precedents include Python `int`, Java `BigInteger`, JavaScript `BigInt`, GNU Smalltalk, Rust, Go, and SMT-LIB bit-vectors: they illustrate the difference between infinite mathematical bit operations, finite machine words, and explicit bit-vector widths.

A (status quo), B (pure Protos-first optional `std:math/Bits`), C (Core messages), D (private native substrate + public library), and E (width-carrying optional bit-vectors) were compared across all 12 criteria and adversarial cases in the evidence. **B is the strongest counterargument**: it can provide the same semantics without a permanent Core selector compatibility obligation, and its potential performance gap has not been measured. The owner nevertheless selected C1 because six stable, general, mathematically defined Integer operations avoid library-level reinvention while keeping finite-word institutions outside Core. No measured speedup is claimed.

## Revision-coupled downstream work and delivery boundary

The normative owner for the new numeric behavior is `guillermomolina/protos:spec/semantics/VALUES_AND_COLLECTIONS.md`; update its current "no bit-manipulation" exclusion, define full operand/result/error semantics, and keep `spec/PROTOS_LANGUAGE_SPEC.md` cross-references coherent. No grammar edit is warranted. The implementation may reuse `protos/lib/core/Integer.protos`, `ProtosStandardIntegerProtocol.java`, `ProtosNumericValueSupport.java` and current guarded numeric dispatch, with exact-source tests and a bounded SHA-256 consumer rewrite. New public selectors cannot be optimized by assuming host Java `long` shift behavior. Apply `spec/AGENTS.md` global spec-revision discipline; version/changelog only in the later product commit.

Implementation ownership: **one atomic `I135` slice in `guillermomolina/protos`**, independently authorized after this decision's durable publication; do not split into implementation sub-slices. The human executes builds, tests, checks, versioning/publication operations under the current project policy. A future measured performance follow-up, if needed, is not pre-approved.

**Publication/validation at decision-record stage:** docs-only GitHub publication, no Protos product edit, no tests/benchmarks executed, no product version/spec revision bump, no production implementation or performance acceptance claimed. The exact docs publication revision is recorded in D230's live Issue after GitHub confirms and the record is re-read.
