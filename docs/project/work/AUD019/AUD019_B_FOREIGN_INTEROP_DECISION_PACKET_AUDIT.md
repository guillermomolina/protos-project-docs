# AUD019-B — Foreign interop decision-packet audit

Status: **COMPLETE — DECISION PACKET ROUTING ONLY**

Parent issue: `AUD019 / guillermomolina/protos#818`

Research date: **2026-10-07**

Primary audited Protos revision:

```text
PROTOS_REVISION=49e8763b68db1fe7583d36347fe5028fbe90d539
PROTOS_VERSION=0.3.258-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

Post-investigation drift revalidated before publication:

```text
LATER_PROTOS_REVISION=08cfc69477ad45e2fe91a95140e5c71dffb9cdc0
LATER_PROTOS_VERSION=0.3.259-SNAPSHOT
LATER_CHANGE=TEST009-AC
DRIFT_CLASS=NON_MATERIAL
```

The later commit changes the internal `ProtosValueLookup` Optional unwrap to
remove an impossible partial-evaluation exception path and adds focused tests.
Its own changelog states that there is no specification or semantic change. It
does not alter module semantics, callable semantics, Actor/P isolation, foreign
interop authority, Context topology, or the AUD019 conclusions.

This record preserves AUD019-B research. It does **not** ratify any Dxxx/PLATxxx
candidate and does not authorize implementation.

## Executive result

AUD019-B reduced the remaining AUD019 design space to the minimum sufficient set
of four substantive decision packets:

```text
1. D188
   Foreign values and foreign-module guest semantics

2. D189
   Foreign callback execution and lifetime semantics

3. PLAT052
   Foreign authority and sandbox enforcement

4. PLAT053
   Foreign provider / Context / runtime / lifecycle / distribution architecture
```

The dependency order is:

```text
D188
  |
  v
D189
  |
  v
PLAT052
  |
  v
PLAT053
  |
  v
implementation
```

The research deliberately does not split module identity, foreign object
identity, equality/hash, ordinary member/call/index behavior, scalar conversion,
or Error projection into additional Dxxx items: they are facets of the same
question, namely what a foreign value means when observed by Protos.

Likewise, Native Image does not require another AUD019-specific platform
decision. PLAT045 already owns the Native guest-JIT capability matrix; PLAT053
only needs to define provider/component availability and pay-as-you-grow
distribution consequences.

## Drift from AUD019-A

AUD019-A remains valid.

The material conclusions are unchanged:

```text
GENERIC_FOREIGN_VALUE_SUBSTRATE=YES
GENERIC_ALL_TRUFFLE_MODULE_IMPORT=NO
PER_LANGUAGE_IMPORT_PROVIDER_REQUIRED=YES
PROTOS_FACING_FOREIGN_FACADE=HYBRID
PROTOS_SOURCE_BINDING_REQUIRED=NO
STD_INTEROP_AND_IMPORT_SHARE_SUBSTRATE=YES
IMPORT_RESOLUTION_SEPARATE_FROM_DEPENDENCY_ACQUISITION=YES
```

The only new current-repository authority that materially narrows the remaining
design space is PLAT028.

PLAT028 is durably ratified with Candidate C:

```text
C-prime owns suspendible callback/post-callback control
native/Java remains leaf with respect to arbitrary guest re-entry
no generic host continuation stack
no hidden child Task
no blocking carrier as continuation model
no whole-host-stack capture
```

This does not itself define foreign-to-Protos callback semantics, but it removes
several platform candidates that AUD019 no longer needs to reconsider.

PLAT051 also adds a generic privileged semantic-value rematerialization
mechanism for explicitly admitted `std:` families. It does not opt foreign
values into Actor/P transfer, create automatic proxying, or authorize another
semantic value family. It is therefore future implementation precedent rather
than present foreign-transfer authority.

## Existing normative constraints that eliminate new decisions

### Modules

`spec/semantics/MODULES.md` already requires:

```text
specifier
    -> canonical ModuleKey
    -> Actor-local module cache
    -> cache before execute
    -> cycles / partial initialization
    -> failure eviction / retry
