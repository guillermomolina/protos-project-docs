# D198 — Owner selection of Candidate C architecture (2026-10-12)

**Formal design issue:** [D198/#876](https://github.com/guillermomolina/protos/issues/876)  
**Consuming implementation:** [I090/#868](https://github.com/guillermomolina/protos/issues/868)  
**Previously ratified semantics:** [D197/#865](https://github.com/guillermomolina/protos/issues/865)  
**Historic product baseline for I090-C1:** [`c75dea75b8db045eb6833aeab1d5b8710eaa0556`](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556)  
**Status:** Candidate C **SELECTED FOR ARCHITECTURE ONLY**; D198 remains **OPEN**; no full public Complex semantic contract is ratified.

## Explicit owner-approval provenance and its precise scope

Following an investigation of D198 and a summary of Candidate C, the project owner explicitly said on **2026-10-12**: **"ok apruebo C"**. The immediately preceding exchange deliberately separated the architectural decision from the unresolved exceptional-number and precision questions. The owner had been advised to approve C first **as architecture**, without closing/ratifying D198 until the NaN/Infinity and precision policies are decided. Consequently the approval is **bounded**: it authorizes the architectural direction but neither picks an exceptional-value policy nor ratifies a complete executable API.

### Selected architectural invariants (Candidate C)

1. **Protos-first semantics.** Public Complex protocol, dispatch, operator composition and mathematical algorithms expressible in Protos should live in Protos source rather than duplicating the entire behavior in Java.
2. **Small, justified host kernel.** Java may provide only irreducible value-authentication/representation, safe canonical component access, immutable numeric payload support, normalization, and narrowly necessary arithmetic/bit-level kernels where Protos-source implementation cannot meet verified semantic or numerical obligations. "Small" is an obligation to justify the boundary, not a preapproved list of Java methods.
3. **Ordinary language mechanisms.** Use existing message dispatch and callable conventions, preserve overrideability/guarded fallback per D013, and do not silently invent a second complex-specific multimethod/coercion subsystem or new syntax. Candidate C is not a mandate to privilege every numeric selector in the runtime.
4. **Integrated numeric family.** Complex is a member of the D197 Number hierarchy, with semantic Integer/Fraction/Float components; no forced reduction to a pair of binary64 doubles. D197's exact imaginary Integer-zero demotion and Float signed-zero retention remain binding.
5. **Two-way mixed arithmetic is an API design requirement to resolve.** A public Complex arithmetic implementation cannot consider only `Complex + real`; it must account for the ordinary receiver-first form `real + Complex` without breaking existing real-number primitive fast paths. Exact selector, admission and fallback contracts remain to be ratified.
6. **Pay as you grow.** Programs using only SmallInteger/Float must not incur mandatory Complex boxing, guest-object initialization, Task/Actor overhead or universal arithmetic dispatch overhead solely because Complex exists.
7. **Safe isolation.** Existing authenticated immutable values, Actor/P/Context ownership/transfer, mathematical equality/hash and family-sensitive identity guarantees remain authoritative; a public factory must not bypass them.

These are architectural direction and preservation obligations, **not** ratification of every example shown in the exploratory Candidate C research. In particular the proposed `Complex(real, imaginary)`, `Number.real()`/`imaginary()`, arithmetic selectors and `std:math/Number.sqrt` remain a candidate API sketch whose exact public contracts, error behavior, module placement and operation ordering require an explicit final design checkpoint. D197's preexisting invariants are authoritative independently of these proposed spellings.

## Counterexamples and constraints carried into the remaining decision

- `Complex(3, 0)` must collapse to an exact real result *if* that factory is selected; `Complex(3, +0.0)` and `Complex(3, -0.0)` must preserve distinct signed-zero imaginary information under D197. A universal "imaginary zero collapses" rule is forbidden.
- Exact `1 / 0` signals Error under D197; Float `1.0 / 0.0` is governed by its existing IEEE binary64 contract. Those two domains cannot be merged merely for a convenient Complex implementation.
- A public Complex constructor must reject ordinary objects that merely delegate to `Number` or `Complex`, and must not accept unauthenticated foreign-context guest values. Legitimate semantic component families and destination ownership must be checked.
- Direct `(a*c+b*d) / (c*c+d*d)`-style complex division can overflow or underflow in intermediate binary64 calculations even when the mathematically correct quotient is finite; an unexamined two-double Java implementation is not accepted by selecting C.
- `real + Complex` cannot be implemented by adding only a `Complex.+` method, due to ordinary receiver-first dispatch. A design must prove the complete route and overrideable fallback, while retaining primitive paths.
- NaN breaks ordinary reflexive equality assumptions; IEEE signed-zero, NaN payload/equivalence and infinities must be reconciled with D197 `==`, `===`, hash, transfer, and printing/interop before ratifying the API.
- Exact complex square roots might require expensive rational perfect-square recognition; selecting source-first architecture does not automatically mandate unbounded general symbolic algebra.

## Alternatives considered and why the architectural choice was meaningful

- **A — minimal public construction / defer useful arithmetic:** lowest immediate implementation cost but leaves Complex largely unusable and pushes two-way interoperability and normalization migration downstream.
- **B — exclusively Protos source without native kernel:** maximizes guest-code purity but risks duplicating privileged authentication, exact component access and IEEE boundary algorithms through unsuitable public machinery.
- **C — Protos-first, minimal native kernel (SELECTED):** uses ordinary guest methods for semantics while retaining a narrow verified host boundary for true representation/numeric primitives.
- **D — host-centric pair of doubles:** conflicts with D197 exact BigInteger/Fraction component retention and unnecessarily couples guest semantics to host precision.

The next evaluation must remain adversarial: in each proposed Java operation, ask why ordinary Protos source cannot implement it without measurable regressions or semantic violations; in each Protos-level complex operation, ask whether exact component conversion/normalization and IEEE exceptional cases are actually preserved. The selected hybrid is falsifiable, not a blanket permission to expand either surface.

## Unresolved D198 checkpoints

1. **Exceptional arithmetic:** permitted NaN/+Infinity/-Infinity Float components; explicit behavior for exact and approximate zero divisor, infinities, NaNs, overflow/underflow, Float signed zeros, equality/identity/hash/transfer, actor boundary and interop. **NOT SELECTED**.
2. **Numerical precision:** mixed exact/Float component conversion order, stable Float complex division, `sqrt` rounding/branch behavior, representability and limits of exact complex sqrt. **NOT SELECTED**.
3. **Final executable public API:** exact canonical `call` signatures, component selectors/receiver family, complete two-way arithmetic and `sqrt` module/selector placement, errors. Architectural illustrative examples **NOT separately ratified**.
4. **Normative publication:** `guillermomolina/protos:spec/` remains unchanged by this selection; no source code, tests, product commit or full-suite result is implied by documentation publication.

**Dependency consequence:** I090-A/B/C1 remain completed as previously evidenced. I090's public Complex work remains blocked on the **unresolved D198 semantic contract**, while independent architecture-consistent implementation planning may be researched. D198 must remain OPEN; do not treat Candidate C selection as full D198 closure.

## Inspection / validation boundary

Inspected: `guillermomolina/protos/AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `src/main/java/com/guillermomolina/protos/runtime/ProtosComplexNumericSupport.java`, current D198 and I090 issue records, and `guillermomolina/protos-project-docs/AGENTS.md`, existing D198 work record and D197 decision. Previous comparative investigation also consulted ordinary callable contracts, standard numeric protocols and external language designs.

This is a **documentation-only architectural approval record**. **No product tests, compilations, or normative specification changes** were run or claimed in this transaction. The project owner explicitly authorized the assistant to publish bounded D198 evidence in `guillermomolina/protos-project-docs` on 2026-10-12.
