# I072 Phase C — execution-context and argument materialization evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase C — execution-context and argument materialization
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=795c1ea75236e028a42bbfb91f8c26f8d12c73e4
PROTOS_REVISION=045bfdb8ec3ccaf09be8b35a2053c124a9edebb8
PROTOS_VERSION=0.3.92-SNAPSHOT
COMMIT_MESSAGE=I072-C: defer guest invocation materialization
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PLAT040_COMPARATIVE_EVIDENCE=docs/project/evidence/PLAT040/PLAT040_CROSS_TRUFFLE_HOT_CALL_DECISION_EVIDENCE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_A_EVIDENCE=docs/project/evidence/I072/I072_PHASE_A_GUARDED_SELECTED_SEND_IMPLEMENTATION.md@4639bab0e9175781ca0c952eef30e909ee55644f
PHASE_B_EVIDENCE=docs/project/evidence/I072/I072_PHASE_B_COMPACT_ORDINARY_CALL_ABI_IMPLEMENTATION.md@b15f43708708fee781415760ba7bff731f4b86cf
PHASE_C_STATE=COMPLETE
I072_STATE=OPEN

## Result

I072 Phase C publishes the conditional guest-state materialization layer of the
ratified PLAT040 Candidate F-prime architecture.

The Phase A selector-stability substrate and Phase B compact target-entry ABI
remain intact. Ordinary source-backed invocation now carries supplied arguments
as an internal positional vector and executes current lexical create/read/assign
operations against the single frame-backed lexical authority without eagerly
constructing either the guest execution-context object or the guest supplied
argument Array.

A genuine guest execution context is created only when semantics actually
observe or require it, including explicit `context` observation, lexical
capture/escape and debugger/reflection projection. Before such observation, the
frame-backed authority remains the single lexical store. When the guest context
becomes observable, the authority first makes its frame escape-safe and is then
installed into the fresh `ProtosExecutionContextValue`, preserving identity and
binding authority.

Supplied guest Array creation is similarly deferred. Parameter/default binding
reads the internal positional vector directly. Rest binding continues to create
the semantically required fresh frozen guest Array.

The published structural outcome is:

```text
PHASE_A_GUARDED_SELECTION_PRESERVED=YES
PHASE_B_COMPACT_TARGET_ENTRY_PRESERVED=YES
PRE_TARGET_RICH_CALLEE_ACTIVATION=NO

ORDINARY_EXECUTION_REQUIRES_EAGER_GUEST_CONTEXT=NO
CONTEXT_MATERIALIZATION_CONDITIONAL=YES
INSTALL_REQUIRES_EAGER_FRAME_MATERIALIZE=NO

ORDINARY_INTERNAL_ARGUMENT_TRANSPORT_REQUIRES_GUEST_ARRAY=NO
SUPPLIED_VECTOR_INTERNAL_GUEST_ARRAY_REMOVED=YES
REST_ARRAY_REQUIRED=YES

PLAT036_I068_COMPATIBILITY=PASS
D179_COMPATIBILITY=PASS
REFLECTION_DEBUGGER_PROJECTION=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

FOCAL_VALIDATION=PASS
EXECUTION_CONTEXT_CONFORMANCE=PASS
STRUCTURAL_HOT_PATH_DIAGNOSTIC=PASS
FINAL_REQUIRED_VALIDATION=PASS
PUBLICATION=PASS
```

## Published implementation

The product revision changes:

```text
src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalFallback.java
pom.xml
CHANGELOG.md
```

### Deferred supplied-argument representation

The compact-invocation activation now owns a `DeferredSuppliedArguments`
carrier containing a copied internal `List` and a lazily created guest Array.

`suppliedArgumentsForRuntime()` exposes the internal vector to the execution
backend without materializing the guest Array. Bytecode parameter presence,
positional loads, upper-bound checks, default binding, and the direct Java
closure execution plan use that vector.

