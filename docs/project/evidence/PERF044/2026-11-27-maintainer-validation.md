# PERF044 — Guarded matching optimization, compiler alignment and publication

Date: 2026-11-27

Owning issue: [PERF044 / #883](https://github.com/guillermomolina/protos/issues/883).

## Published implementation

- PROTOS_REVISION: [948fa4c16bba3ef0c5096a06a7a3a73526bbebed](https://github.com/guillermomolina/protos/commit/948fa4c16bba3ef0c5096a06a7a3a73526bbebed).
- Implementation version: 0.4.249-SNAPSHOT, confirmed in the [changelog](https://github.com/guillermomolina/protos/blob/948fa4c16bba3ef0c5096a06a7a3a73526bbebed/CHANGELOG.md).
- Maintainer confirmed PERF044 integrated into main, clean git diff --check and all local tests passing. The published history replays the rebased source commits with implementation-version metadata; the pre-publication rebased HEAD was 02348764485eb5a4ef258e7c6fba0234d88fa74f.
- The literal subject-comparison/last-use fusion experiment was reverted; it is not part of the retained optimization.
- The final integration repair uses separate constant LocalAccessor destinations for arithmetic completion and suspension, reconciles the PE inventories and adapts instruction-level tests to the actual fused lexical lookup and comparison-result cleanup.

## Retained mechanism and correctness scope

Eligible standard caseOf Map-literal sites lower to guarded selection in the enclosing bytecode root. Standard Integer literal comparisons and admitted actions can execute directly, without ordinary message sends in the admitted primitive matching path. The retained work also specializes standard Map construction/hash/equality, selected method invocation, numeric dispatch and lexical lookup. Dynamic/custom matching, caseOf overrides, key evaluation/order/duplicate policy, capture and action behavior, errors, suspension and return homes remain governed by the ordinary matching protocol. Contextual match is I176 source sugar, not a second runtime matching authority. No public matching semantics change is claimed for PERF044.

The published change includes conformance matching cases and regression coverage for matching inlining, Map construction, custom protocols, numeric operations, lexical selection and suspension. Relevant source/test anchors are visible in the exact Protos revision and its changelog; no blanket claim that every dynamic matching shape achieves primitive-path performance is made.

## Historical measurement evidence (not a post-publication run)

Published original harness/reference: [2b002766f26264144056a87c5454deea32bb7274 / PERF044](https://github.com/guillermomolina/protos-benchmarks/tree/2b002766f26264144056a87c5454deea32bb7274/results/reference/protos-d2bd4bed4527ef4856841c95b49f20b068d62d55/perf044).

The retained local Protos measurement snapshot was a2038fc69368b051846b20ce656deb8bcfcbaca1 (0.4.227-SNAPSHOT); its local evidence directory was results/local/perf044-compact-lexical-identity-20261127-032848. All five workload correctness checks passed. The following counts are nodes after TruffleTier in the selected guest root, not elapsed times.

| Workload | Initial Protos DSL operations/execution | Retained Protos DSL operations/execution | Historical GraalPy DSL operations/execution | Retained Protos T1/T2 nodes | Historical GraalPy T1/T2 nodes |
|---|---:|---:|---:|---:|---:|
| primitive-switch-first | 241 | 27 | 18 | 111/54 | 107/50 |
| primitive-switch-middle | 284 | 29 | 24 | 111/54 | 107/50 |
| primitive-switch-last | 327 | 31 | 30 | 111/54 | 107/50 |
| primitive-switch-default | 359 | 31 | 30 | 111/54 | 107/50 |
| integer-switch | 370781 | 42518 | 35515 | 232/123 | unavailable (ambiguous)/136 |

The first primitive-switch initial guest graph measured 7334/7448 nodes (T1/T2); the retained snapshot measured 111/54. These are historical structural measurements, not a claim of equivalent reductions in execution time.

## Integer-switch compiler-version investigation

The original GraalPy reference used GraalVM 25.4.4.1.1 (compiler 95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b). Protos used GraalVM 25.5.5.0.3, compiler e3cb74fceaf6ee4f50366d5ab2bbf602e464475d. The benchmark toolchain alignment is published separately at [9ca4bfeeb44b67fbc00d1c32715d6295f0e94758](https://github.com/guillermomolina/protos-benchmarks/commit/9ca4bfeeb44b67fbc00d1c32715d6295f0e94758). Maintainer output reported 314 harness tests passing, 26 correctness workloads each for GraalPy and GraalJS passing, and identical compiler/runtime source identities for the Protos and updated peer container. TruffleRuby retains its independent toolchain.

Aligned local capture: results/local/perf044-aligned-integer-switch-20261127-053321/tier-2/integer-switch/python (GraalPy 25.5.5.0.3; Tier 2, all roots, 2000 invocations, first/last thresholds 100/1000, CPU 0; capture valid and analysis successful). The existing Protos capture was reused rather than rerun.

| Guest Tier 2 phase | GraalPy 25.4 | GraalPy 25.5 | Protos 25.5 |
|---|---:|---:|---:|
| After TruffleTier | 136 | 136 | 123 |
| Before HighTierLowering | 145 | 103 | 109 |
| After high tier | 161 | 119 | 121 |
| After mid tier | 217 | 241 | 258 |
| After low tier | 253 | 328 | 345 |

Both updated graphs have two loops, six loop ends and seven mid-tier guest safepoints. The original LowTier difference of 92 nodes shrank to 17 (+5.18% against updated GraalPy); 75 of the original 92 nodes are accounted for by the changed peer toolchain/language combination, not a further Protos change. Both exact compiler revisions were inspected: the newer MidTier introduces an aggressive partial-unroll phase before safepoint insertion, unlike the older ordering. The comparison does not isolate a compiler-only intervention from a simultaneous peer-language version change. No compiler options or safepoints were disabled. The host adapter LowTier measured 568 Protos vs 599 updated GraalPy nodes.

A short pre-alignment JFR timing run recorded steady minima of 22250 ns (Protos) and 21530 ns (GraalPy); the recording had only three and two identifiable guest execution samples, respectively. Neither timing significance nor allocation/hotspot conclusions are established. The owner explicitly decided not to pursue the remaining 17 nodes and to close the work as retained.

## Final integration evidence

Maintainer-supplied pre-publication logs showed:

- LocalAccessor guard: 132 sinks, 83 PE-reachable receiver and BytecodeNode proofs, zero unresolved receivers, PE risks or unknown reachability after correction; topology reconciled.
- Generated BCI guard: 538 transitions, 536 proven, two recognized dynamic exception-table transitions, zero risks/unknowns; topology exact.
- Real synchronous Truffle BCI check: 51 compilations (43 bytecode roots), zero permanent or BCI/PE compilation failures; PASS.
- Native runtime compilation gate self-tests: 97/97 PASS; full make check reported PASS.
- First integrated Java parallel suite: 7159 tests, 3 assertion failures, 0 errors, 11 skipped. The three failures were reconciled in the two test classes without deleting their semantic or cleanup checks. Do not present that initial run as green.
- Final owner report after integration: git diff --check clean; all local tests PASS. The separate final full-suite transcripts and exact final test totals were not supplied.

No new benchmark was requested after the integration repairs and version replay. Raw final local measurement directories are not claimed to have been published; the linked historical reference and harness revisions are public, while the numbers from later local captures are retained here as maintainer-supplied evidence. No final-revision speedup, whole-language parity, release publication or independent CI PASS is claimed.

## Closure boundary

The owner directed completion of the bounded PERF044 optimization on the retained implementation, with the remaining structural difference accepted. The initial wider research questions remain useful historical context; this closure does not certify every proposed workload or universal performance parity. D263/I176 remain independent source-language work and no other work item is closed by this record.
