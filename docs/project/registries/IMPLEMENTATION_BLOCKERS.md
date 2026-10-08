# Protos Implementation Blockers

This file records implementation work that is blocked by unresolved normative
semantics. It is implementation state, not a normative specification.

Agents must re-check every blocker against the current normative specification
on the current `main` branch. The specification state recorded when a blocker
was created is historical context only.

Blocker states:

- `BLOCKED`: the normative unblock condition is not yet satisfied.
- `READY`: the current normative specification satisfies the unblock condition,
  but the blocked implementation work has not yet been completed.
- `CLOSED`: the dependency was resolved and the corresponding implementation
  work was completed or made obsolete.

Do not mark a blocker `READY` merely because a relevant specification file
changed. Verify the stated unblock condition against the current normative text.

## B001 — Empty Sequence execution

Status: CLOSED

Implementation area:
Truffle lowering / execution of a `CanonicalSequence` containing zero
expressions.

Normative dependency:
The normative execution semantics now state that a semantic `Sequence` containing
zero expressions completes normally with canonical `null`.

Specification authority:
- `spec/semantics/EXECUTION_AND_CONTROL.md`

Unblock condition:
The current normative specifications explicitly and uniquely determine the
result of evaluating an empty `Sequence`.

Current consequence:
Implemented. Empty `CanonicalSequence` lowering/execution completes normally
with the canonical `null` value, matching the normative contract.

Independent work:
Literal/value representation and all other execution work whose observable
semantics are already defined may continue. Internal host representation is an
implementation choice and is not, by itself, a normative blocker.

History:
B001 was temporarily broadened to "Runtime value materialization". That was too
broad: repository policy and the runtime specification explicitly permit
implementation-specific internal representations when observable Protos
semantics are preserved. The blocker is therefore narrowed back to the actual
unresolved observable case: the result of an empty `Sequence`.

## B003 — Delegation parent / lookup chain of canonical Boolean values

Status: CLOSED

Implementation area:
Standard prototype/delegation bridge for the canonical `true` and `false` runtime
representations, including ordinary member lookup and polymorphic invocation
through their delegation chains.

Normative dependency:
D027 closed the portable Core topology. Canonical `true` and canonical `false`
delegate directly to the unique root `Object`; Core v0.1 defines no standard
prelude binding, object, or prototype named `Boolean`.

Specification authority:
- `spec/semantics/OBJECT_MODEL.md`
- `spec/semantics/VALUES_AND_COLLECTIONS.md`

Unblock condition:
The current normative specification explicitly and uniquely determines the
delegation parent / ordinary lookup chain of canonical `true` and `false`,
including whether any standard Boolean prototype object exists.

Current consequence:
Implemented. The runtime value-lookup bridge maps both canonical Boolean host
singletons directly to `Object` for ordinary lookup. Inherited Object behavior
therefore preserves the original canonical Boolean receiver during dispatch,
and polymorphic invocation follows the same ordinary lookup path. No synthetic
or Protos-visible `Boolean` prototype is introduced.

Independent work:
Numeric and other value families whose standard prototype/delegation hierarchy
is already normatively closed may continue independently.

History:
B003 was originally `BLOCKED` while D027 still owned the unresolved parent
topology. Once D027 closed that topology, the blocker became logically `READY`.
This implementation completes the previously excluded Boolean bridge, so the
final transition is `BLOCKED -> READY -> CLOSED`.

## B002 — Delegation parent of `without` / `alias` result objects

Status: CLOSED

Implementation area:
Standard `Object.without(name)` and `Object.alias(sourceName, aliasName)` message
behavior and any runtime helper that constructs their result objects.

Normative dependency:
The normative object model requires both operations to return a new ordinary
object containing copied local-slot bindings, but it does not currently state
what delegation parent that result object has. The delegation parent is
observable through ordinary lookup and therefore cannot be chosen as an
implementation detail.

Specification authority:
- `spec/semantics/OBJECT_MODEL.md`

Unblock condition:
The current normative specification explicitly and uniquely defines the
delegation parent of the ordinary object returned by both `without(name)` and
`alias(sourceName, aliasName)`.

Current consequence:
Implemented. Runtime structural-view helpers now construct both results as fresh
open ordinary objects whose immediate delegation parent is the unique root
`Object`, copy only local bindings shallowly, preserve exact stored values, and
never inherit the receiver's structural state or parent.

Independent work:
Composition conflict validation, local-slot snapshots, atomic contribution
application, parser/canonical AST work, and unrelated execution/runtime work may
continue independently.

## B004 — Public Group/GroupRef acquisition and discovery API