```

The same canonical `ModuleKey` in one Actor denotes the same active Protos
module instance. Across Actors, module instances are distinct.

A Python/JavaScript/Ruby/Java provider cache may exist internally, but it cannot
silently replace those Protos module-identity rules.

### Invocation

`spec/semantics/CALLABLES.md` defines parenthesized invocation through ordinary
lookup of the `call` slot, whose selected value must be a Closure.

Therefore `InteropLibrary.isExecutable` cannot become a hidden second Protos
callability predicate. A foreign executable needs an approved Protos-facing
`call` projection or explicit `std:interop` operation.

### Indexing

`spec/semantics/VALUES_AND_COLLECTIONS.md` already defines:

```text
receiver[index]       -> receiver.at(index)
receiver[index] = x   -> receiver.atPut(index, x)
```

No foreign-specific indexing syntax is needed. D188 must decide only how a
foreign value may faithfully participate in those existing protocols.

### Identity and equality

Protos `===` remains semantic identity and is not overridable.

`==`, `hash`, normal `Map`, and `IdentityMap` keep their existing Protos
contracts. Java/Python/JavaScript/Ruby equality or
`InteropLibrary.isIdentical` cannot become Protos equality/identity
automatically.

### Errors

`spec/semantics/ERRORS.md` owns one Protos Error model. Raw
`PolyglotException`, `UnsupportedMessageException`, Java Throwables, or guest
exception values cannot become a second Protos-visible exception system.

### Blocking foreign calls

`spec/concurrency/ACTORS.md` §24J already establishes that a synchronous
foreign call remains part of the current Protos execution segment and creates
no implicit suspension or Actor-local reentrancy. Physical offload may be
implementation-invisible.

No new Dxxx is required for the baseline synchronous blocking rule.

### Actor/P transfer

`ACTORS.md` §24K/§24L and `PARALLEL_EXECUTION.md` already prohibit arbitrary
foreign mutable state/resources from becoming shared Actor/P aliases and prohibit
silent proxy substitution.

The safe baseline therefore is:

```text
foreign value without explicit crossing contract
    -> non-transferable
```

A universal copy/proxy/rematerialization design is not required before initial
foreign interop.

## D188 — foreign values and foreign-module semantics

Live Issue: `guillermomolina/protos#819`.

D188 is the root semantic decision. It must define one coherent Protos semantic
projection for:

- foreign module adaptation to existing `ModuleKey` and Actor-local module
  identity;
- raw foreign value and facade identity;
- `===`, `==`, `hash`, `Map`, and `IdentityMap`;
- member reads/writes/calls;
- extraction and receiver behavior;
- callability through the existing Protos `call` protocol;
- construction;
- indexed/hash/iterator projection;
- exact/lossless primitive conversion;
- foreign failure -> Protos Error projection;
- ordinary syntax vs explicit `std:interop`.

Candidate families:

```text
A — direct InteropLibrary mapping
B — explicit std:interop only
C — hybrid Protos semantic projection
```

AUD019-B recommendation pending owner review:

```text
RECOMMENDED_CANDIDATE=C
```

The recommendation preserves raw foreign values where the mapping is faithful
and introduces provider/runtime facades only where Protos semantics require
adaptation.

No candidate is selected by AUD019-B.

## D189 — foreign callback semantics

Live Issue: `guillermomolina/protos#820`.

D189 depends on D188 and owns the reverse control direction:

```text
Protos -> foreign -> Protos Closure
```

It must define Actor/Task ownership, receiver and lexical state, conversion and
Error propagation, synchronous reentrancy, nested callbacks, foreign-created
threads, suspension, cancellation, lifetime, retained callbacks, and Process /
Context shutdown behavior.

Candidate families:

```text
A — dynamic synchronous callback baseline
B — new Task per foreign callback
C — retained/general asynchronous callbacks from the baseline
```

AUD019-B recommendation pending owner review:

```text
RECOMMENDED_CANDIDATE=A
```

This keeps the initial callback inside the current Actor/Task and dynamic foreign
call. It does not preserve arbitrary foreign/native frames across Protos
suspension; a later retained/asynchronous callback facility may introduce an
explicit Actor/Task/Future/message ingress boundary.

PLAT028 remains authoritative for continuation ownership.

## PLAT052 — authority and sandbox enforcement

Live Issue: `guillermomolina/protos#821`.

PLAT052 depends on D188 and D189.

It must define how foreign import / `std:interop` receives only explicitly
granted host authority, including:

- HostAccess;
- host class lookup / classloading;
- I/O/filesystem;
- network;
- subprocess/process creation;
- environment/system properties;
- reflection;
- native access / library loading;
- cross-language PolyglotAccess;
- trusted provider/library modes;
- stronger provider-specific isolation.

Fixed boundary:

```text
import itself is not an authority grant
Process does not imply filesystem/network/subprocess/native authority
dependency presence is not execution authority
allowAllAccess(true) is not an acceptable baseline
```

Candidate families:

```text
A — trusted/all-access default
B — deny-by-default in-process authority mapping
C — out-of-process isolation for every foreign runtime
```

AUD019-B recommendation pending owner review:

```text
RECOMMENDED_CANDIDATE=B
```

A trusted mode may later be an explicit host/embedder policy. Provider-specific
out-of-process isolation remains an escape path where in-process controls cannot
preserve the Protos capability model.

## PLAT053 — provider/runtime topology

Live Issue: `guillermomolina/protos#822`.

PLAT053 depends on D188, D189, and PLAT052.

It owns:

- provider scheme routing and provider responsibility;
- canonical foreign `ModuleKey` production;
- provider registry ownership;
- Engine/Context/session ownership;
- provider and foreign-value lifetime;
- provider cache vs Actor-local Protos module-cache boundary;
- thread-affinity/concurrency profiles;
- language discovery/options/search paths;
- host Java vs optional Espresso distinction;
- lazy provider initialization;
- JVM/Native provider availability;
- pay-as-you-grow distribution cost.

Candidate families:

