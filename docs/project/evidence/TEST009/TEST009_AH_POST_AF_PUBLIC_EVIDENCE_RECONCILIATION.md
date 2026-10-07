# TEST009-AH — post-AF public-evidence reconciliation and executable next-slice routing

Status: **COMPLETE — NO NEW POST-AF OWNER ATTRIBUTION; NEXT SLICE IS GUARDED EXECUTABLE IMPLEMENTATION**

Formal work: `guillermomolina/protos#795`

Date: **2026-10-07**

## Scope

TEST009-AH is the read-only investigation released by TEST009-AG after the
post-AF residual `NoSuchElementException.<init>` aggregate remained measured
but not attributed to one exact current Protos owner/callsite.

AH was required to consume only public evidence and to execute no commands,
diagnostics, repository mutations, commits or pushes while performing the
investigation.

The investigated product and durable-record state was:

```text
EXAMINED_PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
EXAMINED_PROTOS_VERSION=0.3.263-SNAPSHOT
EXAMINED_PROTOS_COMMIT=TEST009-AF: keep context projection failure out of PE

AG_RECORD_REVISION=bf802aeda24c7939b26514b0a51dc205c710393f
AG_RECORD_PATH=docs/project/evidence/TEST009/TEST009_AG_POST_AF_CAUSAL_OWNER_SELECTION.md

AH_PROJECT_RECORD_BASE_REVISION=8db21433ff708611b34464036f3e56d43e51216b
```

The project-record repository had advanced after AG only through unrelated
D188/I080 coordination work. That movement does not alter TEST009's product
revision, fixed root, or post-AF causal evidence.

## Fixed root and current measured residual

AH keeps the established single-root identity unchanged:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
ROOT_CHANGE=NO
```

The latest published same-root result remains:

```text
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE

Throwable.fillInStackTrace
  frames=96
  cumulative=6864

NoSuchElementException.<init>
  frames=66
  cumulative=4884
```

Therefore:

```text
NSEE_AGGREGATE_IS_MEASURED_CURRENT=YES
NSEE_EXACT_CURRENT_OWNER_ATTRIBUTION=NO
```

## Public-evidence reconciliation

At AH completion:

```text
PROTOS_HEAD_ADVANCED_SINCE_AG=NO
POST_AG_TEST009_PRODUCT_CHANGE=NONE
POST_AG_TEST009_DURABLE_RECORD=NONE
POST_AG_ISSUE_DIAGNOSTIC_COMMENT=NONE
```

The live Issue contains no comment after the TEST009-AG completion entry that
publishes a fresh post-AF `TraceMethodExpansion`, `TraceNodeExpansion`,
`TraceInlining`, BGV-derived decomposition, or equivalent exact attribution of
the 66 current NSEE constructor frames.

No current public evidence therefore improves AG's causal-attribution gate.

## Current-source reconciliation

The candidate source shapes inspected by AG remain present at the examined
product revision:

- `ProtosPrelude.arrayPrototype()` still performs lazy
  `readLocalSlot("Array").orElseThrow()`; eager retention would still move
  validation timing.
- `ProtosPrelude.standardErrorPrototype(String)` still performs dynamic named
  lookup and Error-hierarchy validation with real reachable failure semantics.
- `ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState(...)` still
  performs `caller.task().isPresent()` followed by a second
  `caller.task().orElseThrow()` observation.
- `ProtosActivation.forObjectConstruction(...)` independently retains the
  corresponding `enclosing.task()` check-then-reread.
- `PreparedBooleanCall.hasCallback()` still operates over constructor-validated
  final enum state with an exhaustive source switch, but there is no exact
  current generated-switch attribution.
- `finishPreparingComposedCall(...)` still checks
  `closure.nativeBody().isPresent()` and then rereads
  `closure.nativeBody().orElseThrow()`; `ProtosClosureValue.nativeBody` is
  final, so the source-level PRESENT-then-EMPTY failure remains impossible for
  one Closure.
- the same broad composed-call helper also contains
  `plan.language().orElseThrow()`, so a method-level historical attribution
  would still be insufficient to identify the exact NSEE callsite.
- `LOCAL_FRAME` remains a historical multi-owner aggregate rather than one
  current owner/mechanism.

The strongest source-level candidate therefore remains the
`finishPreparingComposedCall -> nativeBody().orElseThrow()` redundant Optional
reread, but TEST009 does not permit source plausibility to replace current
same-root causal attribution.

## Gate result

No candidate clears all mandatory release gates because every candidate still
fails:

```text
CURRENT_CAUSAL_ATTRIBUTION=NO
```

Some candidates also retain owner-specific blockers:

```text
arrayPrototype
  -> validation timing must remain lazy