The existing public/runtime `arguments()` projection remains available and
materializes one stable guest Array only when that representation is actually
observed.

Rest binding intentionally remains different: `BindClosureRest` constructs the
fresh frozen guest Array required by Protos semantics from the applicable suffix
of the internal supplied vector.

### Deferred execution-context materialization

Compact invocation no longer calls `prelude.newExecutionContext()` at
activation construction.

The current activation retains a deferred frame lexical authority. Current local
membership, reads, creation, and assignment can operate directly against that
authority. The lowering therefore no longer loads the `context` intrinsic just
to create an ordinary current local, and unqualified writable-target resolution
can retain a current-context marker instead of materializing the guest object
before RHS evaluation.

`ProtosActivation.context()` is the explicit materialization seam. On first
observation it:

```text
prepares the deferred lexical authority for guest observation
creates the fresh ProtosExecutionContextValue
installs the same authority
retains the resulting guest object identity
```

Closure capture calls this seam deliberately so escaped lexical state owns an
escape-safe materialized context object.

### Conditional frame materialization

`ProtosFrameLexicalBindingAuthority` no longer receives a materialized frame
during ordinary root installation.

The new backend-neutral
`ProtosLexicalBindingAuthority.prepareForContextObservation()` hook is a no-op
for authorities that need no preparation. The frame-backed implementation uses
it to replace its live frame reference with `frame.materialize()` only when the
guest execution context is about to become observable.

Therefore ordinary execution does not pay frame materialization merely because
the root established frame-backed lexical authority.

### Repeated-root authority handoff

The first integrated full-suite run exposed an invalid Phase C assumption:
one `ProtosActivation` may execute more than one root over the same semantic
current context, notably in REPL persistence and repeated module evaluation.

The initial implementation rejected installation of a second frame authority.
The observed failures were:

```text
ProtosReplTest.nestedClosureMultilineInputExecutesOnlyWhenComplete
ProtosReplTest.multilineCounterClosureIncrementsAcrossCalls
ProtosCsvModuleTest.standardLibraryModuleAndParserInstancesAreActorLocalAndIndependent
```

The CSV failure reported:

```text
IllegalStateException: activation already owns another frame lexical authority
```

The repair preserves the same handoff semantics already used by a materialized
`ProtosExecutionContextValue`: before replacing an unmaterialized deferred
authority, existing bindings are projected in establishment order and migrated
into the new frame-backed authority. There is still one current authoritative
store, not two parallel lexical authorities.

The final integrated gate passed after this repair.

## Semantic validation

The first focused Java gate covered I068/PLAT036 execution-context and lexical
projection:

```text
ProtosI068Slice5CapturedMaterializedLexicalLoweringTest
ProtosI068Slice6DebuggerReflectionProjectionTest
ProtosI068Slice7ActivationLexicalDecompositionTest
ProtosPlat036Slice3FrameBackedCurrentLocalTest
ProtosExecutionContextValueTest
ProtosLexicalBindingAuthoritySeamTest

Tests run: 43, Failures: 0, Errors: 0, Skipped: 0
```

The second focused gate covered positional/default/rest and closure activation
behavior:

```text
ProtosI068Slice4SequentialParameterLoweringTest
ProtosPerf006B2C3B2SimpleDefaultBindingTest
ProtosPerf006B2C2GeneralPositionalArityTest
ProtosPerf006B2AClosureActivationTest
CanonicalBindingAnalyzerTest

Tests run: 28, Failures: 0, Errors: 0, Skipped: 0
```

The selected production execution-context conformance files were then run
through the bundled Test Tool:

```text
execution-context/open-creation-and-value-mutation-preserved.protos
execution-context/escape-close-freeze-and-present-null-preserved.protos
execution-context/remove-slot-restored-ordinary-object-unaffected.protos
execution-context/capture-by-reference-and-late-nearer-creation-retargeting.protos

13 passed, 0 failed
```