```text
A — every enabled foreign language in the existing Protos Process Context
B — Process-owned lazy provider compartments
C — one foreign Context per Actor as universal topology
```

AUD019-B recommendation pending owner review:

```text
RECOMMENDED_CANDIDATE=B
```

Provider compartments preserve one Protos-side foreign-value/provider contract
while allowing each language ecosystem to use the smallest runtime/Context
topology that satisfies its own threading, lifetime and authority constraints.

## Intentionally deferred non-blocking questions

The following do not block the smallest correct foreign-import baseline:

- universal Java CompletionStage / Future -> Protos Future adaptation;
- universal Python awaitable -> Protos Future adaptation;
- universal JavaScript Promise -> Protos Future adaptation;
- universal Ruby async/fiber adaptation;
- foreign cancellation propagation;
- retained/asynchronous callbacks beyond D189's initial contract;
- generic Actor/P/Process foreign-value copying/rematerialization/proxying;
- third-party provider registration API;
- Maven/PyPI/npm/RubyGems dependency acquisition;
- mandatory generated shims;
- mandatory Protos-written bindings;
- one distribution containing every foreign guest runtime;
- richer mixed-language stack/cause diagnostics unless selected by D188.

These are deliberately future-compatible rather than future-preimplemented.

## Mechanical implementation after decisions

Only after all four decisions are approved and ratified should implementation be
considered mechanical. Expected implementation surfaces then include:

- `ForeignImportProvider` substrate and scheme dispatch;
- canonical foreign `ModuleKey` construction;
- Actor-local foreign-module record/facade integration;
- foreign-value adapters;
- `std:interop`;
- exact scalar conversions;
- Error projection;
- Java host provider;
- GraalPy provider;
- GraalJS provider;
- TruffleRuby provider;
- optional Espresso provider;
- provider-specific Context/session creation;
- Context enter/leave and provider shutdown;
- callback bridge under D189 + PLAT028;
- HostAccess / PolyglotAccess / I/O/native/process authority enforcement;
- provider/component availability and Native distribution gates;
- focused cross-provider, identity, error, authority and zero-use-cost tests.

## Proof matrix routed to later implementation

At minimum later implementation must prove:

- repeated same-`ModuleKey` import;
- distinct spellings canonicalizing to one key;
- cyclic/partially initialized foreign imports;
- failed initialization and retry;
- foreign identity round trips and facade identity;
- `===`, `==`, hash, Map and IdentityMap behavior;
- member read/write/call;
- construction;
- indexing and iteration;
- exact integer/Float/String/null conversion;
- foreign exception -> Protos Error conversion without raw host leakage;
- callback same-Actor/same-Task behavior;
- nested callback and late-callback behavior selected by D189;
- access-denied failure paths;
- provider unavailable / initialization failure;
- Process/provider/Context shutdown;
- concurrent Actors;
- provider cache distinct from Protos module cache;
- enabling provider X does not initialize provider Y;
- unused foreign capability has near-zero startup/runtime/distribution cost.

## Coordination outcome

AUD019-B satisfies the audit's remaining routing obligation by allocating the
four independently meaningful decision checkpoints:

```text
D188    -> guillermomolina/protos#819
D189    -> guillermomolina/protos#820
PLAT052 -> guillermomolina/protos#821
PLAT053 -> guillermomolina/protos#822
```

The currently available GitHub connector does not expose native Parent/Sub-issue
or native blocked-by mutation. Each child therefore includes textual
`Parent: #818` / prerequisite metadata, but GITHUB006/GITHUB009 require the
native relationships before hierarchy/dependency coordination can be called
fully reconciled. This limitation is coordination-only; it does not change the
technical audit result.

## Final result

```text
AUD019_B_STATUS=COMPLETE

PROTOS_REVISION=49e8763b68db1fe7583d36347fe5028fbe90d539
PROTOS_VERSION=0.3.258-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1

LATER_PROTOS_REVISION=08cfc69477ad45e2fe91a95140e5c71dffb9cdc0
LATER_PROTOS_VERSION=0.3.259-SNAPSHOT
AUD019_A_DRIFT=NON_MATERIAL

GENERIC_FOREIGN_VALUE_SUBSTRATE_STILL_VALID=YES
PER_LANGUAGE_IMPORT_PROVIDER_STILL_VALID=YES

REQUIRED_GUEST_DECISION_PACKETS=2
REQUIRED_PLATFORM_DECISION_PACKETS=2
MINIMUM_DECISION_COUNT=4

SECURITY_PACKET=SEPARATE
CALLBACK_PACKET=SPLIT_D_AND_PLAT

DECISION_DEPENDENCY_ORDER=
  D188
  -> D189
  -> PLAT052
  -> PLAT053

DEFERRED_NON_BLOCKING_QUESTIONS=
  universal async/Future/Promise bridging
  foreign cancellation
  retained async callbacks
  generic Actor/P foreign transfer
  provider plug-in API
  dependency acquisition
  full multi-language Native distribution

IMPLEMENTATION_AUTHORIZED_AFTER_B=NO

NEXT_WORK_ITEM=D188
NEXT_WORK_ITEM_ISSUE=819
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
