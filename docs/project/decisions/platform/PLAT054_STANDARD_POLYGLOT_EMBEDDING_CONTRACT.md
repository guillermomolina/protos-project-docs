# PLAT054 — Standard Polyglot embedding and language-bindings contract

Status: **RATIFIED PLATFORM ARCHITECTURE — NORMATIVE SPECIFICATION PUBLISHED (0.1.449), RUNTIME IMPLEMENTATION PENDING**

Selected: **Candidate A — one lazily initialized Protos Process per embedding Polyglot Context, standard Truffle scope and executable interop, ordinary Protos module semantics**.

Decision issue: https://github.com/guillermomolina/protos/issues/838
Performance parent: https://github.com/guillermomolina/protos/issues/831
Callable prerequisite: PERF033 / https://github.com/guillermomolina/protos/issues/832
Previous evidence: [PLAT054-0](../../evidence/PLAT054/PLAT054_0_STANDARD_POLYGLOT_BINDINGS_HARNESS_RECONCILIATION.md)
Owner approval and comparative evidence: [PLAT054-1](../../evidence/PLAT054/PLAT054_1_OWNER_APPROVAL_NORMATIVE_RECONCILIATION.md)
Normative publication evidence: [PLAT054-2](../../evidence/PLAT054/PLAT054_2_NORMATIVE_PUBLICATION.md)

Approval: explicit project-owner response on **2026-10-08**, *"ok aprobado"*, to the PLAT054-1 research packet and its exact seven-clause approval text. This approval ratifies the candidate and identified refinements; it does **not** assert that Protos normative files were already modified, that the Java implementation is available, or that tests passed.

## Governing architecture

The supported public embedding shape is:

~~~java
try (Context context = Context.newBuilder("protos").build()) {
    context.eval(source);
    Value run = context.getBindings("protos").getMember("truffleRun");
    Value result = run.execute();
}
~~~

It must not require ProtosStandaloneHostedSession, prepareTopLevel, a benchmark-specific bootstrap, or a Java-supplied mandatory Core path.

The chosen architecture has exactly one logical Protos Process and RootActor for a live embedding Context. Bootstrap is lazy at the first valid guest source execution. Creating/closing an unused Context and requesting empty pre-evaluation language bindings must not bootstrap Core, Process, RootActor, provider compartments, or scheduler machinery. Polyglot Context and Protos Process are correlated lifetimes, not identical semantic objects (PLAT001).

The selected Core override option is `protos.CoreRoot`. Precedence is (1) explicit override, (2) distribution/language home, (3) packaged internal language resource where language home is unavailable. An invalid explicit override fails rather than silently falling back. Core resource reads cannot grant arbitrary guest filesystem access. The exact option spelling and precedence were expressly part of the PLAT054-1 approval.

Process arguments, environment snapshots and standard streams derive from the actual TruffleLanguage.Env, not ambient System.getenv, System.in/out/err or an unrestricted JVM filesystem. Program filesystem access follows explicit host authority; a default Network capability is absent unless explicitly provisioned. Core source/readability authority and ordinary program filesystem/network authority remain distinct. Unused foreign providers remain lazy (PLAT053).

The Process persists between successful Context.eval calls. Each host evaluation uses normal Protos module instances, not an implicit REPL namespace, host global-variable table, export registry, or forced ModuleKey. An importable canonical entry uses its Actor-local ModuleKey cache and cache-before-execute rules; a READY entry is not reinitialized. A subsequent host evaluation of a READY canonical entry returns its recorded initialization terminal result without repeating its effects. A standalone entry with no importable canonical ModuleKey gets a distinct moduleContext on each evaluation, even for an identical Source or display name.

The first RootActor entry alone receives its bootstrap-local process slot and any authorized default filesystem/network slots. Subsequent host entries do not acquire these slots implicitly. Earlier effects and references survive ordinary module-initialization failures where the Process itself survives. A failure does not select a new bindings module. A normally completed entry becomes the last-selected module. No separate module-export registry is introduced.

One stable TruffleLanguage.getScope projection exposes the *own local slots* of the last normally completed entry module to Java. It must not leak the prelude, delegated slots, or imported-module slots as if they were local. Java member writes, removals and insertions are rejected. Ordinary guest mutation retains normal slot behavior. Retained host bindings reflect later successful selection, but a Value obtained earlier never resolves the name again when executed.

