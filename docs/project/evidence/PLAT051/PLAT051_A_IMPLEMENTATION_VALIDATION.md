# PLAT051-A — generic semantic-value transfer implementation validation

Status: **COMPLETED AND PUBLISHED**

Evidence date: **2026-10-07**

Decision Issue: `guillermomolina/protos#813` — PLAT051

Parent workstream: `guillermomolina/protos#431` — LIB014

Governing semantic authority for the first consumer: `guillermomolina/protos#809` — D187.

Maintained platform decision:

`docs/project/decisions/platform/PLAT051_STANDARD_LIBRARY_SEMANTIC_VALUE_TRANSFER_REMATERIALIZATION.md`

Prior implementation release record:

`docs/project/evidence/PLAT051/PLAT051_A_IMPLEMENTATION_RELEASE.md`

## Published product revision

```text
PUBLISHED_SHA=4da84e57c8a6716982412dd955583fcc34d4a0b7
COMMIT_MESSAGE=PLAT051-A: add generic Standard Library semantic-value transfer mechanism
IMPLEMENTATION_VERSION=0.3.258-SNAPSHOT
```

The published revision contains the PLAT051-A generic mechanism, its tests, the implementation-version bump and the corresponding CHANGELOG entry.

## Human-executor validation

After publication, the project owner explicitly reported:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
```

No build or test execution is invented by this record. A separate `mvn -Pnative package` result was not reported and is not claimed here.

## Implemented generic mechanism

PLAT051-A implements Candidate C as three internal runtime building blocks:

```text
ProtosSemanticTransferFamily
ProtosSemanticTransferValue
ProtosSemanticTransferPayload
```

The family descriptor owns an exact `std:` `ProtosModuleKey`, source-local semantic extraction, destination-local reconstruction, minting of family values, and one fail-closed rematerialization entry point shared by the transfer implementations.

`ProtosSemanticTransferValue` is a specialized internal `ProtosObjectValue` representation carrying only its trusted family descriptor. The generic ordinary `ProtosObjectValue` layout is unchanged.

`ProtosSemanticTransferPayload` is an inert immutable acyclic payload tree whose leaves are restricted to host scalar data. It is not a public serialization format and cannot contain a Closure, Protos object graph, capability, execution context or other runtime value.

## Bootstrap authority

Trusted-family authority is fixed at Prelude construction.

`ProtosCoreBootstrap.standardSemanticTransferFamilies()` currently supplies no production family. The Prelude stores an immutable mapping from exact owning module key to exact descriptor identity, with at most one family per module key.

Authorization therefore requires the exact bootstrap-provided descriptor identity. A guest-created look-alike object, slot, String, shape, tag, delegation parent or second descriptor for the same module key does not gain transfer authority.

A package-private bootstrap seam exists only for tests.

```text
STANDARD_LIBRARY_MEMBERSHIP_ALONE_IMPLIES_PORTABILITY=NO
BOOTSTRAP_AUTHORITY=IMPLEMENTED
GLOBAL_MUTABLE_REGISTRY=NO
PUBLIC_USER_SERIALIZATION_PROTOCOL=NO
```

## Actor and P integration

`ProtosActorValueTransfer` and `ProtosParallelRuntime.Transfer` recognize only `ProtosSemanticTransferValue` after their existing memo/shared-standard checks.

For an authorized semantic value they:

1. validate the destination Prelude authority;
2. require a frozen source value;
3. extract only the inert semantic payload;
4. reconstruct a fresh destination-local value;
5. require the reconstructed value to be fresh, frozen and owned by the same exact family; and
6. place the result in the existing transfer identity memo.

Failure is closed as the existing transfer failure (`NonTransferableValue` for Actor transfer and `NonParallel` for P). There is no fallback to copying a source Closure, parent/callable implementation graph or private execution state.

The existing P rule that specially projects the explicitly executed computation Closure remains independent and unchanged; arbitrary Closures did not become transferable.

## Identity, aliasing and cycles

Both integrations use the pre-existing transfer-wide identity memo.

```text
REPEATED_SOURCE_IDENTITY_ONE_TRANSFER=ONE_DESTINATION_IDENTITY
DISTINCT_SOURCE_IDENTITIES=NOT_MERGED
SOURCE_IDENTITY_EQUALS_DESTINATION_IDENTITY=NO
SURROUNDING_ORDINARY_GRAPH_CYCLE_HANDLING=UNCHANGED
```

The semantic payload itself is deliberately acyclic. A future semantic family requiring cyclic payload construction remains outside the current opt-in contract until it has an explicit safe two-phase reconstruction design.

## Pay-as-you-grow result

The ratified localized-cost gate is preserved structurally.

For an ordinary object transfer, the added production work is one final-class `instanceof ProtosSemanticTransferValue` recognition check after existing transfer checks.

Programs not using an authorized rematerializable value pay:

```text
ORDINARY_OBJECT_LAYOUT_NEW_REQUIRED_FIELD=NO
ORDINARY_TRANSFER_STDLIB_SCAN=NO
ORDINARY_TRANSFER_MODULE_IMPORT=NO
ORDINARY_TRANSFER_SOURCE_EXECUTION=NO
ORDINARY_TRANSFER_SEMANTIC_PAYLOAD_ALLOCATION=NO
GLOBAL_MUTABLE_RECONSTRUCTION_REGISTRY=NO
```

The Prelude owns one small immutable family map created once at bootstrap. Extraction, payload construction, family authorization beyond the recognition branch, and destination reconstruction are paid only by values that actually opt in and cross an isolation boundary.

No dedicated transfer microbenchmark existed or was added for this slice; this is structural implementation evidence rather than a quantified throughput claim.

```text
PAY_AS_YOU_GROW=PASS_STRUCTURAL
PLAT051_REOPEN_REQUIRED=NO
```

## Negative and forgery gates

The retained tests cover:

- Actor and P transfer of a bootstrap-authorized test family;
- repeated alias preservation and distinct source identity preservation;
- fresh destination identity;
- surrounding ordinary graph cycles;
- guest-look-alike names, slots, shapes and delegation not granting authority;
- an impostor descriptor for the same module key not granting authority;
- missing Prelude authority;
- malformed/null/throwing extraction or reconstruction;
- unfrozen sources and invalid reconstructed values;
- Closure-bearing payload attempts being rejected by payload construction;
- ordinary Closure, activation and execution-context negative transfer behavior;
- unchanged P explicit-computation Closure projection;
- ordinary graph behavior not invoking family extraction or module import; and
- absence of Regex-specific, Java-serialization, reflection-driven or ServiceLoader production machinery in the PLAT051-A mechanism.

The human-reported complete local suite passed with these retained tests present.

## Regex remains deliberately unmodified

PLAT051-A does not opt any production Standard Library family into the mechanism.

In particular:

```text
REGEX_SPECIFIC_GENERIC_COPIER_BRANCH=NO
REGEX_PATTERN_OPT_IN=NO
REGEX_MATCH_OPT_IN=NO
CURRENT_REGEX_MATCHING_IMPLEMENTATION_VALID=YES
```

No D187 public Regex semantics, Pike matching rules, replacement/split behavior or source API were changed by A.

## Native Image / portability boundary

The generic production mechanism uses ordinary Java object/type operations and immutable data. It introduces no Java serialization, reflective class loading, dynamic plugin registry, JNI contract, ServiceLoader requirement or wire format.

The human-reported local suite passed, including the repository's ordinary local validation surface. A separate standalone `mvn -Pnative package` was not reported, so this record claims structural Native Image compatibility plus existing-suite validation only, not an additional independent native-package build.

## PLAT051-B release

PLAT051-A's required implementation and local validation gates are complete. The previously ordered second slice is therefore mechanically released:

```text
SLICE=PLAT051-B
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
GOAL=opt std:regex/Regex Pattern and Match into the ratified semantic-transfer mechanism
```

PLAT051-B must implement the D187 portable data mapping without changing either generic copier into a Regex-specific switch.

Pattern mapping remains:

```text
PAYLOAD=source + canonical flags
DO_NOT_TRANSFER=Pike program, source Closures, lexical/module execution context, matcher state
DESTINATION=compile/reconstruct with destination-local Regex authority
```

Match mapping remains:

```text
PAYLOAD=captured texts including group 0 + scalar half-open bounds + capture count + capture-name/group-number relation
DO_NOT_TRANSFER=whole subject merely for convenience, Pattern implementation graph, Pike program, matcher state, source Closures
DESTINATION=construct result locally without rerunning Regex
```

PLAT051-B must also solve the concrete representation integration exposed by A: Regex Pattern/Match objects are currently created in Protos source, while PLAT051 authority requires values minted by the trusted family. Any minting bridge must remain bootstrap-authorized, module-local, non-forgeable and not become a public serialization institution. The specialized value representation must retain ordinary guest object/member behavior; do not broaden unrelated hot-path exact-class optimizations merely to make the subclass fast unless evidence requires it.

## LIB014 coordination

With A complete, LIB014 is no longer blocked on generic PLAT051 infrastructure. It remains open only for the final D187 portability implementation/proof owned by B.

```text
PLAT051_DECISION=RATIFIED_AND_CLOSED
PLAT051_A=COMPLETE
PLAT051_B=READY_FOR_IMPLEMENTATION
LIB014_FUNCTIONAL_BASELINE=COMPLETE
LIB014_FINAL_PORTABILITY_CLOSURE=BLOCKED_BY_PLAT051_B
D187_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

