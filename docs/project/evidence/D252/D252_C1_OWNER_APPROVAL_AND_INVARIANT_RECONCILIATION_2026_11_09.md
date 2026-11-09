# D252-C1 — Owner approval, comparative investigation and GITHUB021 invariant reconciliation (2026-11-09)

**Formal decision:** [D252 #1098](https://github.com/guillermomolina/protos/issues/1098).
**Approved candidate:** **C1 — normal `each(block)` on String, Vector and Matrix**, with one element argument, scalar Strings, Vector ascending index and Matrix row-major ordering, shallow numerical snapshots, original receiver return, early callback validation and ordinary suspension/control transfer.
**Exact owner statement:** **“perfecto me gusta C1, apruebo”** (2026-11-09, after the complete D252 comparative research and a shorter explicit comparison of B1 versus C1). [GitHub issue approval record](https://github.com/guillermomolina/protos/issues/1098#issuecomment-6731616772).
**Non-normative durable decision:** [D252 C1 decision](../../decisions/language/D252_COLLECTION_TRAVERSAL_STRING_VECTOR_MATRIX_C1_RATIFIED.md), first published in [project-docs commit `b8e0e19eb96d307a6983713cf195566ce2c544dd`](https://github.com/guillermomolina/protos-project-docs/commit/b8e0e19eb96d307a6983713cf195566ce2c544dd).
**Exact product baseline:** [`guillermomolina/protos@52b6da5da98e7eedb525ab4eb5111258c136193e`](https://github.com/guillermomolina/protos/commit/52b6da5da98e7eedb525ab4eb5111258c136193e).
**Operation scope:** Decision record/evidence and live coordination only. No product `spec/`, implementation, tests, version bump or benchmark executed by this ratification.

## A. Approval provenance and ownership boundaries

1. [AUD023 #1087](https://github.com/guillermomolina/protos/issues/1087) completed on 2026-11-07 after **B** was approved. That approval is historical fact; it remains in the original closure and immutable evidence.
2. On 2026-11-09 the owner changed the **preferred next direction** to **AUD023-C**, recorded at [comment 6731237139](https://github.com/guillermomolina/protos/issues/1087#issuecomment-6731237139): narrowly scoped new iteration on String/Vector and optionally Matrix; **not** acceptance of precise callback rules.
3. D252 was independently allocated for that semantic choice, as [D252 #1098](https://github.com/guillermomolina/protos/issues/1098), remaining OPEN/READY until owner selection.
4. The owner then approved **C1**, verbatim **“perfecto me gusta C1, apruebo”**, after seeing the explicit candidate matrix, proposed String/Vector/Matrix contract, error/shape/Actor/P constraints and implementation boundaries. Approval was posted as [comment 6731616772](https://github.com/guillermomolina/protos/issues/1098#issuecomment-6731616772).
5. This is a **new, bounded public protocol addition**, not a retroactive rewriting of the AUD023 audit or a unilateral implementation decision.

## B. GITHUB010 alternatives and trade-off evidence

The 2026-11-09 investigation compared four coherent D252 choices:

| Candidate | New standard `each` | Advantage | Limitation / why not chosen |
| --- | --- | --- | --- |
| A0 | None | No change and no present runtime cost | Does not serve owner-selected AUD023-C capability |
| B1 | String, Vector | Smallest incremental addition; Matrix can follow later | Retains Matrix's non-iterability despite its canonical row-major element order |
| **C1 — selected** | **String, Vector, Matrix** | `for(x in value)` covers all three with ordinary `each`; no new hierarchy/syntax | Matrix numerical snapshots can cost O(rows × columns), only on invocation |
| D1 | Generic Iterable/Iterator hierarchy, axis views and adapters | Broadest speculative genericity | Unnecessary cost, contradictory/incompatible universality and new lifecycle burdens |

The qualitative 1–5 comparison presented to the owner (higher favorable; predictions, **not benchmarks**) used all twelve GITHUB010 axes: invariant preservation (5/5/5/2); Protos philosophical fit (3/5/5/2); present proportionality (4/5/4/1); incremental growth (3/5/5/2); future option preservation (4/5/5/2); scalability (4/5/5/3); conceptual simplicity (4/5/4/1); portability (5/5/5/3); runtime/resource cost (5/4/4/2); failures/operability (5/5/5/2); reversibility/migration (2/4/5/2); evidence/implementation maturity (5/4/4/3). Each tuple is **A0 / B1 / C1 / D1**, respectively. Confidence in normative invariant classification was high; performance/implementation forecasts for new methods are medium and D1 implementation forecasts lower. These are deliberation aids, not acceptance proof or arithmetic selection authority.

**Strongest counterargument to C1:** Large Matrix snapshots may be expensive, and callers sometimes need row/column coordinates rather than scalar values. **Response accepted as design trade-off:** one ordinary value iteration contract now; future distinct `eachCell` / `eachRow` or read-only no-copy traversal may be evaluated independently, without silently weakening C1. **Strongest alternative B1:** defer Matrix entirely at zero compatibility cost; the owner preferred eliminating the remaining `for` gap now.

Prior-art contrasts considered: Smalltalk/Pharo `do:` ordinary callback iteration; Ruby/TruffleRuby explicit String traversal units; Python/GraalPy and JavaScript/GraalJS Unicode/iterator contrasts; Swift Character versus unicodeScalars; Rust `chars`, `char_indices` and ndarray axis iteration; Java UTF-16 `chars` versus `codePoints`; TruffleString's codepoint cursor; Pkl generator semantics. None supplies normative authority for Protos, whose existing semantics decide the units and object model.

## C. Source-backed exact baseline — files materially inspected

The D252 research used the current product repository source/spec at the exact revision listed above. The following paths were directly inspected or located for the decision and have these precise relevance boundaries:

| Product path | Relevant preserved fact |
| --- | --- |
| `AGENTS.md`; `AGENTS.work/DESIGN.md`; `AGENTS.work/COORDINATION.md`; `AGENTS.work/REFERENCE.md`; `spec/AGENTS.md` | Owner approval, GITHUB010/GITHUB021, unique formal issues and invariant retention |
| [`spec/semantics/VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/spec/semantics/VALUES_AND_COLLECTIONS.md) | String scalar indexing and immutability; Array/Bytes snapshot `each`; D214 Vector/Matrix shape, indexing and numeric storage; Range's distinct no-snapshot return-`null` traversal |
| `spec/semantics/EXECUTION_AND_CONTROL.md`; `spec/semantics/CALLABLES.md`; `spec/semantics/OBJECT_MODEL.md`; `spec/PROTOS_GRAMMAR.md` | `for` is ordinary `each` lowering; ordinary invocation, closures, result propagation and member lookup |
| `spec/concurrency/ACTORS.md`; `spec/concurrency/PARALLEL_EXECUTION.md` | Actor/P transfer isolation, non-local return boundaries, no automatic parallel iteration |
| [`ProtosStandardStringProtocol.java`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardStringProtocol.java) | `size`, `at`, `+`, `concat` installed; `each` not yet installed |
| [`ProtosStringValue.java`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/runtime/values/ProtosStringValue.java) | Validated UTF-16 backing and scalar count; index-from-start scalar lookup argues for O(n) sequential traversal |
| [`ProtosStandardLinearAlgebraProtocol.java`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java) | Module-scoped frozen Vector/Matrix standard prototypes, numeric operations, no `each` |
| [`ProtosVectorValue.java`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/runtime/values/ProtosVectorValue.java); [`ProtosMatrixValue.java`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java) | Indexed snapshot capability; R1/R2-Float Matrix physical representations and zero-volume shape concerns |
| `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/structured/ProtosStructuredEachCalls.java`; `.../ProtosGuardedSendSelection.java`; `.../ProtosSendPreparation.java` | Existing structured Array/Bytes `each` callback/suspension machinery is an implementation integration constraint, not a newly approved public runtime category |
| `docs/guide/04-control-flow-through-protocols.md`; `docs/guide/library/linear-algebra.md`; `protos/lib/math/linear_algebra/module.protos` | Public documentation and Standard Library module topology to update **only after code/spec publication** |

Relevant earlier decision/audit evidence: [D213 #949](https://github.com/guillermomolina/protos/issues/949), [D214 #955](https://github.com/guillermomolina/protos/issues/955), [AUD023 #1087](https://github.com/guillermomolina/protos/issues/1087), [AUD009-B9 #620](https://github.com/guillermomolina/protos/issues/620), [AUD009-E1 #646](https://github.com/guillermomolina/protos/issues/646), D157 and D231. The [AUD023 2026-11-07 matrix](../../work/AUD023/AUD023_COLLECTION_PROTOCOL_COHERENCE_AUDIT.md) remains an exact historical audit rather than a current implemented-protocol assertion.

## D. Explicit GITHUB021 owner-invariant/delta review

| Previously ratified invariant or historical decision | Proposed C1 delta | Disposition |
| --- | --- | --- |
| D157: `String.size` and `String.at` count Unicode scalars; String immutable | `String.each` visits exactly those scalar-String values | **PRESERVE** |
| D213/AUD009: Core Arrays fixed-length and array copy/edit operations nonmutating | No Array change | **PRESERVE** |
| D214: Vector and Matrix fixed shapes; `add` algebraic; Matrix no multidimensional bracket syntax | Add only standard traversal to existing prototypes | **PRESERVE** |
| D214: Matrix row-major constructor; packed Float optimization must be invisible | `Matrix.each` row-major logical snapshot independent of representation | **PRESERVE** |
| D231: generic `for` is an ordinary single-argument callback `each` send | Add receivers that implement the selector; no parser change | **PRESERVE** |
| Array/Bytes/Map/Range standard `each` different iteration and result laws | New receiver-specific law; no global universal rule | **PRESERVE** |
| Actor/P isolation; `freeze` shallow; callbacks may suspend | Snapshot per invocation, ordinary control transfer, no extra transfer authority | **PRESERVE** |
| AUD023 B (2026-11-07): document existing contracts; defer new String/Vector `each` | Owner explicitly changed preferred future direction to AUD023 C on 2026-11-09 | **EXPLICITLY SUPERSEDED AS FUTURE POLICY**; historic closure left intact |
| AUD023 historic absence of standard String/Vector/Matrix `each` | D252 owner approved additive three-family `each` | **EXPLICITLY REOPENED AND REPLACED** within bounded C1 scope |
| No universal Collection/Iterator hierarchy | C1 adds no hierarchy, row views or generic `add` | **PRESERVE** |

**GITHUB021 result:** no unapproved contradiction identified; the **only** behavioral addition is the specifically presented/approved new standard `each` on the three named domains. Other invariants remain in force. Scope exclusions and performance risk must carry into implementation.

## E. Implementation and verification not yet completed

Product `spec/` and code at the baseline do **not** implement these methods. A later Ixxx must:
- update canonical normative owners and standard-module documentation;
- add String scalar cursor iteration, Vector/Matrix snapshot iteration, ordinary callable validation and correct receiver return;
- integrate nested callbacks with Task/Actor suspension and non-local return, no runtime-visible dependency on Java `String` or Matrix R2 storage;
- test receiver and arity failures, non-invokable callback on empty, mutation during iteration, frozen/closed objects, zero-volume distinct Matrix shapes, Unicode combining sequences/supplementary characters, embedded lookups and shadowing, R1/R2, Task and Actor/P isolation;
- run the required focal/full/Native gates by human executor; record product commit, spec revision, version/changelog only after completed validations.

**Observed verification for D252 ratification:** source/read-only inspection and GitHub owner response, not execution. `PRODUCT_SOURCE_MUTATION=NONE`; `D252_TESTS=NOT_RUN`; `BENCHMARKS=NOT_RUN`; `NORMATIVE_C1_SPEC_PUBLISHED=NO`; `IMPLEMENTATION=NOT_STARTED`.

```text
FORMAL_IDENTIFIER=D252
FAMILY=D
OWNER_APPROVED_CANDIDATE=C1
APPROVAL_DATE=2026-11-09
APPROVAL_PROVENANCE=https://github.com/guillermomolina/protos/issues/1098#issuecomment-6731616772
GITHUB021_INVARIANT_RECONCILIATION=PASS
PRODUCT_REVISION=52b6da5da98e7eedb525ab4eb5111258c136193e
PRODUCT_SOURCE_AND_SPEC_CHANGES=NONE
DURABLE_DECISION=docs/project/decisions/language/D252_COLLECTION_TRAVERSAL_STRING_VECTOR_MATRIX_C1_RATIFIED.md
D252_TESTS=NOT_RUN
C1_IMPLEMENTATION=NOT_STARTED
```