Host member reads of Closure-valued slots preserve CALLABLES.md receiver-bound extraction semantics, including fresh extracted Closure identity on successive ordinary member reads, receiver and methodHome provenance, lexical capture by reference, and original return home. The extracted Value retains its exact Closure; no dynamic redirection after slot mutation or another eval. The interop layer must support source-backed and native Closures and ordinary supplied arguments/default/rest semantics where applicable, without a second guest invocation engine. A host-facing interop arity failure must not mask effects or precedence of normative guest parameter/default binding.

A host-initiated synchronous ordinary execution in the RootActor is an Actor-local turn for fatal unhandled Error purposes, irrespective of whether a physical RootTask/ProtosTask is materialized. An Error handled by Protos does not itself kill the RootActor. An unhandled Error escaping the outermost boundary kills that RootActor; in the minimal Process its failure is fatal to the Process. The same Context must not automatically recreate a Process, retry, resume, or convert the fatal failure into a recoverable ordinary result. Compilation/parse failures before Process bootstrap do not themselves create or fatally terminate an otherwise nonexistent Process.

When the Process terminates, retained Values may not resume guest execution and language bindings expose no live Process-owned members. Context.close() ensures semantic Process termination, revokes Process-local authority, releases existing provider sessions/compartments and host resources, and completes disposeContext without executing arbitrary guest code after termination. Closing after a fatal failure is idempotent with respect to semantic termination.

No universal per-call serialization, fresh RootTask/ProtosTask, Actor task registration, scheduler dispatch, rich activation, Process-host routing, duplicated Context enter/leave, outcome envelope, global Closure registry, or foreign provider creation. Unsafe simultaneous entry into the same mutable RootActor domain is rejected; independent Actor/P progress under PLAT001 remains possible. Reuse the PERF033 ordinary callable compact direct-call/Truffle interop boundary and PLAT046 caller-thread placement. Keep the stronger reusable hosted-session lifecycle contract separate.

## GITHUB021 — approved-invariant reconciliation

The decision preserves all eight recorded owner invariants:

1. One lazy Process per embedding Context — **PRESERVED**.
2. Core distribution default plus explicit override — **PRESERVED**, exact option and fallback approved.
3. Env-controlled I/O, args, environment, restricted filesystem and denied-by-default Network — **PRESERVED**.
4. Persistent Process, ordinary modules and last-completed module bindings, no REPL — **PRESERVED**, entry identity/READY behavior clarified.
5. Java read-only bindings; ordinary guest-side mutation — **PRESERVED**.
6. No mandatory serialization; reject unsafe concurrent same-RootActor entry — **PRESERVED**.
7. Retained Value captures exact Closure, no later redirection — **PRESERVED**, extraction identity clarified.
8. PAY AS YOU GROW, no universal Task/RootTask/scheduler/rich activation — **PRESERVED**.

New, *explicitly owner-approved* consequences: fatal host-entry RootActor turn classification; READY canonical host-eval result reuse; read-only dynamic scope and terminated-scope invalidation; Closure extracted-method identity plus native/arity interop; CoreRoot option and precedence; no Process resurrection. These consequences were visible in the seven-point approval text, not inferred from general permission to continue. No ninth contradictory direction is introduced.

## Normative authority and ordering

This PLAT decision is non-normative. It cannot supersede the specification. The approved guest-visible additions were published in `guillermomolina/protos@ab1f2196e6736d4df01e83f685a3fc8aa3f606ac` (specification revision `0.1.449`) in:

- `spec/semantics/MODULES.md`: host entry-module/cache/identity/result and bindings-local-slot selection; preserve existing cache-before-execute, cycles, failure eviction, independent reachability and standalone distinctions.
- `spec/io/PROCESS_IO.md`: lazy embedding/bootstrap, exact initial-entry authority, Truffle Env snapshot and authority restrictions, fatal Process custody/disposition.
- `spec/concurrency/ACTORS.md` §24C: host-entry Actor-turn classification without requiring physical Tasks.
- `spec/PROTOS_SPEC_CHANGELOG.md`: global normative revision `0.1.449` identifying those owners and effects.

