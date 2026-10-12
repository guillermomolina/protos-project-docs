# D198 — Owner approval of E2: Complex exceptional arithmetic signaling envelope

**Date:** 2026-10-12  
**Decision:** [D198/#876](https://github.com/guillermomolina/protos/issues/876) — **OPEN**, not fully ratified.  
**Implementation consumer:** [I090/#868](https://github.com/guillermomolina/protos/issues/868) — remaining public Complex functionality **BLOCKED on D198**.  
**Governing model:** [D197/#865](https://github.com/guillermomolina/protos/issues/865) — closed/ratified.  
**Preceding D198 bounded approvals:** [C, Protos-first minimal host kernel](D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md) and [B, all genuine Float components including NaN and infinities](D198_B_NONFINITE_FLOAT_COMPONENT_ADMISSION_OWNER_APPROVAL_2026_10_12.md).  
**Proposal evaluated:** [six-system E1/E2/E3 comparative investigation](D198_COMPLEX_EXCEPTIONAL_ARITHMETIC_AND_ZERO_DIVISION_RESEARCH_2026_10_12.md) (published at [`e36a1f1f3aff4a043d86a8a78f06f6eb8555ca11`](https://github.com/guillermomolina/protos-project-docs/commit/e36a1f1f3aff4a043d86a8a78f06f6eb8555ca11)).

## Explicit owner approval provenance

On **2026-10-12**, after a concrete explanation comparing candidates E1/E2/E3 and emphasizing that E2 would **not** yet settle any exact NaN/Infinity/signed-zero output tuple, the project owner explicitly said **"apruebo E2"**. The same message said **"el git diff check esta limpio. Todos los tests han pasado en local"**. Those test statements are human-reported; they do **not** establish validation of any not-yet-implemented E2 public behavior and no new product/source changes were made as part of this documentation coordination.

**Selection scope: the Error-versus-numeric-result policy only.** The owner did **not** approve a floating-point complex algorithm, a universal final-result table, a public factory selector, source modifications, or complete D198 ratification.

## Approved E2 semantics

1. **Exact / exact:** Standard Complex division where *all relevant numerator and denominator components are exact semantic Integers/Fractions* remains mathematical exact rational arithmetic. Dividing by the exact mathematical complex zero signals a fresh normal Protos `Error` under existing Core error semantics. No artificial Float conversion, infinity or NaN is produced merely because exact zero was encountered.
2. **Approximate-domain operations:** When standard Complex arithmetic enters an inexact domain because a genuine semantic Float component participates in the operation, floating numerical exceptional conditions **do not in themselves signal Protos `Error`**: zero denominator, NaN, Infinity, overflow and underflow yield numerical Float-bearing values in accordance with a **future explicit, portable complex-result table**. This preserves the ratified Float-domain non-signaling policy *at the failure/result-domain level*, without pretending scalar IEEE specifies the full Complex result.
3. **Non-finite operands remain valid:** Under already-approved B, a genuine Float NaN or infinity is not, by itself, an error at arithmetic admission. Normal wrong-arity, incompatible non-Number, forged family, invalid original receiver, or other independently normative protocol failures can still signal Error. E2 is not blanket suppression of errors.
4. **Existing semantic identities retained:** Preserve Float ±0.0, canonical Float NaN `===` identity and nonreflexive `==`, no payload exposure, all exact component values without double conversion, exact-integer imaginary-zero demotion, retained signed Float imaginary zero, numeric hash and IdentityMap laws, actor/Context isolation and transfer.
5. **No special complex infinity sentinel:** Distinct authenticated component pairs remain semantically observable through existing rules. The project has not selected a projective infinity value/family or new global sentinel.
6. **Ordinary dispatch / pay-as-you-grow:** Candidate C Protos-first implementation with minimal justified native numeric kernels and D013 overrideable standard messages remains binding; no mandatory Integer/Float primitive hot-path overhead.

### Concrete examples — provisional API syntax, approved failure envelope

```text
1 / 0                                  -> Error                    [existing D197]
1.0 / 0.0                              -> Float +Infinity          [existing D197]
0.0 / 0.0                              -> Float NaN                [existing D197]
Complex(1,2) / Complex(0,0)            -> Error                    [E2 exact case]
Complex(1,2) / Complex(0.0,+0.0)       -> numeric result, not Error solely due to zero [E2]
Complex(1,2) / 0.0                     -> numeric result, not Error solely due to zero [E2]
Complex(NaN,-0.0) + Complex(2,3)       -> no Error solely because of NaN         [E2]
```

**IMPORTANT:** `Complex(...)`, `Float.nan` and `Float.infinity` are illustrative; their final public selector spellings and arities are not ratified. E2 does **not** select the numeric result components, signs or required NaN recovery behavior of the examples.

## GITHUB021 invariant/delta audit

| Existing owner-approved invariant | E2 effect | Conclusion |
|---|---|---|
| D197 exact mathematical division by exact zero signals Error | Keeps Error for all-exact Complex division by exact complex zero | PRESERVED |
| D197 Float binary64 zero/NaN/Infinity/overflow are numerical outcomes | Inexact Complex operations follow the same non-signaling exceptional *policy*, without asserting a complete output tuple | PRESERVED |
| D197 exact Integer/Fraction components and canonical normalization | Exact complex arithmetic remains exact and normalized; no eager all-double carrier | PRESERVED |
| D197 Float signed zero, NaN identity, hash, equality, Actor/P/Context transfer | No proposed change to these observable values | PRESERVED, implementation conformance pending |
| D198 architecture C | Ordinary Protos semantics with limited numerical Java kernel only if justified | PRESERVED |
| D198 B admitting every genuine Float component | Non-finite component admission remains valid for Complex operations | PRESERVED |
| Previous explicit owner choice about *complex arithmetic output tables* | No prior table had been approved | NO CONFLICT / NOT SELECTED |

There is no hidden contradiction or retroactive supersession of D197, C or B. E2 only selects the exceptional failure-vs-value boundary.

## Open owner decisions (must NOT be inferred)

1. **Detailed complex arithmetic / precision policy:** numerical result-family/component-preservation and component-conversion ordering for mixed exact/Float (including whether Float contagion acts per component, operation, or pair); finite arithmetic rounding/stability; complete `+`, `-`, `*`, `/` and unary negation results for NaN, infinities, signed zeros, over/underflow, spurious NaNs, exact-zero collapse and consistent `==`/hash/identity.
2. **Complex `sqrt` and branches:** exact perfect rational roots, negative exact squares, inexact nonperfect roots, signed-zero selection around the negative real branch cut, NaN/Infinity special inputs, tolerance/rounding, extremely large operands and portable implementation constraints.
3. **Public API finalization:** exact `Complex(real, imaginary)` call receiver/arity/factory conditions and ownership, `real`/`imaginary` location and inheritance, two-way mixed dispatch, error/protocol validation, `std:math` module/function placement and end-to-end source acceptance. Previous research examples remain proposals, not a ratified API.
4. **Publication and I090:** D198 still OPEN; no normative spec change, product implementation or new Issue has been authorized. I090 remains BLOCKED *for its pending public Complex remainder*; A/B/C1 retain historical completion.

**Next slice:** a consolidated **D198 implementation-independent INVESTIGATION** of IEEE exceptional output tables + mixed numeric precision/stability + `sqrt` and any inseparable final public selectors, from current GitHub repository evidence and external authoritative numerical standards/implementations. Human executor runs **no builds/tests/Git** for an investigation; the agent reads and reports and stops for explicit owner approval. Do not allocate a new formal Issue for work already within D198.

## Publication and execution evidence

Materially inspected and governing: `guillermomolina/protos/AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, the exact rules in `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/ERRORS.md`, `spec/semantics/CALLABLES.md`, internal `ProtosComplexNumericSupport.java`, `ProtosStandardComplexProtocol.java`, the ratified D197 durable record, D198 comparative E2 study, [D198 #876](https://github.com/guillermomolina/protos/issues/876), [I090 #868](https://github.com/guillermomolina/protos/issues/868), and docs-repository `AGENTS.md`. No extra source code was changed or executed.

The user reported `git diff --check` clean and **all local tests PASS**; **repository/HEAD, test names, run logs and commit revision were not supplied with that report**, so they may not be used as claimed proof of an E2 implementation or published commit. The assistant performed GitHub coordination and bounded docs direct publication under explicit task-specific owner authorization. No local Git/validation commands or product tests executed by agent.

```text
D198_ARCHITECTURE=C_OWNER_SELECTED
D198_NONFINITE_FLOAT_COMPONENTS=B_OWNER_SELECTED
D198_EXCEPTION_POLICY=E2_OWNER_APPROVED_2026_10_12
D198_PRECISE_EXCEPTION_RESULT_TABLE=OPEN
D198_PRECISION_SQRT_PUBLIC_SELECTORS=OPEN
D198_FULL_RATIFICATION=NO
I090_REMAINDER=BLOCKED
LOCAL_TESTS=PASS_AS_REPORTED_BY_OWNER_UNSCOPED
PRODUCT_SPEC_TEST_EDITED_BY_AGENT=NO
```
