# TOOL009-C / #686 — Slice 4 Group Suite-Native Readiness Evidence

Date: 2026-09-22

## Investigated revision

The investigation was performed against the published Protos `origin/main` revision:

```
4d8d3f5846c68f6931a308a6e2cf99f090ac0267
```

This is the published revision containing TOOL009-C Slice 3 (Actor migration).

## Result

```
TOOL009C_SLICE4_GROUP = NEEDS_GROUP_FACILITY

GROUP_MANIFEST_ENTRIES_TOTAL = 10
GROUP_ALREADY_SUITE_NATIVE = 0
GROUP_REQUIRING_MIGRATION = 10

GROUP_LOGICAL_CASE_BINDING_PRESENT = NO

GROUP_CAN_REUSE_ORDINARY_LOGICAL_CASE_FACILITY = YES
GROUP_CAN_REUSE_ACTOR_PATTERN = YES
GROUP_NEEDS_DISTINCT_BOOTSTRAP = YES

NEW_DECISION_REQUIRED = NO
```

## Binding state

`protos/test/group` currently publishes:

```
executionAsync
executionInspectAsync
resourceExecutionAsync
resourceExecutionInspectAsync
```

It does not publish:

```
logicalCaseExecutionAsync
```

The binding registry already contains the optional Logical Case binding machinery used by the ordinary and Actor lanes. The missing Group route is therefore a plumbing gap, not a new execution model.

Primary implementation source:

- `src/main/java/com/guillermolina/protos/cli/ProtosTestExecutionRequirementRegistry.java`

## Existing Group bootstrap

The legacy Group execution path is already backed by a Group-specific Prelude and resolver.

The Group resolver overlays:

```
"workers"
    -> module key "tool002-group:workers"
    -> protos/tests/conformance/group/modules/workers.protos
```

The module provides the Group fixtures' `tagged`, `readValue`, and `echo` worker behavior.

Primary implementation source:

- `src/main/java/com/guillermolina/protos/cli/ProtosCli.java`
- `src/main/java/com/guillermolina/protos/cli/ProtosTestToolAsyncExecutionScope.java`
- `protos/tests/conformance/group/modules/workers.protos`

The suite-native Group route must preserve this exact Group-specific resolver/bootstrap authority.

## Suite-native comparison

The existing suite-native Logical Case facility already establishes the required execution model:

```
Logical Case
    -> logicalCaseExecutionAsync
    -> fresh Process / Prelude
    -> rematerialized source
    -> selected Test resolution
    -> selected Test.call()
```

Actor Slice 3 proves that the same `ProtosTestLogicalCaseExecutionFacility` can be reused with a distinct bootstrap slot and a specialized fallback resolver/overlay.

Therefore Group does not require a parallel Case authority or a new Logical Case execution abstraction.

Required Group composition:

```
existing ProtosTestLogicalCaseExecutionFacility
    +
Group-specific Logical Case bootstrap slot
    +
Group-specific fallback resolver
        "workers" exact overlay
```

## Invariants

The current architecture establishes:

```
DISCOVERY_OBSERVATIONAL = YES
SELECTED_TEST_AUTHORITY = YES
TEST_BODY_EXECUTES_EXACTLY_ONCE = YES
FRESH_PROCESS_PER_CASE = YES
GROUP_STATE_ISOLATED_BETWEEN_CASES = YES
INTRA_CASE_GROUP_STATE_PRESERVED = YES
```

Discovery remains observational. The selected `Test.call()` is the Case execution authority. Each Logical Case receives a fresh semantic Process, so Group state created inside one Case cannot leak into another Case; state shared among Group members within one Case remains available for that Case.

No current evidence requires a different authority model for Group.

## Group corpus inventory

Current `protos/tests/conformance/group/manifest.tsv` contains ten entries:

```
acquisition-requires-member.protos          error
acquisition-rejects-non-actorref.protos     error
same-ref-alias-stable.protos                boolean
separate-acquisitions-distinct.protos       boolean
request-before-members-ready.protos         future-integer
request-selects-one-eligible-member.protos  future-integer-one-of
request-snapshots-argument.protos           future-integer
groupref-transfer-preserves-identity.protos future-boolean
stopped-member-not-routed.protos            future-integer
groupref-has-no-stop.protos                 error
```