Status: CLOSED

Implementation area:
I011 public ActorGroup acquisition through the exact Core v0.1 surface closed by D039.
Portable service discovery remains explicitly outside Core v0.1 and is not an I011 closure
requirement.

Normative dependency:
Specification revision `0.1.376` / D039 defines exactly
`Actor.group(firstMember, additionalMembers...) -> GroupRef`: one or more explicit ActorRefs,
synchronous fresh Group/GroupRef creation, caller-Process ownership, initial membership only,
communication-only GroupRef authority, and no Core name/identity lookup or discovery registry.
Section 72B explicitly keeps service discovery outside Core v0.1.

Specification authority:
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` §50 Runtime Groups
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` §50A Core ActorGroup Acquisition
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` §72B Service Discovery Implementation Is Not Core Semantics
- `spec/concurrency/ACTORS.md` §8 Core public Actor surface

Unblock condition:
Satisfied by specification revision `0.1.376` / D039. Two independent implementations
can now implement the same portable acquisition selector, argument domain, creation cutover,
ownership/lifetime, result identity, authority boundary, and explicit absence of Core discovery
without choosing new observable semantics.

Current consequence:
Implemented and published by I011-21. The frozen Core `Actor` object exposes exactly
`spawn`, `current`, and `group`; `Actor.group(firstMember, additionalMembers...)` validates the
complete ActorRef vector before cutover, establishes one Process-owned Group with set membership,
and returns one fresh GroupRef acquisition. Owning-Process termination terminates the Group
without stopping members. No public Group object, lookup/reacquisition selector, post-creation
membership/controller API, registry, discovery namespace, endpoint syntax, placement policy, or
transport-selection surface was introduced.

History:
B004 moved `BLOCKED -> READY` when specification revision `0.1.376` / D039 closed the exact
portable acquisition surface and explicitly excluded Core service discovery. I011-21 implements
that surface and its Process-owned lifetime integration, so the blocker is now `CLOSED`.

Independent work:
Optional service discovery, post-creation Group control/membership, desired-cardinality/controller
APIs, durability, explicit Group termination, placement policy, and richer distributed Authority
remain future extension/design work and do not block completion of the now-closed Core v0.1
ActorGroup acquisition requirement.


## B005 — `super` without a physical methodHome

Status: CLOSED

Implementation area:
I020-D execution/failure semantics for a syntactically valid `super.message(...)`
whose current activation has no physical `methodHome`, including execution in an
unbound role or after a semantic boundary that intentionally removes caller
method metadata.

Normative dependency:
Satisfied by specification revision `0.1.377` / D040.
`spec/semantics/EXECUTION_AND_CONTROL.md` §8 defines super validity as a
dynamic invocation property. After the ordinary caller-supplied argument/spread
vector completes, absence of `methodHome` signals one fresh standard
`InvalidSuper` occurrence and performs no lookup. A present `methodHome` with no
delegation parent instead has an empty super lookup search and signals
`SlotNotFound`.

Specification authority:
- `spec/semantics/EXECUTION_AND_CONTROL.md` §8 `super`
- `spec/semantics/ERRORS.md` for `InvalidSuper` parentage and fresh standard
  failure identity
- `spec/PROTOS_GRAMMAR.md` for the unchanged syntactic validity/scope of
  super-message-send
- `spec/semantics/CALLABLES.md` for the ordinary caller-supplied argument vector,
  the single Closure value kind, dynamic method role, and captured method metadata

Unblock condition:
Satisfied by specification revision `0.1.377` / D040. Independent
implementations can determine the exact validity point, argument-before-dispatch
precedence, absence of lookup on the invalid-context path, standard failure
category and freshness, and the distinct `SlotNotFound` result for an empty or
exhausted valid super lookup.

Current consequence:
Implemented by I020-D. Core source publishes `InvalidSuper` as a standard direct
child of `Error`, the frozen prelude exposes that exact prototype, and the typed
runtime Error factory creates one fresh occurrence per missing-`methodHome`
failure. The super-send execution path evaluates the complete ordinary
argument/spread vector before dispatch and then translates absent `methodHome`
into the standard Protos `InvalidSuper` control transfer without attempting
lookup or exposing the former host `IllegalStateException`. I020-A valid
method-bound receiver/methodHome behavior and `SlotNotFound` outcomes remain
unchanged.

History:
B005 moved `BLOCKED -> READY` when specification revision `0.1.377` / D040 made
the missing-`methodHome` behavior normative. I020-D implements that rule and
publishes focused Java plus language-level conformance, completing the transition
`READY -> CLOSED`. The earlier `InvalidSuper()` name in
`spec/runtime/ABSTRACT_RUNTIME.md` remains informative only and was not used as
independent normative authority.

Independent work:
No implementation work remains blocked by B005.

## B006 — Atomic package metadata replacement

Status: CLOSED

Implementation area:
Package-tool Filesystem Slice 2B and future package commands that safely publish
`protos.toml` or `protos.lock`.

Normative dependency:
Satisfied by specification revision `0.1.379` / D042 and the closed I021
production implementation. Core Filesystem supplies standard confined
`open`, `replace`, and `remove`; D042 preserves final-entry non-follow selection,
failure-atomic namespace visibility, commitment/cancellation semantics, and
non-recursive removal.

Closure evidence:
Package-tool Filesystem Slice 2B provisions one explicit confined project
Filesystem to the bundled Protos tool with three independent authority sets:
read access to `protos.toml`/`protos.lock`, write-only positioned `createNew`
access to the two exact staging names, and namespace mutation over those target
and staging entries. `self:MetadataPublication` performs content encoding,
standard File staging write/close, atomic `filesystem.replace(...)`, and explicit
`filesystem.remove(...)` discard in Protos code. An existing staging entry is not
silently removed or reused: `createNew` fails before the target is changed.

Current consequence:
B006 is CLOSED. Package metadata has a production-safe publication composition
without in-place target truncation, `PackageNative` filesystem helpers, ambient
host paths, or another package-only privileged mutation path. Future `add`,
`remove`, `resolve`, and `update` command policy may reuse this mechanism without
reopening the atomic-publication blocker. Full package-store/archive operations
remain outside B006 and still require their separately designed capabilities.

History:
B006 was introduced as BLOCKED because Filesystem initially exposed only `open`;
D041 / revision `0.1.378` defined atomic replacement/removal and moved it to
READY, D042 / revision `0.1.379` corrected final-entry selection, and I021-A/B/C
published the general runtime/backend/conformance. Package-tool Filesystem Slice
2B now completes the remaining write/staging-authority integration and closes the
blocker.

Library dependency:
None. B006 remains a general Filesystem capability/integration boundary and does
not allocate a `LIBxxx` item.

## B007 — Standard `while` protocol semantics

Status: CLOSED

Implementation area:
Standard Closure-specific `Object.while(body)` protocol, including validation,
ordinary Closure activation, strict Boolean decision, synchronous control
transfer, Future/task ownership composition, suspension/replay, cooperative
cancellation and bounded retained execution state.

Normative dependency:
Satisfied by D044 / specification revision `0.1.381`. During implementation,
B008 exposed one independent structured-ownership ambiguity; D045 /
specification revision `0.1.382` clarified that structured ownership is scoped to
the enclosing asynchronous task execution rather than every synchronous
activation.

Specification authority:
- `spec/semantics/EXECUTION_AND_CONTROL.md` §17 `Iteration and Loops`;
- `spec/semantics/CALLABLES.md` for ordinary `Object.while` placement,
  Closure-family receiver domain, lookup/extraction/shadowing and activation;
- `spec/semantics/VALUES_AND_COLLECTIONS.md` for canonical Booleans;
- `spec/semantics/ERRORS.md` for Error/control transfer;
- `spec/concurrency/FUTURES_AND_TASKS.md` for suspension, cancellation and
  D045 task-scoped structured ownership;
- `spec/PROTOS_GRAMMAR.md` for the unchanged ordinary call/trailing-Closure
  syntax.

Unblock condition:
Satisfied by D044. Independent implementations can determine receiver/body
validation and order, exact callback activation timing, strict `true`/`false`
decision, canonical completion, control-transfer behavior, Future-result
composition, suspension/replay and cancellation composition without inventing a
loop-specific scheduling or ownership rule.

Current consequence:
Implemented and published by I023-A/B/C/D. The reference runtime exposes the
ordinary inherited `Object.while` selector, retained Protos conformance covers
the complete synchronous and asynchronous interaction surface, replay state is
bounded across completed and repeatedly suspending iterations, and the final
Core native-boundary audit remains 111 construction sites across 30 providers.
B008 is also CLOSED under D045.

The Programming Guide control-flow slice DOC001-E was subsequently published
and is now CLOSED. That documentation closure remains separate from the earlier
I023/B007 implementation closure; chapter 04 records the programmer-facing
explanation while the normative specification remains authoritative.

Independent work:
No implementation blocker remains for the standard `while` protocol. Future
library/documentation work may rely on the published behavior, subject to its
own dependency and current-main audits.

History:
B007 began BLOCKED because the old language description did not uniquely define
a portable standard loop protocol. D044 resolved that ambiguity and moved B007
to READY. I023-A published the selector/runtime cutover; B/C then closed replay,
validation/control-transfer, Future ownership, suspension and cancellation
composition. The unpublished per-synchronous-activation ownership experiment was
rejected after it broke ordinary Future-shaped APIs; B008/D045 resolved that
general ambiguity instead. I023-D performs the final cross-slice and architecture
audit and closes B007.

## B008 — Structured ownership when a task-backed Future escapes an activation

Status: CLOSED

Implementation area:
I023-B2D2 structured-ownership/cross-B2 closure and any implementation/conformance
work that depends on the lifetime rule for task-backed Future-producing work
created inside an ordinary synchronous activation.

Normative dependency:
Satisfied by D045 / specification revision `0.1.382`.

`spec/concurrency/FUTURES_AND_TASKS.md` now defines structured ownership at the
current asynchronous task execution scope rather than at every synchronous
Closure/method invocation. Synchronous nested activations do not create implicit
concurrency scopes. A task-backed Future may therefore be returned pending from an
ordinary synchronous invocation without waiting, detaching, transferring,
re-parenting, or otherwise changing its ownership edge. The edge remains owned by
the same surrounding task-scoped execution context until terminality or explicit
`Future.detach()`.

A distinct asynchronous child task establishes the scope that owns work created by
that child. When an owning asynchronous computation itself reaches otherwise
normal terminal completion, it waits for all remaining non-detached task-backed
children to become terminal without implicitly observing their result. Adoption
continues to transfer outcome only and never ownership. P and `Future.then()` use
the same general rule.

D044 therefore composes without a loop special case: a `while` condition/body
activation is synchronous and does not establish a structured scope. A normal
Future body result is ignored, the next loop step is not delayed merely by that
returned Future, and `while` does not await/adopt/flatten/cancel/detach/re-parent
it. Any task-backed child remains owned by the enclosing task-scoped execution
context under the ordinary rule.

Specification authority:
- `spec/concurrency/FUTURES_AND_TASKS.md` §§27, Future `then()`, 23 and 24;
- `spec/concurrency/PARALLEL_EXECUTION.md` for P result-Future composition;
- `spec/semantics/EXECUTION_AND_CONTROL.md` for D043/D044 `ensure`/`while`
  composition;
- `spec/semantics/CALLABLES.md` for ordinary synchronous Closure activation.

Unblock condition:
Satisfied. Two independent implementations can now determine the same ownership
and ordering without escape analysis or API-specific inference:

1. the structured owner is the current asynchronous task execution scope, not each
   nested synchronous activation;
2. synchronous return neither waits nor changes the ownership edge;
3. `then`, P, `detach`, adoption and Future-shaped library APIs compose under one
   general rule;
4. task-backed work is treated identically whether or not its Future is returned,
   stored or wrapped;
5. D044 `while` adds no loop-specific ownership/scheduling behavior.

Current consequence:
Closed by I023-B2D2 conformance after D045. The reference runtime already used
the task-scoped ownership model selected by D045, so no production/runtime
ownership change was required. A `while` body can return a newly-created
task-backed Future and the synchronous body/loop continues without draining that
child; the surrounding asynchronous task still retains the child and does not
become terminal until every non-detached child is terminal. The retained
Future-shaped JSON overlap conformance also remains green, guarding against the
rejected per-synchronous-activation drain interpretation.

I023-B2D2, B2D, B2 and B are CLOSED. I023-C is READY. B007 remains READY until
final I023-D closure.

History:
B008 was created after an unpublished activation-drain experiment broke the
existing Future-shaped JSON overlap contract, proving that per-synchronous-
activation draining was observable. D045 resolves the ambiguity by making the
already-composable task-scoped model explicit rather than adding Future-return
escape transfer, implicit detach, or per-result ownership heuristics.

Independent work:
B008 no longer blocks implementation work. I023-C may proceed; unrelated work
remains independent. B007 remains READY until final I023-D implementation
closure.

## B009 — Portable Filesystem tree observation for immutable package verification

Status: CLOSED

Implementation area:
`I024 — Filesystem directory observation + captured-tree capability`, followed by
`TOOL001-F2E2` verified read-only package-store binding.

Normative dependency:
Satisfied by D046 as amended by specification revision `0.1.384`.

D046 adds exactly the general capability boundary required by the closed F2E1
ContentIdentity contract:

```text
filesystem.entries(path)     -> Future<Array>
filesystem.captureTree(path) -> Future<Filesystem>
```

`entries` exposes exact direct-child names plus no-follow entry kind as inert
frozen descriptor data and deliberately resolves with one complete eager Array
for the selected directory in Core v0.1. `captureTree` returns a fresh immutable
read-only Filesystem containing the recursively captured structure/regular-file
bytes and never follows captured child links. No standard Directory/DirectoryEntry
identity is introduced. The captured Filesystem itself carries no
programmer-managed close/release obligation; implementation-managed immutable
backing remains behind the Filesystem semantic boundary.

The capture is intentionally not an atomic point-in-time snapshot of a mutable
source. The returned captured Filesystem itself is the stable logical tree.
Verify-then-use code validates and later consumes that same capability,
eliminating the source-tree TOCTOU re-read.

Specification authority:
- `spec/io/FILESYSTEM.md` section 20.4 / D046;
- `spec/io/IO_CORE.md` for Future-shaped I/O failures;
- existing Path/File/Filesystem authority, confinement and cancellation rules.

Objective unblock condition:
Satisfied. Two independent implementations can now determine exact direct-child
observation, eager complete-result behavior, no-follow kind classification,
name/case/uniqueness behavior, confinement, stable captured-tree authority, the
absence of a captured-Filesystem caller-managed close obligation,
concurrent-source semantics and Future/cancellation/error behavior.

Current consequence:
B009 is CLOSED by I024-D. I024-A/A2/B/C/D now publish the complete general D046
surface and integrated Protos-visible conformance: eager exact-name/no-follow
`entries`, immutable read-only `captureTree`, source-independent captured bytes,
ordinary Future cancellation/failure behavior, and no public Directory or
captured-Filesystem close/release obligation. The final architecture guard
confirms that Filesystem still uses one audited native-Closure construction helper.

`TOOL001-F2E2` is therefore READY. It must consume this general capability to
capture one already-selected store root, verify the closed ContentIdentity
contract against that captured Filesystem, and pass that same immutable authority
forward. Package-specific Java/NIO traversal remains prohibited.

Independent work:
F2E1 remains CLOSED; package acquisition/network/store-write remain independently
scoped; F2E3/F2E4/F2E5 remain gated behind F2E2.

## B010 — Shared root `Object` structural mutability under Process hosting

Status: CLOSED

Implementation area:
`I026-A4B2B3` concurrent Core-root publication and the closure gate for
`I026-A4B2B` / `I026-A4B2` before `I026-A4B3` may become READY.

Normative dependency:
Satisfied by D049 / specification revision `0.1.389`. The standard shared-object
publication rule now requires every physically shared standard object whose
ordinary structural state is observable to be published `FROZEN` before guest
observation and to remain frozen for the sharing interval. The unique standard
root `Object` is explicitly covered; shared standard Closures independently obey
the same semantic-immutability boundary.

Specification authority:
- `spec/semantics/OBJECT_MODEL.md` for the standard root publication state and
  ordinary frozen-object mutation behavior;
- `spec/semantics/MODULES.md` for the general shared-prelude immutability rule and
  object-by-object shallow publication boundary;
- `spec/concurrency/ACTORS.md` for Actor isolation.

Unblock condition:
Satisfied. Two independent implementations can determine that the shared root
`Object` is `FROZEN` from first guest-observable access, that no guest-visible
root mutation may cross Actor boundaries, that freezing remains shallow, and
that every other physically shared standard object must independently be
semantically immutable. No hidden overlay, per-Actor root identity or global
mutable singleton is required.

Current consequence:
`I026-A4B2B3` is CLOSED in `0.2.280-SNAPSHOT`. A publishes the standard root through one bounded
atomic frozen cutover and seals each shared standard graph before Prelude exposure.
B publishes the approved A+ executable split: globally shared source-backed root
Closures retain their semantic/template identity while each entered
`ProtosLanguageContext` owns distinct language-bound plans and CallTargets. Two
Process Contexts on one Engine execute the same shared root behavior concurrently
and overlap without a global guest lock; neither projected plan is written back to
the shared Closure. The blocker's full semantic and executable-layer exit condition
is therefore satisfied. A4B2B/A4B2 are CLOSED and I026-A4B3 is READY.

History:
B010 was introduced when A4B2B3 exposed that a bootstrap lock could remove a
publication race without resolving guest-visible mutable root sharing. D049
selects the shared-frozen-standard-object model after explicit project-owner
approval and moves the blocker from `BLOCKED` to `READY`.

Independent work:
Published A4B2A/A4B2B1/A4B2B2 hosting and carrier-routing evidence remains
valid. `I026-D` and unrelated implementation work may continue independently.
`ContextPolicy.REUSE` and `ContextPolicy.SHARED` remain deferred platform
optimizations and are not required by D049.

## B011 — Standard Polyglot embedding default Filesystem authority and base

Status: READY — approved Alternative A published in spec 0.1.451; runtime Filesystem integration pending

Implementation area:
I086 / PLAT054 standard `Context.newBuilder("protos")` embedding,
bootstrap-local `filesystem` capability provisioning and host-restricted
Filesystem operations. Existing embedded Process bootstrap passes no
Filesystem capability; the embedded guest therefore receives no default
`filesystem` slot. This must not be interpreted as permission to weaken
host filesystem restrictions.

Normative dependency:
`spec/io/PROCESS_IO.md` requires the slot only when the host grants a
default Core Filesystem and requires the capability to remain bounded by
the host's Polyglot Context authority. `spec/io/FILESYSTEM.md` §20
defines a confined Filesystem namespace rooted at an authority base, but
specification **0.1.450 now selects** the effective host-granted file-I/O
mapping and the authorized Context working directory as initial Filesystem
base. The remaining open observable choice is **which fail-closed outcome**
applies when file access is granted but that base cannot be represented and
confined: fail the initial bootstrap, or continue without the `filesystem`
slot. Neither outcome has been approved as mandatory.

Specification authority:
- `spec/io/PROCESS_IO.md`, Standard Polyglot embedding bootstrap and authority
  / Root filesystem capability provisioning;
- `spec/io/FILESYSTEM.md` §20, Filesystem confinement, Path and base;
- `spec/io/IO_CORE.md`, I/O and resource/cancellation contracts;
- PLAT054 ratification record for the explicit host-authority ceiling.

Objective unblock condition:
The effective permission and base are normatively published in `0.1.450`.
The remaining objective unblock gate is an owner-approved and normatively
published selection of the **unsafe/unrepresentable base** consequence
(bootstrap fails vs initial `filesystem` absent), if one uniform outcome
is required before enabling this product path. The approved design must preserve the
configured Context filesystem restrictions without allowing the
runtime's existing `java.nio`-backed `ProtosNio*` operations to bypass
them. A `TruffleFile`-respecting or demonstrably equivalent confined
backend is **implementation work after** this decision, not a substitute
for the authority decision.

**Owner-decision update — 2026-10-08 (PLAT054-3E1):** the project owner explicitly approved HOST-FS-1/FS-2, published as a non-normative decision and provenance record in [PLAT054-3E1](../evidence/PLAT054/PLAT054_3E1_HOST_SUSPENSION_AND_AUTHORITY_OWNER_APPROVAL.md). Effective Polyglot file-I/O authorization (including applicable custom provider/restrictions) is the authority source; base = the Context's effective authorized working directory; failure to prove a safe base is **fail-closed** with no unrestricted NIO fallback. The owner **did not select** whether an unprovisionable safe base aborts bootstrap or simply omits the default slot. **Historical at 0.1.449:** status BLOCKED pending normative publication. As of 0.1.450 the general permission/base contract **is** published; only the above fail-closed outcome remains unselected. Do not claim a completed backend or grant.

Current consequence:
I086-3D at `guillermomolina/protos@e0bb880584908f65cb78e66896eba334f3d5e13b`
published Actor carriers and Context close without granting a default
Filesystem. No `filesystem` capability is provisioned in the standard
embedding; no positive host filesystem grant is claimed.

Independent work:
I086 Future.value host-entry suspension attribution, Network grant
decision (B012), I087 application-module resolution, native validation
and existing Actor/lifecycle regression work may advance separately.

**Publication checkpoint — 2026-10-08, spec 0.1.450:** [exact commit and verification](../evidence/I086/I086_PLAT054_3E1_NORMATIVE_HOST_ENTRY_AND_AUTHORITY_PUBLICATION.md). The published rule expressly leaves the two fail-closed outcomes open; **B011 remains BLOCKED only for this narrow observable choice**. Independent confined `TruffleFile` adapter investigation/planning is permitted, but do not select a guest-visible policy or publish a positive grant path depending on it. No completed Filesystem backend or Native gate is claimed.

**Coordination checkpoint — after PLAT054-3E2:** host-entry `Future.value()` is implemented in [`protos@55f06a29`](../evidence/I086/I086_PLAT054_3E2_SUSPENDIBLE_HOST_ENTRY_FUTURE_PUBLICATION.md); it no longer blocks this Filesystem area. B011 is **still BLOCKED** only for the unselected unsafe-base consequence. Slice PLAT054-3E3 is deferred; independent B012 Network implementation goes next as 3E4. No positive Filesystem host grant or Native validation is implied.

**PLAT054-3E3-FS0 exact A approval — 2026-10-08:** the owner expressly selected **“evidentemente apruebo A”** after the GITHUB010 investigation: for an effective File I/O grant whose configured provider/Context working-directory base cannot be safely represented/confined, **abort initial Process bootstrap before any guest module source expression** and report an explicit host bootstrap provisioning failure. **Do not** replace that failure with an absent `filesystem` slot, permit NIO/provider bypass, broaden the base or introduce a new guest Error category. Without effective authorization, the slot is still absent; a valid virtual logical base is acceptable. PAY AS YOU GROW continues to prohibit eagerly starting unused FS operation resources. The approved alternative is precisely recorded in [PLAT054-3E3-FS0 decision evidence](../evidence/PLAT054/PLAT054_3E3_FS0_FILESYSTEM_BOOTSTRAP_ABORT_OWNER_APPROVAL.md) and the [PLAT054 decision](../decisions/platform/PLAT054_STANDARD_POLYGLOT_EMBEDDING_CONTRACT.md); GITHUB021 invariant reconciliation PASS.

**Present B011 status: BLOCKED — EXACT OWNER DECISION APPROVED; PRODUCT SPEC PUBLICATION PENDING.** Approval and documentation publication alone do not satisfy the mandatory normative gate: product spec `0.1.450` still explicitly leaves A/B unspecified. Next `PLAT054-3E3-SPEC` is `TYPE=IMPLEMENTATION` in `guillermomolina/protos`, human-executor publication of the narrow `spec/io/PROCESS_IO.md` paragraph and the next current-HEAD-derived `spec/PROTOS_SPEC_CHANGELOG.md` revision; do not start the positive Filesystem guest-authority backend first. Once the exact product normative commit is published/re-read, B011 may move to READY for the coherent Filesystem backend slice; mark CLOSED only upon implementation publication. I086 remains OPEN for this backend and final Native/portable acceptance; B012 stays CLOSED. This new checkpoint supersedes the earlier statement above that no option was selected, without rewriting the historical snapshot.

**Normative publication checkpoint — 2026-10-08 (PLAT054-3E3-SPEC):** B011's owner-approved Alternative A was published and verified in `guillermomolina/protos@1d6d537d79d69fae3ad1e613cefd76b7a97abe38`, global spec **`0.1.451`**. The exact product commit changes only `spec/io/PROCESS_IO.md` and `spec/PROTOS_SPEC_CHANGELOG.md`, replacing the formerly open A/B outcome with mandatory initial bootstrap failure for an effectively authorized but unsafe/unrepresentable provider-confined base, before guest source expression execution. An absent authorization still yields an absent slot. No guest public Error family or broader fallback is introduced; virtual providers and PAY AS YOU GROW remain supported. The maintainer reports all local tests PASS, clean `git diff --check`, and push; the coordinator verified product GitHub publication but did not execute tests. See [immutable publication evidence](../evidence/I086/I086_PLAT054_3E3_SPEC_BOOTSTRAP_ABORT_NORMATIVE_PUBLICATION.md).

**CURRENT B011 = READY FOR IMPLEMENTATION**, replacing the earlier historical BLOCKED statements above. `READY` recognizes approved **and normatively published** semantics; there is no completed guest Filesystem runtime grant yet (the embedded bootstrap still passes `null` for default FS at this commit). Next `PLAT054-3E3-FILESYSTEM` is one grouped implementation slice in `guillermomolina/protos` under I086/#840, with human-run build/tests/version and publication. Transition B011 to **CLOSED** only on verified implementation publication, not from this spec release. B012 remains CLOSED; I086 Native Image/portable final validation remains independent and UNVERIFIED.

## B012 — Standard Polyglot embedding default Network grant and thread authority

Status: CLOSED — spec 0.1.450 published and default embedding Network implemented at protos@f8f4ebe2

Implementation area:
I086 / PLAT054 default `network` capability provisioning in the
standard Polyglot Context and compatibility of its asynchronous
Network backend with Context thread restrictions. The default
embedding supplies no Network capability.

Normative dependency:
`spec/io/PROCESS_IO.md` requires `network` only when the host
explicitly grants Network authority, and `spec/io/NETWORK.md`
owns capability behavior. The approved standard-embedding platform
record establishes **deny by default**, but now defines the mapping in **spec 0.1.450**: the *effective* Context
socket authorization after restrictions is an explicit grant for the initial,
host-bounded TCP-only `network` capability. Thread creation is independent;
no guest code may run in unauthorized hidden workers. A broad grant counts
only insofar as effective socket access is allowed.

Specification authority:
- `spec/io/PROCESS_IO.md`, Standard Polyglot embedding bootstrap and authority
  / Root network capability provisioning;
- `spec/io/NETWORK.md`, Network capability contracts;
- `spec/io/IO_CORE.md`, asynchronous I/O/cancellation/resource ownership;
- `spec/concurrency/ACTORS.md` and PLAT054 for Context-local thread/lifetime boundaries.

Objective unblock condition:
**SATISFIED by product spec 0.1.450**, published at
`guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`.
The implementation acceptance now requires an actual backend preserving
effective socket authority, independent guest-thread policy, Process/Context
custody, termination cleanup and zero unused Network resources. Native and
portable acceptance remain unverified.

**Owner-decision update — 2026-10-08 (PLAT054-3E1):** the project owner explicitly approved HOST-NET-1/NET-2 in [PLAT054-3E1](../evidence/PLAT054/PLAT054_3E1_HOST_SUSPENSION_AND_AUTHORITY_OWNER_APPROVAL.md). Effective Polyglot socket authorization is the default Network grant condition; scope remains host-bounded Protos TCP; thread-creation authorization is **independent** and not a prerequisite for the Network slot. The backend must satisfy the Context's effective thread policy and Process-close custody, without unapproved guest workers. **Historical at 0.1.449:** BLOCKED awaiting normative publication. That publication is now complete in 0.1.450. **READY now means implementation authorized, not implemented or validated.**

Current consequence:
I086-3D at `guillermomolina/protos@e0bb880584908f65cb78e66896eba334f3d5e13b`
keeps the default Network slot absent. The current NIO poller
creates host Java threads, so `Env.isSocketIOAllowed()` alone is
not an implementation-ready permission/custody model.

Independent work:
Filesystem grant decision (B011), I086 host Future.value
classification, embedded Actor/lifecycle work already published,
I087 non-standard app module bootstrap and Native Image validation
can continue.

**Publication checkpoint — 2026-10-08, spec 0.1.450:** [exact commit and verification](../evidence/I086/I086_PLAT054_3E1_NORMATIVE_HOST_ENTRY_AND_AUTHORITY_PUBLICATION.md). **B012 is READY for implementation**. The current `ProtosNioHostIoPoller` still starts a raw Java thread and lacks embedded-Context-specific lifecycle integration; no positive Network grant or passing Native gate is claimed by the spec-only publication.

**Coordination checkpoint — after PLAT054-3E2:** host-entry Future suspension was published at [`protos@55f06a29`](../evidence/I086/I086_PLAT054_3E2_SUSPENDIBLE_HOST_ENTRY_FUTURE_PUBLICATION.md); B012 is **READY**, but the embedded guest still receives no default Network because `ProtosEmbeddedProcess.bootstrap()` passes `null`. Next slice **PLAT054-3E4 / IMPLEMENTATION** integrates effective host socket permission, bootstrap-local Network capability, lazy Context-owned backend and correct close/thread custody with the complete affected regression tests. Do not mark B012 CLOSED until the product implementation is actually published and verified. B011 remains independent and BLOCKED.

**Final B012 closure checkpoint — 2026-10-08 (PLAT054-3E4):** **CLOSED** under the blocker registry's rule: the normative authorization and thread/lifecycle choices were published in spec `0.1.450`, then corresponding embedded Network implementation was published at [`guillermomolina/protos@f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2`](https://github.com/guillermomolina/protos/commit/f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2), product `0.3.294-SNAPSHOT`. See [durable I086-3E4 evidence](../evidence/I086/I086_PLAT054_3E4_DEFAULT_NETWORK_POLYGLOT_PUBLICATION.md). `ProtosEmbeddedProcess.bootstrap()` uses `Env.isSocketIOAllowed()` and provisions `ProtosEmbeddedNetworkCustody` and the bootstrap-local `network` slot only with effective socket permission; permission does not inspect file or guest-thread policy. Custody lazily creates a host-only NIO poller on first TCP acquisition, releases and revokes all Network resources on Process termination/Context close/disposal, and prevents poller creation after close. New conformance/race tests are published. Maintainer reports all local tests PASS and clean `git diff --check`, but coordinator has not run or inspected logs. Native Image/portable final acceptance remains an I086 issue-level open gate, **not a reason to keep B012 blocked**. The earlier `READY`, `null` default and "next 3E4" statements above are historical snapshots superseded by this closure.