This covers the Phase C-sensitive semantics including open mutation,
escape/close/freeze, `PRESENT(null) != ABSENT`, D179 remove/recreate,
capture-by-reference and late nearer creation/retargeting.

## Structural hot-path discriminator

After compilation, generated Bytecode DSL roots were present at:

```text
target/generated-sources/annotations/com/guillermomolina/protos/execution/ProtosBytecodeRootNodeGen.java
target/generated-sources/annotations/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNodeGen.java
```

The reported structural falsifier passed:

```text
COMPACT_FRAME_ARGUMENT_ABI=YES
PRE_TARGET_RICH_CALLEE_ACTIVATION=NO
ORDINARY_EXECUTION_REQUIRES_EAGER_GUEST_CONTEXT=NO
CONTEXT_MATERIALIZATION_CONDITIONAL=YES
ORDINARY_INTERNAL_ARGUMENT_TRANSPORT_REQUIRES_GUEST_ARRAY=NO
SUPPLIED_VECTOR_INTERNAL_GUEST_ARRAY_REMOVED=YES
REST_ARRAY_REQUIRED=YES
INSTALL_REQUIRES_EAGER_FRAME_MATERIALIZE=NO
GENERATED_BYTECODE_DSL_PRESENT=YES
I072_C_STRUCTURAL_FALSIFICATION=PASS
```

The final candidate re-check after metadata finalization also reported:

```text
PHASE_B_COMPACT_TARGET_ENTRY_PRESERVED=YES
ORDINARY_EXECUTION_REQUIRES_EAGER_GUEST_CONTEXT=NO
CONTEXT_MATERIALIZATION_CONDITIONAL=YES
ORDINARY_INTERNAL_ARGUMENT_TRANSPORT_REQUIRES_GUEST_ARRAY=NO
SUPPLIED_VECTOR_INTERNAL_GUEST_ARRAY_REMOVED=YES
REST_ARRAY_REQUIRED=YES
I072_C_FINAL_CANDIDATE=PASS
```

## Integrated validation and publication

The initial integrated `make test` run was intentionally treated as a real
semantic gate, not waived. It found the repeated-root authority bug described
above. After the authority-handoff repair, the human executor reported:

```text
make test
RESULT=PASS
```

Before finalization:

```text
HEAD=795c1ea75236e028a42bbfb91f8c26f8d12c73e4
origin/main=795c1ea75236e028a42bbfb91f8c26f8d12c73e4
AHEAD_BEHIND=0 0
WORKTREE=Phase C source delta only
```

Late finalization then selected:

```text
PROTOS_VERSION=0.3.92-SNAPSHOT
```

and added only the required Maven version/changelog metadata plus a source
comment correction; no executable behavior changed after the final green
`make test`.

The published product commit is:

```text
045bfdb8ec3ccaf09be8b35a2053c124a9edebb8
I072-C: defer guest invocation materialization

Refs guillermomolina/protos#719. AI assistance: substantial.
```

The product worktree was reported clean after push to `origin/main`.

## Phase boundary

Phase C intentionally does not perform Phase D/E work. Existing
`PreparedClosureCall`, optional control-mode state, Task/suspension/continuation
machinery and structured-control callback convergence remain for later bounded
I072 phases.

## Next slice

```text
NEXT_I072_PHASE=Phase D — optional control/state and prepared-call separation
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

Phase D must start from current `origin/main`, preserve the Phase A/B/C
selection, compact frame ABI and conditional guest-state materialization, and
make ordinary calls stop paying unrelated universal
`PreparedClosureCall`/control-mode state. Exact non-local return, Error/dynamic
control, Task ownership, suspension and continuation semantics remain mandatory.

If current source evidence reveals that the smallest representation for optional
state requires a substantive architecture choice not already selected by
PLAT040, Phase D must stop and route that choice through PLAT governance rather
than inventing a Protos-only optimization.
