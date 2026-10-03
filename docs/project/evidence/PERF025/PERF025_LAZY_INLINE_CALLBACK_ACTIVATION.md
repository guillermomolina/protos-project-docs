# PERF025 — Lazy semantic activation for PLAT044 B′ inline callbacks

## Status

```text
STATUS=COMPLETE
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=bd8617157d9c8cc6e1ce0edc0502b7a361e1e891
PRODUCT_PARENT_REVISION=85bd050e8706c02205db4ede8824681a1180d075
PRODUCT_VERSION=0.3.168-SNAPSHOT
PRODUCT_COMMIT=PERF025: lazily materialize inline callback activations

CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
CALLBACK_LAZY_SEMANTIC_ACTIVATION=COMPLETE
OPTIONAL_RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=UNDECIDED_PENDING_POST_SLICE_AUDIT

PLAT044_B_PRIME_COMPATIBLE=YES
NEW_PLATFORM_DECISION_REQUIRED=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
```

## Purpose

Complete the second inline-callback residual slice after
`4fa64c38`: keep a PLAT044 B′ inline callback's semantic Activation fresh per
invocation while avoiding construction of a Java `ProtosActivation` at callback
region entry whenever execution can remain entirely frame-native.

The implementation reuses the existing
`ProtosBytecodeRootNode.PreparedInlineLiteralCall` compact invocation carrier.
No second callback ABI or identity object is introduced.

## Published representation

The inline region now stores the invocation carrier in
`inlineCallbackCall` rather than eagerly storing a semantic Activation.

For an admitted frame-native callback:

```text
REGION_ENTRY_PROTOSACTIVATION=NO
PREPARED_INLINE_LITERAL_CALL_IS_IDENTITY_CARRIER=YES
FRESH_SEMANTIC_ACTIVATION_PER_INVOCATION=YES
MATERIALIZATION_AT_MOST_ONCE=YES
FIRST_SEMANTIC_OR_TOOLING_OBSERVER_MATERIALIZES=YES
```

The carrier's existing compact target arguments remain the publication point:
once `ProtosFrameArguments.activation(...)` materializes the Activation it
publishes that exact object back into the compact argument array, so every later
consumer and continuation of the same callback invocation observes the same
Activation.

## Frame-native ordinary path

The frame-native inline callback path now has carrier-operand operations for the
work that does not require a rich Activation:

- supplied argument reads and upper-bound checks;
- parameter establishment;
- current-local creation and multiple creation;
- statically resolved current-local reads;
- PERF028-A resolved current-local destination selection and writes.

Consequently zero-, one- and two-parameter B′ callback shapes can execute their
ordinary frame-native lexical work without constructing the semantic Activation.

The structural focal coverage distinguishes the two activation operations
exactly:

```text
FRAME_NATIVE_UNOBSERVED_REGION_CONTAINS_LoadInlineCallbackActivation=NO
FRAME_NATIVE_UNOBSERVED_REGION_CONTAINS_MaterializeInlineCallbackActivation=NO

OBSERVER_PATH_CONTAINS_MaterializeInlineCallbackActivation=YES
OBSERVER_PATH_CONTAINS_LoadInlineCallbackActivation=NO

NON_FRAME_NATIVE_CONTEXT_OBSERVER_PATH_USES_LoadInlineCallbackActivation=YES
```

## Durable transition at first observer

The initial implementation draft exposed a correctness window in which an
Activation could exist before its frame-native callback bindings had moved to
durable storage. That draft was not published.

The released implementation closes the window with
`MaterializeInlineCallbackActivation`.

While the callback frame is live, this operation receives:

```text
PreparedInlineLiteralCall
LocalRangeAccessor
ProtosFrameLexicalLayout
VirtualFrame
```

and, before returning the Activation to the observer:

1. creates or reuses the one Activation published in the carrier;
2. copies only PRESENT callback bindings in layout order to a new durable
   map-backed lexical authority;
3. installs that authority on the Activation;
4. clears/relinquishes the ephemeral callback block locals;
5. marks the carrier's frame bindings transferred;
6. returns the same published Activation.

Therefore:

```text
DURABLE_TRANSFER_BEFORE_OBSERVER_RETURN=YES
POST_MATERIALIZATION_BINDING_OPERATION_REQUIRED=NO
CONTEXT_OBSERVER_SEES_DURABLE_BINDINGS=YES
BLOCK_LOCAL_ALIAS_AFTER_ESCAPE=NO
TWO_SIMULTANEOUS_AUTHORITATIVE_STORES_AFTER_TRANSFER=NO
```