After PLAT051-B passes its gates and is published, LIB014 may close if no new concrete defect or independently required slice remains. LIB014-4 acceleration is still optional and requires separate measured evidence.

## Result

```text
PLAT051_A_IMPLEMENTATION=PUBLISHED_AND_VALIDATED
PUBLISHED_SHA=4da84e57c8a6716982412dd955583fcc34d4a0b7
IMPLEMENTATION_VERSION=0.3.258-SNAPSHOT
GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
ALL_LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

GENERIC_PRIVILEGED_REMATERIALIZATION_MECHANISM=PASS
ACTOR_INTEGRATION=PASS
P_INTEGRATION=PASS
BOOTSTRAP_AUTHORITY=PASS
FORGERY_NEGATIVE_GATE=PASS
ALIAS_IDENTITY_GATES=PASS
ARBITRARY_CLOSURE_TRANSFER=NO
CAPABILITY_RULE_DELTA=NONE
REGEX_SPECIFIC_BRANCH=NO
REGEX_OPT_IN=NO
ORDINARY_OBJECT_LAYOUT_FIXED_COST_DELTA=NONE
GLOBAL_MUTABLE_REGISTRY=NO
ORDINARY_STDLIB_SCAN_IMPORT_EXECUTION=NO
PAY_AS_YOU_GROW=PASS_STRUCTURAL
D187_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

NEXT=PLAT051-B
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from ChatGPT using the published PLAT051-A product revision, the ratified PLAT051/D187 records, current repository coordination state, and the project owner's explicit human-executor validation report. No unreported build or test execution is claimed.