Expectation-family counts:

```
boolean               = 2
error                 = 3
future-integer        = 3
future-integer-one-of = 1
future-boolean        = 1
```

These families can be represented by the current suite-native Test-body/Assertions model without introducing a new expectation mechanism.

## Special Group behaviors requiring preservation

The fixtures exercise concrete Group semantics including:

- member validation during Group acquisition;
- GroupRef identity and distinct independent acquisition;
- request before members are ready;
- eligible-member selection;
- argument snapshotting across asynchronous request;
- GroupRef transfer preserving identity;
- routing after a member terminates;
- absence of a GroupRef stop operation.

The worker module and the Group runtime bootstrap remain required test authority for these behaviors.

## Existing characterization coupling

`src/test/java/com/guillermomolina/protos/execution/ProtosExactExecutionFacilityTest.java` directly consumes:

```
protos/tests/conformance/group/request-selects-one-eligible-member.protos
protos/tests/conformance/group/modules/workers.protos
```

That characterization must be decoupled as part of the Group migration rather than weakened or removed.

The existing execution-requirement registry tests also currently encode the fact that Group lacks the Logical Case binding. Those assertions must be updated when the facility is implemented.

A new focused Group Logical Case characterization should cover at least:

- Group-specific `workers` resolver/overlay usage;
- selected `Test.call()` execution;
- Group request behavior inside the selected Test;
- fresh Process / Group isolation across independent Cases.

## Scope boundary

The readiness result does not require changes to:

- D152;
- D153;
- CaseAuthority;
- Process Snapshot;
- Actor Logical Case execution;
- legacy Group execution semantics;
- normative language semantics.

No design checkpoint is required.

## Materially inspected sources

Repository instructions:

- `AGENTS.md`
- `AGENTS.work/TOOL.md`
- `AGENTS.work/IMPLEMENTATION.md`
- `protos/AGENTS.md`
- applicable `src/AGENTS.md`
- applicable `spec/AGENTS.md`
- `AGENTS.work/REFERENCE.md`

Current Protos implementation/tests:

- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`
- `src/main/java/com/guillermolina/protos/cli/ProtosTestExecutionRequirementRegistry.java`
- `src/main/java/com/guillermomolina/protos/cli/ProtosTestToolAsyncExecutionScope.java`
- `src/main/java/com/guillermolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java`
- `src/main/java/com/guillermolina/protos/execution/ProtosTestResourceExecutionScope.java`
- `src/test/java/com/guillermolina/protos/execution/ProtosExactExecutionFacilityTest.java`
- `src/test/java/com/guillermolina/protos/execution/ProtosActorTestLogicalCaseExecutionFacilityTest.java`
- `src/test/java/com/guillermolina/protos/cli/ProtosTestToolExecutionRequirementRegistryTest.java`
- `src/test/java/com/guillermolina/protos/cli/ProtosTestToolH2B3PublicIntegrationTest.java`
- `src/test/java/com/guillermolina/protos/execution/ProtosTestToolActorGroupOwnershipArchitectureTest.java`

Group corpus:

- `protos/tests/conformance/group/manifest.tsv`
- all ten Group fixture sources
- `protos/tests/conformance/group/modules/workers.protos`
- `protos/tests/conformance/README.md`
- `docs/guide/tools/test-tool.md`

Relevant live coordination records:

- #600 — TOOL009
- #686 — TOOL009-C
- #687 — TOOL009-D
- #688 — TOOL009-E
- #689 — D178

## Conclusion

Group migration is technically blocked only by the missing suite-native facility/binding.

The next implementation is directly bounded to:

1. install a Group-specific Logical Case bootstrap slot;
2. reuse `ProtosTestLogicalCaseExecutionFacility`;
3. supply the existing Group-specific resolver with the `workers` exact overlay;
4. publish `logicalCaseExecutionAsync` for `protos/test/group`;
5. add focused Group Logical Case characterization;
6. migrate the ten Group corpus entries and decouple the directly dependent characterization test.

This evidence records an investigation only. No Group fixture, manifest, Java source, version, or legacy execution implementation was changed by the investigation itself.
