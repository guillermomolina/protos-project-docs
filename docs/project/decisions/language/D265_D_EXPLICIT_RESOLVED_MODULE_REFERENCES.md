# D265-D — Explicit resolved module references

Date: 2026-11-26.
Decision: [D265 #1193](https://github.com/guillermomolina/protos/issues/1193).
Consuming implementation: [I180 #1199](https://github.com/guillermomolina/protos/issues/1199).
Trigger: [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12).
Research and approval review product baseline: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70).
Project-record review baseline: `78a961d87374219ef32c017653e73acef627af63`.

**Status: D265-D explicitly selected by the owner; bounded design approval recorded. Normative publication and remaining exact-contract checks pending.** This is a durable non-normative language-decision record, not a claim that the product implements the selected design.

## Owner approval and scope

In the active D265 investigation on 2026-11-26, the agent presented D265-D after comparative research, then summarized the recommendation at the owner's request: explicit opaque immutable ModuleRef, resolution without execution in the acquiring application's context, Actor-local loading, explicit delegation of access to a particular private module, preservation of existing String APIs, reuse of the current loader, no HTTP exception, initially only same-Process Actor.spawn transfer, ordinary messaging and cross-Process distribution deferred. The summary explicitly stated that representation feasibility must be verified before implementation. The owner responded: "ok apruebo la recomendación." The same message authorizes bounded commit and push of evidence here and allocation of implementation work as Ixxx. This is approval of the presented candidate, not of unrelated future semantic refinements or an unverified Java representation.

The full candidate also limits initial acquisition to source-backed MODULE and SOURCE targets, not FOREIGN, and uses the existing authorized resolution context. Recognition of genuine references is needed; the suggested `import.isReference(value)` spelling in the longer research is not an excuse to invent an additional Core family or public constructor. Preserve ordinary object protocol and existing syntax. Examples of the selected API direction:

```protos
ref: import.resolve("self:Prices")
module: import(ref)
actor: Actor.spawn(ref, "start")
host: Host.owning(listener, ref, "handle", workers: 4)
```

The analogous Host.owningWithFactory path accepts the reference as handler module argument with existing binding, initialization payload and named configuration. No options object is introduced. Existing String behavior remains unchanged: passing a relative String to Host still interprets it in the Worker context; explicit resolution by the application is the composition mechanism.

## Problem and inspected implementation

WEB016-A3 reported installed Protos 0.4.192-SNAPSHOT on Linux x86_64: direct package import, callable lookup, Router HTML/404, direct Actor.spawn("self:Prices","start") request and termination passed. Host startup/drain with a globally resolvable std:uri module passed; this did not prove real HTTP responses. Host.owning with self:Prices failed at ready() with startup HostError. I154's Native CLI smoke exercised direct Server.serveOne.owning, not this package Host case.

At the exact baseline, [host_worker/module.protos](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/http/internal/host_worker/module.protos), ownCallable at lines 286–307, imports the handler String under the std: Worker module's lexical identity. Both callable and factory bootstrap paths share this defect. The package resolver requires a package importer for self: and resolves dep: against its exact typed declaring node and alias edge. SOURCE relative paths are resolved against their actual importing source and confined to their owner. There is no ambient root alias.

In contrast, ProtosModuleRuntime.resolveModuleKey uses caller.currentModuleKey(), and ProtosStandardActorProtocol already resolves the bootstrap String in the creator before Actor creation cutover and loads its canonical key in the destination. The source-backed policy in protos/lib/core/import.protos uses its private bridge resolve/lookup/begin/execute. ProtosStandardImportProtocol's private ResolvedModuleToken is bound to runtime, Actor state and execution domain; pending tokens additionally bind Task and single-claim execution. These are not transferable public values and must not be weakened or exposed. Existing authorization including std:inspect is enforced before cache access. Resolution does not imply source existence, initialization, cache insertion, or immutable bytes in mutable workspaces/direct-file execution.

The general defect is loss of the canonical target when a context-dependent spelling crosses a reusable-library boundary. String spelling, canonical module identity and Actor-local instance are three distinct concepts.

## GITHUB021 invariant and delta review

Applicable normative authorities at the baseline:
- [MODULES.md](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/semantics/MODULES.md): resolution, canonical identity, cache/lifecycle and Actor-local module state.
- [ACTORS.md](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/concurrency/ACTORS.md): creator-side resolution, exact own bootstrap binding, creation transfer, no implicit authority, Actor readiness and communication.
- [DISTRIBUTED_RUNTIME.md](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/concurrency/DISTRIBUTED_RUNTIME.md): placement/admission after cutover; no public placement control.
- [HTTP.md](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/io/HTTP.md) §15: Host listener custody, Worker quotas and readiness, factory initialization, drain/abort and failure classification.

| Existing decision or invariant | Selected delta and preservation |
|---|---|
| D053 #157 exact inert V2 resolution graph | Preserve exact typed target, alias edge, version/export authorization and content identity policy. No capability placed in package plan. |
| D242 #1060 MODULE own exports, private initializer; SOURCE visibility | Preserve name-based export checks at acquisition and ordinary MODULE/SOURCE visibility. Explicit new delta, presented and owner-approved: a holder can delegate access to one authorized private target. This does not grant arbitrary package lookup or private initializer slots. SOURCE target exposes its existing SOURCE context, not a new export filter. |
| ACTORS creator-side bootstrap and atomic transfer | Add genuine references to bootstrap and creation-argument admission only; no Closure, Future, context, resource or module-instance transfer. Same Process and authorized resolution context. String spawn retains existing distributed behavior. |
| D225 #992 Actor/Group service and capability separation | No implicit communication or service grant, no automatic proxy or transfer by ordinary messages/GroupRef. |
| D216 #964 Host custody and Worker independence | Preserve named configuration, pre-custody structural failure, post-custody startup error and cleanup, exactly W Workers, delegated TcpAcceptor only, quotas, independent accepts, factory once after initialization request and before Host ready, Actor READY distinct, drain and abort. |
| I154 #1076 explicit service/network grants | No CLI or embedding privilege enlargement. Destination authorization still checked, even on cache hit. |
| PLAT075 #1124 / I167 #1125 source-backed import; PLAT025 exact private execution identity | Preserve private token checks, C-prime and continuation ownership, cache-before-execute, partial identity on cycles, exact failed-initialization eviction/retry; no rollback of completed side effects. Protos-first policy and irreducible private host operations only. |

No ambient self:, caller-stack search, implicit RootActor or module privilege, or global registry is approved. A public reference is target-specific load authority, not a context-wide resolver or raw canonical key. Acquisition applies existing resolution authorization; subsequent use preserves destination-specific grants, including restrictions on root-only services and foreign facilities. Two references to one canonical key need not be the same reference object; importing either within an Actor targets the same active module instance. Do not equate reference identity with module identity or promise identity preservation across snapshots without settling the exact transfer contract.

The owner expressly approved the identified private-target delegation delta and same-Process boundary in the summary; no hidden change to existing String resolution, Actor locality, or module export structure is selected. GITHUB021 consistency is satisfied for this bounded selection, subject to not treating unresolved representation/placement/error details as already approved.

## Comparative research (GITHUB010)

Reviewed mechanisms from several language families; they are evidence, not Protos authority:

- Python [resolve_name](https://docs.python.org/3/library/importlib.html#importlib.util.resolve_name), find_spec and import_module distinguish contextual spelling from loading; find_spec may import parents. [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) spawn starts a fresh interpreter, not a shared live callable environment. GraalPy [importlib bootstrap](https://github.com/oracle/graalpython/blob/master/graalpython/lib-python/3/importlib/_bootstrap.py), _resolve_name and _find_and_load, and [ImpModuleBuiltins](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/ImpModuleBuiltins.java), CreateBuiltin/ExecBuiltin and dynamic counterparts, establish guest policy/private host mechanism separation. They do not prove an equivalent transferable code-authority representation.
- [Node ESM](https://nodejs.org/api/esm.html#importmetaresolvespecifier) resolves with module provenance, and [Worker threads](https://nodejs.org/api/worker_threads.html) accept entry URLs and structured-cloned data; functions are not cloned. URL identity is not a Protos package authority.
- [Erlang processes](https://www.erlang.org/doc/system/ref_man_processes.html) spawn(Module, Function, Args) separates entry identity from arguments, but node code loading is not Protos Actor-local initialization.
- [Dart Isolate.spawnUri](https://api.dart.dev/dart-isolate/Isolate/spawnUri.html) contrasts explicit entry identity with closure capture and independent isolate execution.
- [Racket module names and loading](https://docs.racket-lang.org/reference/Module_Names_and_Loading.html) distinguishes module path indexes, resolved paths and namespace instances; resolution is not instance transfer.
- [Endo Compartment](https://github.com/endojs/endo/tree/master/packages/ses) and module resolve/import hooks distinguish referrer-relative resolution and compartment authority; do not copy broad resolver hooks or grant arbitrary private access.

Candidates: A status quo (application-owned direct Server loop or embedding app: catalog); B application-owned bootstrap composed around ordinary Actor.spawn; C contextual resolver capability; D explicit resolved module reference (selected); E narrower module-plus-binding EntryRef. Reject ambient root aliases, stack lookup, global name scheme, stringified internal keys, shared module/Closure, global registry and HTTP-only resolver: these lose identity/authority, violate locality, or duplicate the general mechanism.

Scores 1–5, each with a concise rationale. Confidence high for conceptual constraints, medium for unproven implementation/portability, low for unmeasured runtime cost; totals are review aids rather than authority.

| Dimension | A | B | C | D | E |
|---|---|---|---|---|---|
| Correctness | 2 no general fix | 4 preserves creator | 3 broad authority | 5 exact target | 5 exact entry |
| Protos alignment | 4 existing | 5 ordinary composition | 3 extra context facility | 5 general mechanism | 4 separate callable model |
| Present proportionality | 2 duplicate Host | 3 bootstrap burden | 2 oversized grant | 5 one target | 4 bounded but narrower |
| Incremental growth | 2 workaround ceiling | 3 coordination extensions | 5 flexible | 5 extend transfer later | 3 more reference kinds |
| Future resilience | 2 recurring issue | 4 ordinary pattern | 4 flexible but broad | 5 separates identity | 4 useful narrow grant |
| Scalability | 3 duplicate orchestration | 4 independent Workers | 4 no central actor required | 5 independent local loads | 5 independent entries |
| Simplicity | 5 no Core change | 3 more coordination | 2 broad resolver contract | 4 one new concept | 3 extra invocation semantics |
| Portability | 4 existing | 5 ordinary protocols | 4 resolver abstraction | 5 identity abstraction | 5 entry abstraction |
| Resource cost | 5 no new value | 4 duplicated bootstrap | 3 contextual retention | 4 one resolution plus reference | 4 one entry reference |
| Failure/operability | 2 repeated failures | 3 coordination | 3 provenance complexity | 5 explicit boundary | 4 entry failures |
| Deferral/migration | 3 workaround rewrites | 4 reusable pattern | 2 broad grant hard to retract | 4 opt-in migration | 4 narrower migration |
| Evidence/risk | 5 existing | 4 existing primitives | 2 unproven authority | 3 representation pending | 3 representation pending |
| Total / 60 | 39 | 46 | 37 | 55 | 48 |

A and B remain escape paths without new semantics but do not solve ordinary reusable Host composition without duplication or additional coordination. C pays for arbitrary resolution today and creates harder-to-audit private access. E may be valuable if later demand calls for binding-limited delegation, but now requires parallel load/invocation rules and does not solve general module loading. D lets simple programs continue using Strings and ordinary import unchanged. It can resolve once for W Workers, each initializing its own module; no central actor, new per-request lookup, shared instance or global registry is required. Actual allocation, synchronization and compilation costs remain unmeasured. Same-Process restriction avoids speculative cross-Process wire format, trust, deployment, code revision and distributed placement machinery. Later extension is possible only through explicit design. Deferral of D would continue library-local resolution failures and application-specific bootstrap rewrites; deferring broader transfer does not block the motivating Host case.

Strongest objection: D adds a first-class load capability and relaxes the package-name access boundary by explicit delegation; a bad representation might accidentally carry Actor state, permit forgery, bypass authorization or add unproven Java infrastructure. The narrower EntryRef or caller-owned bootstrap would reduce some exposure but at the cost of generality and/or duplicated coordination. The selection is contingent on preserving authority and proving an appropriate existing Truffle value/transfer pattern; precedent above is not sufficient proof. If that cannot be done without changing the selected semantics, return to D265 rather than invent infrastructure.

## Normative and implementation checkpoints

Before production implementation, reconcile MODULES, ACTORS, HTTP and any affected VALUES_AND_COLLECTIONS / DISTRIBUTED_RUNTIME owners and PROTOS_SPEC_CHANGELOG. Verify exact reference recognition, identity/snapshot behavior, same-Process placement/admission and error precedence, stale/mismatched resolution context, missing-source failure timing, and public protocol integration; if these require a substantive choice not covered by the approved candidate, obtain explicit owner approval. Do not silently restrict existing distributed String spawn or infer a public placement mechanism. No new syntax, FOREIGN references, revocation, persistent serialization, ordinary send/request transfer, GroupRef/P transfer or cross-Process reference transport is selected.

I180 acceptance:
- identical spelling across packages, exact dependency aliases/versions, direct-file relative source, std:/app: compatibility, explicit private MODULE and SOURCE delegation, no unrelated private acquisition, invalid/forged reference, destination grant checks including cache hits, root-only controls, no initialization during resolution, distinct Actor-local instances, cycles and failure eviction/retry;
- supported spawn reference transfer and rejection at unapproved boundaries, with creation cutover and placement invariants;
- real package and direct-file CLI HTTP 200/404/requests for both Host variants, factory payload/lifecycle, ready/drain/abort/quotas and network-grant controls, JVM and Linux x86_64 Native (not an extra ARM gate); distinguish actual responses from readiness-only controls;
- establish equivalent representation/transfer mechanism in existing Truffle languages, especially GraalPy, before any necessary Java change; reuse existing canonical resolution/loading and actor admission, keep ordinary policy in Protos;
- report human-executed static, focal and required integrated validation, exact product publication, specification/version/changelog and guide changes as applicable, and durable cross-reference closure evidence.

## Evidence boundaries

Initial live evidence: [D265 investigation comment](https://github.com/guillermomolina/protos/issues/1193#issuecomment-7199352216). Inspected governing files: root AGENTS.md; AGENTS.work/DESIGN.md, COORDINATION.md, REFERENCE.md; normative owners linked above; HTTP Host and worker sources, core import, ProtosModuleRuntime, ProtosStandardActorProtocol, ProtosStandardImportProtocol, package-resolution implementation, and relevant D053/D216/D225/D242/I154/PLAT075/I167 records. Some large source fetches were inspected through relevant sections rather than exhaustive unrelated line-by-line reads. No product code, specification, release or runtime test has been changed or run for this selection. GITHUB010 research is comparative, not measured performance or a proof of implementation feasibility.
