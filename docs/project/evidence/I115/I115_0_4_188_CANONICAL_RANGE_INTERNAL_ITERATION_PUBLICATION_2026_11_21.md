# I115 — Canonical Range construction and inline each/eachStep publication (2026-11-21)

**Record type:** immutable, revision-bound, non-normative product publication and validation evidence. This records a post-closure I115 implementation publication. It does not reopen I115, alter the normative specification, or close independent PERF045.

## Exact publication and ownership

- **Product:** [`guillermomolina/protos@a5e4d6089a0764d4a55ebc033ac47cb2c7e0ccc2`](https://github.com/guillermomolina/protos/commit/a5e4d6089a0764d4a55ebc033ac47cb2c7e0ccc2), version **`0.4.188-SNAPSHOT`**, 17 changed paths.
- **Commit subject:** `I115: Construct canonical Ranges and iterate Range each/eachStep internally behind runtime identity proofs and refresh the PE baselines`.
- **Formal implementation issue:** [I115 #931](https://github.com/guillermomolina/protos/issues/931), previously **closed/completed** on 2026-11-20 after the 18-workload structural campaign; this later product commit does not invalidate or retroactively alter that closure.
- **Independent performance investigation:** [PERF045 #889](https://github.com/guillermomolina/protos/issues/889), still **open/ready**. Its full each-family causal/physical-performance acceptance is not implied by this I115 slice.
- **Separate diagnostics defect:** [BUG039 #1143](https://github.com/guillermomolina/protos/issues/1143), remains **open/ready**.

## Published implementation

The source commit extends the existing Truffle Bytecode DSL execution and lowering, retaining Protos message syntax and dynamic semantics:

1. **Canonical Range creation:** direct `Range(a, b)` and `InvertedRange(a, b)` calls can use separate `IsStandardRangeConstructionSelection` and `MakeRange` operations after runtime identity/behavior proofs. The proof covers the selected factory, Range, standard Number equality, and standard Object `parent`/`freeze` behavior. A constructed result is still a fresh, ordinary frozen Protos object with the appropriate prototype, `start` and `stop` slots.
2. **Canonical traversal:** eligible Range/InvertedRange `each` and `eachStep` use separate `IsStandardRangeIterationSelection`, `PrepareRangeIteration`, and `RangeIterationNext` operations and an internal Bytecode DSL loop with one lexically inlined callback body. Both signed-64 `long` and arbitrary-precision Integer bound/overflow paths are retained; ascending, descending, equal-bound and step-sign rules remain source-semantic.
3. **Dynamic fallback:** aliased and shadowed factories, named/spread argument forms, custom receivers, and unproven or redefined selections retain normal dispatch. Ordinary two-argument calls such as `callback(7, 9)` retain their direct lowering shape instead of paying global Range admission/local-save overhead. Aliases retain their guest semantics without a guarantee of the canonical construction fast path.
4. **Callback/lowering proof:** `CanonicalCompilationFacts`, `CanonicalInlineStoragePlan`, and `CanonicalToBytecodeLowerer` share site-aware inline admission so closure-context/ReturnHome demands are retained on the ordinary branch. Existing context identity, captures, non-local return, error, nesting, escape and suspension semantics remain required.
5. **Canonical method identity:** `ProtosStandardNumberEqualityProtocol` and `ProtosStandardObjectProtocol` supply existing-protocol canonical identity checks for standard equality and object methods; no standalone replacement language execution system was added.
6. **PE baseline updates:** `tools/java_generated_bytecode_bci_pe_baseline.json` and `tools/java_local_accessor_pe_baseline.json` reflect the added operations and local accessors.

The exact 17-path source publication includes:

- Production: `ProtosBytecodeRootNode.java`, `CanonicalCompilationFacts.java`, `CanonicalInlineStoragePlan.java`, `CanonicalToBytecodeLowerer.java`, `ProtosStandardNumberEqualityProtocol.java`, `ProtosStandardObjectProtocol.java`.
- Java regressions: `ProtosArrayEachNativeIterationTest.java`, new `ProtosPerf045RangeEachInlineTest.java`, `ProtosPerf025VirtualReturnHomeTest.java`.
- Protos conformance: `protos/tests/conformance/collections/array-iteration.protos`, `control/for-contextual-syntax.protos`, `library/collections/range-callbacks-and-control.protos`, `library/collections/range-prelude-and-module.protos`.
- PE baselines: the two JSON files named above.
- Publication metadata: `pom.xml`, `CHANGELOG.md`.

No normative `spec/` file is changed in this commit. The 17-path summary and version are verified from the published commit and its `pom.xml`/`CHANGELOG.md`.

## Validation provenance

**Maintainer attestation after push (2026-11-21):** “pushed, el git diff check esta limpio. todos los tests pasan en local.” Accordingly, the final local tests and `git diff --check` are **owner-reported PASS**. No complete exact-post-commit test transcript, Native Image result, CI run, or independent rerun was supplied; none is inferred.

Earlier human-supplied local evidence in this iteration included `make check-range-each-graph-capture` with 65/65 Python harness tests PASS and a pre-publication `array-iteration-for` guest `run` `After TruffleTier` graph of **Tier 1 = 608**, **Tier 2 = 497**, against historical GraalPy structural targets **378/271** (excess **+230/+226**). The source/capture SHA is not proven identical to the final published `a5e4d608`, and the user did **not** provide a new final exact-post-publication BGV pair. Therefore **do not** claim structural parity, Tier 1/Tier 2 node reduction, throughput improvement, or closure of PERF045 on this evidence.

Two local experimental variants were separately reported as **669/558** and **737/626**; they were deemed regressions against 608/497 and discarded. They are diagnostic history, **not** accepted results or measurements of the published product.

The conformance debugging also found that `Array.each` returns its Array receiver. An assertion helper ending in `Array(frozen, plain).each(...)` inadvertently returned an Array containing a mutable object, failing isolated logical Case result-copy. The helper was corrected to return `null` after its assertions. That **fixture correction** does not fix the independent [BUG039 Test Tool error-reporting defect](https://github.com/guillermomolina/protos/issues/1143), which can expose only a bare `INFRA` Error instead of the underlying transfer failure.

## Limits and follow-up

- **I115 #931:** retain **closed/completed**, with this post-closure publication recorded as additional implementation evidence; do not rewrite the 2026-11-20 measured 18-workload closure result as if measured at `a5e4d608`.
- **PERF045 #889:** retain **open/ready**. The published Range fast paths are a meaningful implementation checkpoint, but independent full-family (including Set/IdentitySet and other receivers) structural/causal and timing acceptance remains. A final root/tier graph at the exact product revision is not supplied.
- **BUG039 #1143:** retain **open/ready**, independently actionable diagnostic propagation defect, despite the corrected Range test case.
- Existing D013 dynamic call selection, lexical shadowing, callback contexts, arbitrary-precision values and standard per-collection result contracts remain normative; this record neither changes nor approves new semantics.

**Source of truth:** [product commit `a5e4d608`](https://github.com/guillermomolina/protos/commit/a5e4d6089a0764d4a55ebc033ac47cb2c7e0ccc2), [I115 live issue #931](https://github.com/guillermomolina/protos/issues/931), [PERF045 live issue #889](https://github.com/guillermomolina/protos/issues/889), [BUG039 #1143](https://github.com/guillermomolina/protos/issues/1143). Earlier independently accepted structural campaign: [I115 final evidence](I115_FINAL_CLOSURE_ALL_WORKLOADS_STRUCTURAL_CONVERGENCE_2026_11_20.md).