The approved detailed draft is retained in the linked PLAT054-1 evidence. Implementation MUST NOT treat historical PERF033 tests or wrapper behavior as semantic authority. In particular, the retained test for an escaped InvalidReturn followed by successful subsequent invocation must be reconsidered against the now approved fatal RootActor turn classification, preserving behavior only for a separately valid handled/non-turn case.

After the normative publication, implement product embedding in a coherent bounded group in `guillermomolina/protos`. Do not change `guillermomolina/protos-benchmarks` in that product phase. Benchmark preparation symmetry is a later independent validation; identical timed `Value.execute()` does not imply Graal graph-node count or latency parity.

~~~text
PLAT054_PLATFORM_DECISION=RATIFIED_2026_10_08
NORMATIVE_SPECIFICATION_PUBLICATION=PASS_0.1.449
PRODUCT_EMBEDDING_IMPLEMENTED=NO
PRODUCT_IMPLEMENTATION_RELEASE_GATE=HEAD_REVALIDATION_AND_CONFORMANCE_TESTS
BENCHMARK_PARITY_CLAIM=NO
~~~

## Alternatives, incremental design and adversarial scope

Candidate A (selected): lazy Process/Context and standard scope. B: eager Process/Context. C: Process per eval. D: mandatory specialized hosted session. E: REPL-like shared namespace or export registry. F: defer/do nothing. The twelve-dimension GITHUB010 comparison, score rationales/confidence, Truffle and external precedent survey, strongest counterargument, regret scenario and escape paths are in PLAT054-1 evidence.

C violates persistent Process/module identity; E invents a second global namespace; B/D impose unused startup or wrapper coordination cost; F does not deliver the requested standard embedding. A alone combines current semantic invariants with a familiar public Polyglot boundary and representation laziness.

Adversarial cases include multiple Contexts on one Engine; repeated importable and standalone sources; cycles and failed initialization with escaped partial objects; replacement module selection; host Values retained across eval and close; foreign/native Closure arguments and nonlocal returns; handled versus unhandled RootActor Errors; hostile Env/filesystem/network access; concurrent calls and Actor/P carriers; unopened/unused Contexts; native image and absence of language home; large Context counts. Current unneeded facilities remain deferred, not preimplemented.

Strongest argument against A: standard host-initiated entry exposes a subtle new RootActor error/lifecycle boundary and a dynamic module scope requiring precise specification and tests. This is preferable to hiding those semantics in specialized sessions. A regret scenario is excessive cold-start cost for thousands of short-lived Contexts; the escape is semantically invisible sharing of frozen Core/code artifacts and bounded infrastructure caches, never shared mutable module/Actor state or Process reuse behind one Context.

## PLAT054-2 normative release (2026-10-08)

The approved contract is now normatively published as `spec/0.1.449` at [`guillermomolina/protos@ab1f2196e6736d4df01e83f685a3fc8aa3f606ac`](https://github.com/guillermomolina/protos/commit/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac). Its four changed paths are `MODULES.md`, `PROCESS_IO.md`, `ACTORS.md` and the specification changelog. The owner reported all local tests passed and `git diff --check` clean; no raw test logs were independently inspected. The normative contract is now the primary authority for the next Java implementation. [Exact evidence and remaining obligations](../../evidence/PLAT054/PLAT054_2_NORMATIVE_PUBLICATION.md).

This release **does not** mean `Context.eval + getBindings` is yet supported by the product runtime, or that benchmark graph/latency parity has been measured. PLAT054-3 is implementation under [I086 / #840](https://github.com/guillermomolina/protos/issues/840) in `guillermomolina/protos`, not benchmark modification.

## Evidence, exact baseline and AI assistance

Reviewed product snapshot: `guillermomolina/protos@ac1e660cc37f8629852062fee41dcc33cf0758f8`, Protos 0.3.281-SNAPSHOT, GraalVM 25.4.4.1.1, global language specification revision 0.1.448. Evidence is source/reasoning only. No commands, builds, tests, A/B runs, graph measurements, or normative/product mutations were performed in preparing this approval record. Current HEAD must be checked again before each subsequent product edit.

This decision and linked evidence were drafted with AI assistance from ChatGPT and explicitly approved by the project owner in the active conversation. No independent human test/review or published specification change is implied.
