# I101-B — Published minimal NORMAL arithmetic and scalar-local Bytecode lowering

**Evidence date:** 2026-10-14  
**Owning work:** [I101 / #892](https://github.com/guillermomolina/protos/issues/892) — **slice B PUBLISHED; parent remains OPEN/READY for slice C**.  
**Governing decision:** [PLAT058-D / #891](https://github.com/guillermomolina/protos/issues/891); no new semantic or architectural ratification is claimed.  
**PROTOS_REVISION:** `df4142919d0222635ab8f9f8c54e7a1966e6edaa`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/df4142919d0222635ab8f9f8c54e7a1966e6edaa  
**Product version:** `0.3.357-SNAPSHOT`  
**Evidence class:** independently verified GitHub product commit and source/test contents; human-reported `make check`, `make test`, and `git diff --check` PASS. No tests or performance measurements independently executed by this coordinating agent.

## Published product result

The product `main` commit **"I101: implement minimal NORMAL arithmetic bytecode"** is a single squash publication of the I101 experimental Builder/graph checkpoint and the final validated corrections. It follows the existing I093 product state and includes the matching `pom.xml` and `CHANGELOG.md` version `0.3.357-SNAPSHOT`.

In admitted NORMAL parameterless scalar-only Closure roots, the canonical lowerer emits direct Bytecode DSL `LoadConstant`, `StoreLocal`, `LoadLocal`, and `Return`, omitting `IsCompactLocalFrame`, `CreateCurrentFrameLocal`, generic activation dispatch, and `MaterializeReturnCarrier` from that physical body. Lexical aliases retain their actual `LoadLocal` instruction; the lowerer does **not** globally replace alias loads or eliminate locals. Subsequent constant propagation and unused-store elimination are delegated to Graal.

Proven exact signed-64 Integer additions are folded before Graal, including canonical literal and established-local arithmetic when the existing conservative proof supports the exact result. Dynamic operands, overflow, custom dispatch, observable contexts, capture, and other non-admitted forms retain their necessary general semantics. This record does not claim arbitrary arithmetic algebra, a new universal effect/value analysis, or cross-Prelude folding outside proved conditions.

The publishing change also retains the immutable Context-owned NORMAL/DEBUG profile, straight-line/non-suspension proof updates for folded additions, and profile-specific tests. DEBUG is the profile that enables debugger breakpoints, source stepping, external pre-entry activation observation, and lexical authority. NORMAL still preserves explicit **guest-language** `context` observation, without emitting debugger machinery for an unobservable scalar-only body. A Java test artificially observing a rich activation before entry is explicitly exercised under DEBUG rather than imposing a hidden NORMAL branch.

## Exact independent Builder-oracle and regression evidence

The published test sources include two independent handwritten `ProtosSemanticBytecodeRootNodeGen.Builder` comparisons. The maintainer supplied the following passing focused output for the checked work:

| Source case / oracle | Physical instructions | Result |
|---|---:|---|
| Pure folded literal addition | 12 | `I101_B_LITERAL_ADD_BYTECODE_EQUAL=true`, semantics and oracle PASS |
| One local creation + read | 16 | `I101_B_CREATE_AND_READ_BYTECODE_EQUAL=true`, result 1, PASS |
| One local creation, creation-result semantics | 16 | `I101_B_CREATE_ONLY_BYTECODE_EQUAL=true`, result 1, PASS |
| Proven local + literal | 20 | `I101_B_LOCAL_PLUS_LITERAL_BYTECODE_EQUAL=true`, semantics and oracle PASS |
| Proven local + local | 24 | `I101_B_LOCAL_PLUS_LOCAL_BYTECODE_EQUAL=true`, semantics and oracle PASS |

The oracles compare complete emitted physical instruction/constant signatures, rather than a selected filtered subset; root-group/topology assertions and semantic outcome checks are present in the product fixtures. The focused PERF025/PERF032 regression run passed after removing outdated assumptions that `1 + 2` must be dynamically dispatched, restoring a real parameter-dependent dynamic-send negative control and ensuring DEBUG-specific breakpoint/observability cases use DEBUG.

The published product contains the independent-oracle tests, relevant PERF025/PERF032 regression updates, and opt-in I101 Graal compilation/graph diagnostic utilities; these utilities are not proof of a stable Tier-1/Tier-2 result or measured speedup.

## Human-executor validation and publication

The maintainer explicitly reported:

- Focal arithmetic/local Builder equality cases **PASS**, with the exact physical signatures and counts above.
- Focused PERF025 and PERF032 regressions **PASS** following test-correctness updates.
- Final `make check` **PASS**, `make test` **PASS**, and `git diff --check` clean on the substantive implementation.
- The squash commit was pushed to product `main` and `git rev-parse HEAD` reported `df4142919d0222635ab8f9f8c54e7a1966e6edaa`.

The coordinating agent independently read back the published product commit on `main` and its `0.3.357-SNAPSHOT` Maven/changelog, but **did not** run `make`, Maven, native binaries, compiler graph captures or benchmarks. The final suite's exact case count was not supplied as a confirmed green summary and is not inferred from earlier failing runs.

## Remaining I101-C work and limits

I101 **remains OPEN/READY** for C: comprehensive cross-family NORMAL/DEBUG, reparsing and root topology, Context/Task, captured lexical authority, ReturnHome, source/debug and other admissibility/fallback acceptance beyond the bounded I101-B tests. Matched compiled graphs and steady performance measurements may be pursued as **secondary** evidence, not as replacements for minimum physical Builder-oracle correctness. The PLAT059 owner-approved no-new-architecture decision remains in effect.

No normative Protos specification modification, universal optimizer architecture, Native Image acceptance, proven Tier convergence, measured GraalJS/GraalPy parity, or numeric speedup is claimed.

## Revision-bound references

- [Published product commit `df4142919d0222635ab8f9f8c54e7a1966e6edaa`](https://github.com/guillermomolina/protos/commit/df4142919d0222635ab8f9f8c54e7a1966e6edaa)
- [Published 0.3.357 implementation changelog](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/CHANGELOG.md)
- [I101-B arithmetic Builder oracle](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/test/java/com/guillermomolina/protos/execution/ProtosI101BArithmeticBuilderOracleTest.java)
- [I101-B scalar-local Builder oracle](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/test/java/com/guillermomolina/protos/execution/ProtosI101BLocalBindingBuilderOracleTest.java)
- [NORMAL/DEBUG regression suite](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/test/java/com/guillermomolina/protos/execution/ProtosPerf032ExecutionModeTest.java)
- [I101-A historical evidence](I101_A_353_CONTEXT_PROFILE_MINIMAL_BOOLEAN_FOLD_PUBLISHED_2026_10_14.md)
- [Owning GitHub issue #892](https://github.com/guillermomolina/protos/issues/892)
