# D168 — Process bootstrap snapshot identity and argument representation

Status: **RATIFIED — Candidate C**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#638`  
Trigger: AUD009-D1 / `guillermomolina/protos#636`  
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`  
Project-record base: `16d67807958ead93566faa29b9dd3ab96d8f7258`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D168 selects **Candidate C — frozen ordinary Array for Process arguments; specialized Environment retained**.

```text
process.args() capability                         KEEP
stable bootstrap argument contents                KEEP
stable argument order                             KEEP
all portable arguments are String                 KEEP
later host argv mutation invisible                KEEP

process.environment() capability                  KEEP
stable bootstrap environment contents             KEEP
Environment native-name semantics                 KEEP
Environment representability rules                KEEP
Process explicit capability authority             KEEP

canonical args snapshot semantic identity         REMOVE
canonical Environment snapshot semantic identity  REMOVE
cross-Actor canonical reacquisition identity      REMOVE

identity relation across separate accessor calls  NOT_PORTABLE
identityHashOf relation across acquisitions        NOT_PORTABLE
IdentityMap key equivalence across acquisitions   NOT_PORTABLE

ProcessArguments semantic family                  REMOVE
ProtosProcessArgumentsValue                       REMOVE
dedicated ProcessArguments native protocol        REMOVE

process.args() result                             FROZEN_ORDINARY_ARRAY_OF_STRING
ordinary Array Actor/P transfer                    KEEP
ordinary Array interop / size / at / each          KEEP

Environment semantic family                       KEEP
Environment native-name domain                    KEEP
Environment contains/get conversion semantics     KEEP
ordinary Map substitution                         REJECT

Arguments wrapper                                 NOT_ADDED
ImmutableSequence abstraction                     NOT_ADDED
common BootstrapSnapshot family                   NOT_ADDED
```

## Core interpretation

D168 preserves **bootstrap content stability**, not one canonical object identity.

Portable code may rely on the fact that one Process has one stable bootstrap argument sequence and one stable bootstrap environment content snapshot. Portable code must not infer Process provenance, sameness, or authority from `===`, `identityHashOf`, default identity-derived hash, or IdentityMap behavior of independently acquired accessor results.

An implementation may cache or rematerialize accessor results as an implementation choice provided all retained observable content/protocol guarantees hold.

## Why ProcessArguments is removed

The current special argument family owns dedicated runtime representation, protocol, transfer branches, interop/diagnostic handling, and structured execution paths, while its useful positive protocol is essentially:

```text
size
at
each
```

A frozen ordinary Array already supplies ordered indexed String storage, `size`, `at`, `each`, ordinary Actor/P transfer and ordinary interop.

Because every portable argument is a String, shallow Array freezing is sufficient to make the argument snapshot transitively immutable with respect to its elements.

A frozen Array retains ordinary Array selectors; mutation selectors such as `atPut` fail because the receiver is frozen. D168 intentionally accepts that ordinary Array surface rather than introducing a replacement wrapper solely to hide mutation selectors.

## Why Environment remains specialized

Environment is not reduced to Map.

Its retained semantics include host/native name identity and representability, deferred String conversion, and lookup behavior where `contains` need not decode an invalid value. Those rules are not ordinary Map semantics and remain independently justified.

Only the **canonical object identity of repeated Environment acquisitions** is removed.

## Comparative evidence

The packet compared multiple approaches, including Python, Java, .NET, Rust, Go, Swift and Node.js.

Across those systems, argument/environment APIs vary between arrays/lists, iterators, maps/objects and copies/snapshots. Stable **container identity** is generally not the portable property applications depend on.

The stronger recurring concern is data/content representation:
- arguments are ordered values;
- environment APIs often retain host-specific representation/equality concerns;
- runtimes commonly distinguish textual convenience from native environment representation.

That supports ordinary Array reuse for arguments while retaining specialized Environment semantics.

## Candidate result

### Candidate A — retain D018/current special representations

Rejected. It preserves a canonical identity promise with no production consumer found and keeps a privileged argument family whose useful behavior duplicates ordinary Array.

### Candidate B — remove canonical identity, retain ProcessArguments

Rejected. It removes one unnecessary promise but leaves most of the dedicated runtime/protocol/transfer maintenance burden.

### Candidate C — frozen ordinary Array + specialized Environment

**Selected.** It preserves all useful bootstrap content and Environment-native semantics while removing both canonical identity and the unnecessary special argument family.

### Candidate D — source/library immutable wrapper

Rejected. It would largely replace one special wrapper institution with another without evidence that hiding ordinary frozen-Array selectors is worth a new abstraction.

### Candidate E — common immutable snapshot abstraction

Rejected. Arguments and Environment do not share enough semantic requirements to justify a new common Core family.

## Strongest argument against Candidate C

A dedicated ProcessArguments family communicates provenance and exposes a narrower positive protocol than ordinary Array. A canonical Process-wide snapshot identity also provides a compact notion of “the argument snapshot of this Process”.

D168 accepts that conceptual neatness is not sufficient to justify observable object identity plus dedicated runtime machinery.

Provenance already comes from the explicit `Process` capability and stable bootstrap contract. It need not be re-encoded as one globally canonical identity-bearing result object.

## Fixed-authority consistency

D168 preserves every fixed authority not explicitly reopened:

```text
process.args() remains                         PASS
stable ordered bootstrap argument content      PASS
portable arguments are valid String            PASS
later host argv mutation invisible             PASS
process.environment() remains                  PASS
stable bootstrap environment content           PASS
Environment native-name/representability       PASS
Process explicit authority                     PASS
ordinary Actor/P transfer rules                PASS
```

D018's canonical identity and ProcessArguments representation portions are superseded by D168; other D018 authority remains intact unless separately changed.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Implementation consequence

D168 requires normative/runtime/test/documentation reconciliation.

The implementation owner must at minimum:

1. change `process.args()` to produce a frozen ordinary Array of Strings;
2. remove `ProcessArguments` as a semantic/runtime family and its dedicated native protocol;
3. remove dedicated Actor/P transfer, interop, diagnostic and bytecode/C-prime paths that exist only for ProcessArguments, relying on ordinary Array machinery instead;
4. remove canonical identity guarantees/tests for independently acquired args and Environment snapshots;
5. preserve stable bootstrap contents and all Environment native-name/representability behavior;
6. preserve Process capability delegation and ordinary isolation-transfer semantics;
7. update guide/spec/changelog material consistently;
8. introduce no replacement Arguments wrapper or common BootstrapSnapshot abstraction.

```text
D168_STATUS=RATIFIED
SELECTED_CANDIDATE=C

CANONICAL_ARGS_IDENTITY=REMOVE
CANONICAL_ENVIRONMENT_IDENTITY=REMOVE
CROSS_ACTOR_REACQUISITION_IDENTITY=REMOVE

PROCESS_ARGUMENTS_FAMILY=REMOVE
ARGS_RESULT=FROZEN_ARRAY_OF_STRING

ENVIRONMENT_FAMILY=KEEP
ENVIRONMENT_NATIVE_NAME_SEMANTICS=KEEP
ENVIRONMENT_MAP_SUBSTITUTION=REJECT

PUBLIC_REPLACEMENT_WRAPPER=NOT_ADDED
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
