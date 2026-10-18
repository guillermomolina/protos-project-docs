# I115 (0.3.382) — Tier-1 graph result, incomplete physical convergence, PLAT066-C research handoff

**Date:** 2026-10-18. **Implementation owner:** [I115 #931](https://github.com/guillermomolina/protos/issues/931). **Research origin:** [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930), within [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924). **Published product revision:** [`guillermomolina/protos@27cb31d20e517d8377e58963ea595e5ec16bfd15`](https://github.com/guillermomolina/protos/commit/27cb31d20e517d8377e58963ea595e5ec16bfd15), changelog `0.3.382-SNAPSHOT`. **Upstream comparison:** `oracle/graalpython@graal-25.4.4`.

## Outcome and state distinction

The product commit explicitly publishes two **implementation phases within the one I115 issue**: I115-A (source/runtime admission foundations) and I115-B (the adapted `InvokeClosureOneTail`, B5+ deferred caller and PROVEN direct transport). **Do not create another I115-B issue from the phase suffix.** The owner reports **I115-A closed/completed**.

**Measured improvement is real; the PLAT066-C objective of physically minimal GraalPy-shaped direct unary dispatch is not yet demonstrated/completed.** This is a remaining *physical convergence* failure/incomplete acceptance, **not** a claim of failing guest semantics, failing human-run tests, or a regression. The human reported green functional tests and supplied a successful focused graph-capture log. Do not close the originating PLAT066-C or represent the parent PLAT066 as complete based solely on this result.

The baseline [I114 Tier-1 BGV analysis](../I114/I114_380_TIER1_BGV_PATH_ATTRIBUTION_2026_10_18.md) records `guillermomolina/protos@fcc5c405413d70df546c5a9e5fa2c57b78c8ec91` and the I114 graph. The I115 capture was produced in the human's local implementation checkout before this evidence publication; the Git commit identifies the subsequently published implementation. Avoid falsely claiming independent byte-identical reproduction from the final published revision.

## Artifact provenance and directly observed results

Identical bare source for the TOP_LEVEL root, without a synthetic run wrapper:

```protos
identity: (value) => { value }
identity(1)
```

- **I114:** local user-supplied `TruffleHotSpotCompilation-3501[...perf042-bare-identity-program.protos|0+42]].bgv`, SHA-256 `53979e1181d69adce89c28526c252153ae5a948f78ed8cadc203f03e1d768187`. Tier 1, After TruffleTier **1,080** nodes; after low tier **3,505**; traced `InvokeClosureOneTail` inclusive source ancestry **429**.
- **I115:** user-supplied `TruffleHotSpotCompilation-3469[...perf042-bare-identity-program.protos|0+42]].bgv`, SHA-256 **`05e2d20433a29d70b8950ffedc3002b0c5cab8a67c677f8ed5c7cb3c86e61a82`**, exact TOP_LEVEL selector **`protos-root:886eb56b583af3af`**. BGV binary remains a user-local/conversation artifact and is **not** committed to this documentation repository.
- Human capture log: `PERF042_RUN_COMPILED_TIER=1`, `PERF042_RUN_BGV_COUNT=1`, `PERF042_RUN_CAPTURE=READY`, **`BUILD SUCCESS`** (1 focused test, 0 failures). `[engine] opt done` reports **`IR 707/2723`**, **`CodeSize 16696`**, `Inlined 0Y 1N`; 11 DEFAULT semantic Bytecode instructions, including the existing `c.InvokeClosureOneTail`, with zero trace/tag instructions.
- **Comparative graph totals:** After TruffleTier **1,080 → 707** (**−373; −34.5%**), final IR **3,505 → 2,723** (**−782; −22.3%**), machine code **19,899 → 16,696 bytes** (**−3,203; −16.1%**). These are whole-root comparisons; they do **not** assign all savings to one operation or guarantee equal compiler setup beyond the recorded Tier 1/source phase.
- Initial I115 BGV source-position inspection reports approximately **125 nodes with inclusive `InvokeClosureOneTail` ancestry**, versus I114's 429. This is an **ancestry classification, not 125 removable nodes** and not a controlled disjoint subtraction: reproduce both graphs with the same decoder and matching property rules before citing an exclusive/causal `429 → 125` reduction.

## What has changed successfully

1. Canonical `Object.call` target and Context/plan facts are established for a PIC entry under a stable root-call assumption, instead of rebuilding the generic ordinary-send decision inside the direct specialization.
2. The direct PROVEN route reaches `OptimizedDirectCallNode.call` / `OptimizedCallTarget.callBoundary`; there is no visible `PrepareSendArguments.exactCaller`, `prepareAndEnter`, generic `prepareClosureCallOneFallback`, or `isSuspended`/deferred C-prime transport within the measured PROVEN source ancestry.
3. B5+ preserves the five fixed header positions with slot 3 carrying published `ProtosActivation` or deferred caller frame-arguments reference. The captured graph has an `Object[6]` guest-call argument array; this allocation site is not by itself a regression, because the call remains non-inlined.

## Residual differences requiring *research*, not yet approved implementation

These are physically observed source paths plus specific replacement **candidates**. They are **not proof** that the proposed removals preserve all semantics, nor a request to change D013.

| Candidate | Remaining Protos path | GraalPy-shaped contrast | Investigation / proof obligation |
| --- | --- | --- | --- |
| R1 — PIC hit re-admission | `InvokeClosureOneTail.guardedSource` guards `admittedCachedInstance(...)`, which tests local `call` presence, unobservable ReturnHome, prepared plan identity and Context projection; the BGV includes `ProtosLexicalBindingAuthorityCalls.contains` on a conditional branch | `CallUnaryMethodNode.callPFunction` / `FunctionCachedInvokeNode` rely on cached function/root identity and code-stability Assumptions before direct invocation | Can the exact mutable-instance facts be guaranteed through coherent invalidation/shape or guard precomputation, **including a local `call` written during argument evaluation**, without moving costs elsewhere? Do not simply delete semantic guards. |
| R2 — caller / B5 header representation | `callerReference(frame)` selects published Activation or its B5 array; constructor/entry validates representation and builds six-slot array | GraalPy `PArguments.create(1)`, `setArgument`, then direct invocation | Which representation validations are provably redundant at the operation boundary? Preserve exact caller identity, cycle rejection, error/Task/debug handling and the universal B5+ layout. No obligation to remove the array crossing `callBoundary`. |
| R3 — result projection | `enterDirectProven` calls `ProtosNumericValueSupport.guestValue(outcome)`; `Long`/`Double` branches and guest numeric materialization remain in source ancestry | GraalPy returns the direct invoke outcome and uses its own value conventions | Identify the precise consumer/representation contract that could avoid conversion for this terminal path without leaking host carrier values, changing numeric identity, or shifting conversion into another DSL operation. |

**Normal non-inlined `OptimizedCallTarget.callBoundary`, frame states and compiler deoptimization guards are not automatically removable overhead.** A `MethodCallTargetNode` that is structurally reachable does not demonstrate that the ordinary direct-hit input executes it. Report conservative graph reachability separately from feasible hot paths.

The original approved owner boundary **leaves D013 unchanged**. An earlier assistant proposed this deferral; it is not evidence that D013 is physically optimal or a demonstrated cause of the residue. Study R1–R3 without assuming a D013 amendment is necessary; if the only correct solution requires a semantic change, identify the specific counterexample and obtain explicit approval **before** implementing it.

## Required next PLAT066-C investigation

**TIPO = INVESTIGACIÓN; does not execute.** Re-read GitHub's current `guillermomolina/protos` HEAD, #930/#931/#924 and source; the local published implementation is at `27cb31d` when this record was created. Study GraalPy `graal-25.4.4` exact `CallUnaryMethodNode.callPFunction`, `CallDispatchers.FunctionCachedInvokeNode`, `FunctionDirectInvokeNode`, `SimpleDirectInvokeNode`, `PArguments` and caller-preparation path. Use both user-supplied BGV artifacts if accessible; do not invent a newer or matched GraalPy BGV. Produce disjoint node tables, executed/alternative branch classification, and **source-precise before/after mechanisms** for R1–R3 with mutation, Context, ReturnHome, Task, suspended/UNKNOWN and numeric semantics proofs. Remove overhead from the operation's compiled physical path; **do not add an `@Operation`, modify bodies of other DSL operations, or relocate work and call it a win**.

Exit of investigation is a decision-ready plan, not an implemented result; an independent next **I-family** implementation item may be proposed only after the remaining changes have been specified and approved. Product changes and test execution are explicitly out of scope.

## Verification boundaries

`PROTOS_REVISION=27cb31d20e517d8377e58963ea595e5ec16bfd15` (published implementation); `PROJECT_RECORD_REVISION` is the commit that publishes **this** record, to be linked from the live Issues. User performs builds/tests and product Git. This record is a non-normative evidence snapshot and does not change an approved language decision.