After transfer, accesses for that invocation use the durable Activation-owned
authority rather than the reusable inline block locals.

## Tooling

`ProtosBytecodeTagTreeNodeExports` now finds the live
`PreparedInlineLiteralCall` carrier and materializes it through the same
durable-transfer boundary when tooling requests the callback scope.

The temporary suspension-local binding snapshot introduced by the preceding
Slice 1 is removed from `ProtosDebuggerScope`; the debugger projects the
durable callback Activation after transfer.

The ratified PLAT044 tooling boundary is unchanged:

```text
CUSTOM_INLINE_ROOTTAG=YES
DISTINCT_CALLBACK_ROOTCALLTARGET=NO
DISTINCT_CALLBACK_FRAMEINSTANCE=NO
DISTINCT_CALLBACK_TRUFFLE_STACK_TRACE_ELEMENT=NO
CUSTOM_DAP=NO
DEBUGGER_ON_DEMAND_MATERIALIZATION=YES
```

## Semantic preservation

The published CHANGELOG records preservation of:

- D179 PRESENT/ABSENT behavior;
- destination selection before RHS evaluation;
- FROZEN/mutation Errors;
- callback arity Errors;
- ReturnHome/non-local return;
- suspension/resumption;
- PLAT044 B′ admission;
- the non-frame-native context-observing fallback.

Multi-Context ownership remains on the existing Context-owned callback-plan and
compact-call machinery; no global callback cache or registry was introduced.

## Focal evidence

The new
`ProtosPerf025LazyInlineCallbackActivationTest` freezes, among other cases:

```text
INLINE_CALLBACK_ENTRY_MATERIALIZES_ACTIVATION=NO
FRAME_NATIVE_ZERO_PARAMETER_CALLBACK_STAYS_LAZY=YES
FRAME_NATIVE_ONE_PARAMETER_EACH_STAYS_LAZY=YES
FRAME_NATIVE_TWO_PARAMETER_EACH_STAYS_LAZY=YES
FRAME_NATIVE_PARAMETER_PATH_WITHOUT_ACTIVATION=YES
FRAME_NATIVE_CURRENT_READ_WITHOUT_ACTIVATION=YES
FRAME_NATIVE_CURRENT_WRITE_WITHOUT_ACTIVATION=YES
FRAME_NATIVE_CREATE_WITHOUT_ACTIVATION=YES

FIRST_SEMANTIC_OBSERVER_MATERIALIZES=YES
MATERIALIZATION_AT_MOST_ONCE=YES
FRESH_IDENTITY_PER_INVOCATION=YES
PREPARED_INLINE_LITERAL_CALL_IS_IDENTITY_CARRIER=YES

DURABLE_TRANSFER_BEFORE_OBSERVER_RETURN=YES
POST_MATERIALIZATION_BINDING_OPERATION_REQUIRED=NO
CONTEXT_OBSERVER_SEES_DURABLE_BINDINGS=YES
BLOCK_LOCAL_ALIAS_AFTER_ESCAPE=NO

UNOBSERVED_INVOCATION_MATERIALIZES_ACTIVATION=NO
UNOBSERVED_INVOCATION_TRANSFERS_BINDINGS=NO
DEBUGGER_MATERIALIZES_ON_DEMAND=YES
SUSPEND_RESUME_PRESERVES_IDENTITY=YES
NLR_SEMANTICS_PRESERVED=YES
```

The user reported the implementation pushed and tested before this evidence
publication. The product commit also includes the 0.3.168-SNAPSHOT version bump
and CHANGELOG publication.

No new benchmark magnitude is claimed by this structural implementation record.

## Publication

```text
PRODUCT_REVISION=bd8617157d9c8cc6e1ce0edc0502b7a361e1e891
PRODUCT_PARENT_REVISION=85bd050e8706c02205db4ede8824681a1180d075
PRODUCT_VERSION=0.3.168-SNAPSHOT
PRODUCT_COMMIT=PERF025: lazily materialize inline callback activations
PRODUCT_PUSH=PASS
VALIDATION_REPORTED_BY_EXECUTOR=PASS
```

## Residual gate

The pre-existing sequencing contract was:

```text
1 CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY
2 CALLBACK_LAZY_SEMANTIC_ACTIVATION
3 OPTIONAL_RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION
```

Steps 1 and 2 are now complete.

Step 3 is explicitly optional and must not be implemented by assumption. The
next work item is a bounded post-Slice-2 audit of the remaining
Activation-materializing consumers on admitted B′ paths. Its output must decide
whether any residual specialization is still justified or whether the inline
callback technical line is complete.