standardErrorPrototype
  -> real dynamic validation must remain semantically reachable

attachTaskOrInheritDynamicControlState
forObjectConstruction Task unwrap
  -> publication/concurrency/observability proof still required

PreparedBooleanCall
  -> exact generated constructor/switch/host owner still required
```

Accordingly:

```text
TEST009_AH_RESULT=SELECT_NONE_NO_NEW_PUBLIC_POST_AF_DECOMPOSITION
SELECTED_OWNER=NONE
SELECTED_MECHANISM=NONE
SELECTED_REPAIR=NONE
IMPLEMENTATION_READY_FROM_PUBLIC_EVIDENCE=NO
TEST009_COMPLETE=NO
```

## Missing acquisition

The missing evidence is not another source-only investigation. It is a fresh
same-root dynamic acquisition against the current product revision that
decomposes the residual:

```text
NoSuchElementException.<init>
  frames=66
  cumulative=4884
```

to the nearest exact Protos owner/callsite.

The current product repository already contains the bounded TEST009 diagnostic
surface:

```text
tools/truffle_root_diagnostic.py

make truffle-root-catalog
make diagnose-truffle-root

TRUFFLE_ROOT_SELECTOR=protos-root:088d2ae81075aba8
TRUFFLE_ROOT_EXPANSION=method|node|none
TRUFFLE_ROOT_DUMP_LEVEL=1|2
```

The diagnostic uses `engine.CompileOnly=<selector>`, synchronous immediate
compilation, `CompilationFailureAction=Print`, one expansion view at a time,
and `Dump=Truffle:<level>` under `target/truffle-compilation/`.

For the next acquisition the relevant expansion views are:

```text
TraceMethodExpansion=truffleTier
TraceNodeExpansion=truffleTier
TraceInlining only if needed to disambiguate the exact current callsite
```

## Slice-granularity correction

The project owner explicitly clarified that TEST009 must not create a chain of
source-only investigation slices when the remaining investigation can be
performed as ordered gates inside one executable slice.

This does **not** relax or skip the TEST009 methodology. Instead, the next slice
must perform the full sequence internally:

```text
fresh fixed-root dynamic acquisition
  ->
exact residual NSEE decomposition
  ->
select exactly ONE owner/callsite or NONE
  ->
identify exact mechanism
  ->
prove semantic/timing/API/observability gates
  ->
identify one bounded repair
  ->
if and only if every gate passes, implement that one repair
  ->
run same-root causal A/B and required tests
```

If the dynamic acquisition does not yield one owner that clears every gate, the
slice must stop with no product edit. It must not guess from historical W sizes
or choose the most attractive source pattern.

Under the project's two requested handoff categories, this is an
**IMPLEMENTATION** slice because it is executable and may conditionally edit the
product after its acquisition gates pass. It is not another no-command
investigation slice.

## AI handoff

```text
NEXT_SLICE=TEST009-AI
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

NEXT_SCOPE=acquire a fresh same-root post-AF method/node expansion for protos-root:088d2ae81075aba8, decompose the current NoSuchElementException.<init> residual to one exact owner/callsite, apply every TEST009 release gate, and only if exactly one owner passes, implement one bounded repair and prove it with same-root causal A/B; otherwise stop without edits

MULTI_OWNER_BATCH=NO
ROOT_CHANGE=NO
AUTOMATIC_BOUNDARY_SEARCH_AS_DESIGN_AUTHORITY=REJECTED
MICRO_REPAIR_BATCH=REJECTED
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
using the current public Protos HEAD, live TEST009/#795 work log, current
TEST009 durable records, current product source, repository governance, and the
project owner's explicit clarification on slice granularity. No independent
human review is claimed.
