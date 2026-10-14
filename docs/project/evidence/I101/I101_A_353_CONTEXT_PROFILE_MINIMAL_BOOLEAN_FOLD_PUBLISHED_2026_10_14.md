# I101-A — Published Context-owned compilation profiles and minimal Boolean-control folding

**Evidence date:** 2026-10-14  
**Owning formal work:** [I101 / #892](https://github.com/guillermomolina/protos/issues/892) — slice A only; I101 remains OPEN for B and C.  
**Governing ratification:** [PLAT058-D / #891](https://github.com/guillermomolina/protos/issues/891), [ratified architecture](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md).  
**PROTOS_REVISION:** `97404f1cc1c1d678c66ef598b47cf62d32014f21`  
**Product version:** `0.3.353-SNAPSHOT`  
**Verified product publication:** https://github.com/guillermomolina/protos/commit/97404f1cc1c1d678c66ef598b47cf62d32014f21  
**Evidence class:** independently verified GitHub publication and source/test presence; maintainer-reported local tests and whitespace validation. No independently executed tests or measured performance claim.

## Published implementation

The product commit is **"I101: Context-owned NORMAL/DEBUG lowering profile and minimal constant Boolean-control fold (0.3.353)"**.

- `ProtosExecutionMode` serves as the typed immutable physical lowering profile. Context-owned top-level parsing (`ProtosLanguage` / `ProtosSourceCompiler`), module initialization (`ProtosModuleRuntime`) and destination-owned cross-Context Closure rebuilds (`ProtosLanguageContext`, `ProtosClosureExecutionPlan` and `ProtosBytecodeClosureExecutionPlan`) explicitly pass the appropriate profile to `CanonicalToBytecodeLowerer`. The lowerer does not consult an ambient entered Context for the mode.
- The runtime `IsNormalExecutionMode` Bytecode DSL operation is removed. An admitted NORMAL I095 Boolean control emits its chosen physical lowering without a DEBUG branch. Dynamic receivers retain the canonical-Boolean classifier and ordinary-dispatch fallback. NORMAL I096 inline calls retain the runtime guard and fallback for ordinary `Object.call` selection; DEBUG retains its ordinary semantic send/callee activation.
- The lowerer snapshots standard `Object` publication at construction, instead of re-reading `isFrozen()` during reparses, to keep Boolean-control and related admission decisions stable.
- A proven terminal Boolean-control expression with a literal Boolean receiver, literal Closure arguments, an exact selected callback result, and an admitted pure-literal inline body emits the literal in place of the send. The canonical tree/source identity remains intact. For `() => { true.ifTrue() { 1 } }` in NORMAL, the emitted executable activation-root instructions match `() => { 1 }` and the handwritten literal-one Builder reference. The effective proven body also qualifies for the existing non-suspending and headerless-entry capabilities, not a new shortcut rule.
- The same commit restricts captured frame-local fast paths to an open owner root. This avoids claiming an unavailable owner frame when an I096 inline copy first lowers an omitted capturing callback.
- `pom.xml` and `CHANGELOG.md` publish `0.3.353-SNAPSHOT`; the change adds `ProtosI101ProfileSpecializedLoweringTest.java` and adapts affected I092/I095/I096, grouping, compilation and structural-conformance tests. The historical bytecode BCI/PE baseline fixture is updated.

This is a physical-lowering implementation under already approved semantics, **not** a language or normative-specification change.

## Structural and behavioral evidence in the published test source

The new `ProtosI101ProfileSpecializedLoweringTest` contains:

- A three-way structural check of **folded source root = direct literal source root = handwritten Builder oracle**. Complete instruction signatures include typed constant operands, not selected or filtered instructions.
- Independent NORMAL/DEBUG root-group topology and arity-rejection target expectations, plus repeated `ensureComplete()`/reparse checks of root and CallTarget identity and instruction stability.
- Separate NORMAL and DEBUG Contexts under one Engine; destination-profile handling for cross-Context Closure projection and hosted module imports.
- Admission boundaries, omitted physical inline callback roots, and semantic outcome checks for folded and non-folded programs, including conservative capability flags outside the proven fold.

These are checks present in the published Java test source. Their **successful execution is attested by the maintainer**, not independently reproduced by this coordinating agent. The publication-transition-before/after-`Object`-freeze case is not independently exercised as a single in-process transition by the focal fixture; a pre-publication lowerer remains conservative by construction.

## Human-executor validation and provenance

The maintainer explicitly reported on 2026-10-14:

- `I101: Context-owned NORMAL/DEBUG lowering profile and minimal constant Boolean-control fold (0.3.353)` was committed and **pushed**.
- `git diff --check` was **clean**.
- **All local tests passed**.

The coordinating agent independently verified the published commit SHA, version/changelog, changed production and test paths, and new I101 fixture on GitHub. It did **not** run Maven, `make test`, a Graal compilation, a native build or benchmarks, nor receive exact test counts/durations or the `I101_A_*_SIGNATURE` output. No JIT speedup, cross-language parity, general constant folding, or general-purpose Closure-call elimination is claimed.

## Remaining work owned by I101

**I101 remains OPEN; only A is published.**

- **B — broader proof-backed minimal physical emission:** extend justified cases across canonical Boolean selector families, eligible literal/binding Closure calls and simple pure arithmetic where semantically proven. Each *new physical-elimination claim* needs its own independent handwritten Builder-oracle structural comparison, correct guarded fallback, and negative semantic witnesses. Do not infer an unsafe fold from the I101-A literal example.
- **C — cross-family acceptance:** reconcile NORMAL/DEBUG instruction and group shapes, source/debug scopes, reparse, Context/Task, captures, ReturnHome and continuation behavior; use compiled graph and performance observations only as secondary evidence rather than as replacement for bytecode/oracle acceptance.

No separate child Issue is opened for these internal slices; live status, priority and scheduling remain owned by [I101 / #892](https://github.com/guillermomolina/protos/issues/892).

## Revision-bound references

- [Exact product commit `97404f1cc1c1`](https://github.com/guillermomolina/protos/commit/97404f1cc1c1d678c66ef598b47cf62d32014f21)
- [Published I101 focal Java fixture](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/src/test/java/com/guillermomolina/protos/execution/ProtosI101ProfileSpecializedLoweringTest.java)
- [Published I101 changelog](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/CHANGELOG.md)
- [I101 owning issue](https://github.com/guillermomolina/protos/issues/892)
- [PLAT058-D decision](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md)
