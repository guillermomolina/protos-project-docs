# PERF025 — PLAT044 B′ inline-callback frame-native bindings

## Status

```text
STATUS=COMPLETE
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=4fa64c38
PRODUCT_PARENT_REVISION=ffc351dc
PRODUCT_VERSION=0.3.166-SNAPSHOT
PRODUCT_COMMIT=PERF025: frame-native inline callback bindings

PLAT044_DECISION=B_PRIME_RATIFIED
NEW_PLATFORM_DECISION_REQUIRED=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
OBSERVABLE_TOOLING_STACK_DELTA=UNCHANGED_FROM_PLAT044
PERFORMANCE_MEASUREMENT_PERFORMED=NO
```

## Purpose

Remove the remaining named-authority cost for statically admitted PLAT044 B′
inline callback bindings without yet making the semantic callback
`ProtosActivation` lazy.

Before this slice, an admitted inline callback already avoided a distinct callback
`RootCallTarget` / `FrameInstance`, but its current lexical bindings were
still lowered through the callback Activation's named runtime authority.

This slice makes eligible callback parameters and current locals use static
Bytecode-local storage in the enclosing physical semantic root while preserving
the already-ratified B′ semantic activation and tooling model.

## Implemented representation

```text
SEMANTIC_CALLBACK_ACTIVATION=FRESH_PER_INVOCATION
JAVA_PROTOSACTIVATION_AT_ENTRY=YES
DISTINCT_CALLBACK_ROOTCALLTARGET=NO
DISTINCT_CALLBACK_FRAMEINSTANCE=NO
CUSTOM_INLINE_ROOTTAG=YES

CALLBACK_STATIC_BINDING_ANALYSIS=YES
CALLBACK_STATIC_LAYOUT=YES
CALLBACK_PARAMETER_FRAME_PATH=YES
CALLBACK_CURRENT_LOCAL_CREATE_FRAME_PATH=YES
CALLBACK_CURRENT_LOCAL_READ_FRAME_PATH=YES
CALLBACK_CURRENT_LOCAL_WRITE_FRAME_PATH=YES

FRAME_LAYOUT_AUTHORITY=CANONICAL_SCOPE_STABLE_LAYOUT
D179_PRESENCE_CONTINUITY_ASSUMPTIONS=REUSED
INLINE_OBJECT_BODY_BEHAVIOR_CHANGED=NO
CAPTURED_BINDING_ARCHITECTURE_CHANGED=NO
```

The lowerer now retains the callback's canonical binding analysis and lexical
scope for the eligible frame-native region. Declared callback bindings receive
implementation-only Bytecode locals prefixed with
`$inlineCallbackBinding:`.

The callback reuses `frameLexicalLayoutForScope(...)`, including the exact
root/name D179 one-way presence-continuity assumptions introduced by
PERF025-D179-A. No parallel callback-only membership model is introduced.

## Conservative authority boundary

The frame-native callback path is used only when the existing persistent-frame
authority analysis proves that the callback does not require durable current
context authority.

A callback that directly observes `context`, or otherwise requires the existing
persistent authority path, retains the previous named semantic binding path.

```text
CONTEXT_OBSERVER_FALLBACK=PASS
DURABLE_ESCAPE_TRANSITION=NOT_IMPLEMENTED_IN_THIS_SLICE
```

This is deliberate. The callback's Bytecode locals have inline-block lifetime;
they are not installed as a durable lexical authority on the Activation.

## Tooling preservation

An external debugger can suspend inside an otherwise frame-native inline callback
even when the guest body never evaluates `context`.

The first focal debugger run exposed that boundary correctly:

```text
INITIAL_DEBUGGER_FOCAL=FAIL
FAILURE=callback formal must be visible
MISSING_BINDING=element
```

The final implementation preserves PLAT044 tooling semantics with a
suspension-scoped read-only debugger projection:

- `ProtosBytecodeTagTreeNodeExports` identifies live prefixed callback bindings
  at the queried Bytecode index;
- it snapshots only PRESENT live callback block locals;
- `ProtosDebuggerScope` treats that immutable snapshot as the current lexical
  scope before captured and receiver/delegation bindings;
- no lexical authority is installed on the callback Activation;
- no `Frame` is retained by the scope;
- no callback debugger frame or `TruffleStackTraceElement` is fabricated.

This is tooling projection only and does not implement the future durable
authority transition.

## Validation

Validation after reconciliation with upstream PERF025-D179-A
(`ffc351dca`) was:

```text
MAVEN_COMPILE=PASS

INLINE_CALLBACK_PREPARATION_FOCAL:
  TESTS=5
  FAILURES=0
  ERRORS=0

DEBUGGER_D1_FOCAL_AFTER_TOOLING_FIX=PASS

PROTOS_I026E_SCOPE_REGRESSION:
  TESTS=6
  FAILURES=0
  ERRORS=0

SLICE_1_SEMANTIC_BATTERY:
  TESTS=106
  FAILURES=0
  ERRORS=0

FULL_MAKE_TEST=PASS
GIT_DIFF_CHECK=PASS
WORKTREE_AFTER_PUBLICATION=CLEAN
```

The 106-test battery covered the existing B′ Boolean B1/B2/B3, while C1 and
each D1/D2/D3 lines plus D179 lexical membership, compact-callee behavior,
PERF028-A current resolved writes, callback suspension/resumption, NLR/error
control, fallback shapes, fresh activation semantics and debugger scope
projection.

No benchmark, timing comparison, JFR, IGV or allocation measurement was
performed for this slice, so no performance magnitude is claimed.

## Publication

```text
PRODUCT_REVISION=4fa64c38
PRODUCT_BRANCH=main
PRODUCT_PUSH=PASS
PRODUCT_WORKTREE=CLEAN
```

## Residual

This closes the first implementation step of the previously identified inline
callback residual:

```text
CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
CALLBACK_LAZY_SEMANTIC_ACTIVATION=NEXT
OPTIONAL_RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=AFTER_LAZY_ACTIVATION_IF_JUSTIFIED
```

The next slice may use the existing per-invocation
`PreparedInlineLiteralCall` carrier to publish exactly one semantic
`ProtosActivation` on first semantic/tooling observation. It must preserve
fresh activation/context identity, ReturnHome/NLR, suspension, multi-Context,
tooling and the exact PLAT044 B′ physical-frame boundary.
