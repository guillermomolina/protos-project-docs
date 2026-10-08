# PLAT054-1 — Owner approval, normative reconciliation and comparative evidence

Date: 2026-10-08
Status: **OWNER-APPROVED RESEARCH AND PLATFORM RATIFICATION EVIDENCE; PRODUCT NORMATIVE REVISION PENDING**
Decision: https://github.com/guillermomolina/protos/issues/838
Parent: https://github.com/guillermomolina/protos/issues/831
Durable decision: [PLAT054](../../decisions/platform/PLAT054_STANDARD_POLYGLOT_EMBEDDING_CONTRACT.md)
Earlier harness evidence: [PLAT054-0](PLAT054_0_STANDARD_POLYGLOT_BINDINGS_HARNESS_RECONCILIATION.md)

## Exact approval

After receiving the PLAT054-1 research result and its seven-clause proposed approval text, the project owner responded in the active conversation:

> ok aprobado

The proposed text expressly covered:

1. Treat ordinary synchronous host-initiated RootActor execution as an Actor turn for fatal unhandled Errors without mandatory physical RootTask/ProtosTask.
2. One Process per Context, with no automatic restart following fatal failure.
3. Ordinary entry-module semantics: distinct standalone moduleContexts without ModuleKey, importable READY instances reused without initialization and returning retained terminal result.
4. Stable read-only Java scope showing local slots of last successfully completed entry, invalidated for live Process-member access upon termination.
5. Ordinary Closure capture/extracted-identity/receiver/arity/native-body/non-local-return semantics.
6. Public \`protos.CoreRoot\` override, distribution home then packaged-resource fallback, without authority amplification.
7. Existing module/error/Process I/O/Actor isolation/concurrency/PAY AS YOU GROW contracts; no REPL/global export registry, second call engine or universal serialization.

This was an exact, surfaced seven-point selection. The earlier eight owner-approved directions remain authoritative. Approval of a platform direction and of exact proposed normative text is **not** publication of the normative text in \`guillermomolina/protos\`.

~~~text
OWNER_APPROVAL=EXPLICIT_2026_10_08
SELECTED_CANDIDATE=A_LAZY_PROCESS_PER_CONTEXT_STANDARD_SCOPE_AND_INTEROP
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_DIRECTION_PRESERVED=YES_8_OF_8
INVARIANT_DELTA_REVIEW=PASS
NORMATIVE_PUBLICATION=NOT_YET
PRODUCT_IMPLEMENTATION=NOT_YET
~~~

## Exact revisions and documents inspected

Product revision at approval reconciliation:
- \`guillermomolina/protos@ac1e660cc37f8629852062fee41dcc33cf0758f8\`.
- Maven version: \`0.3.281-SNAPSHOT\`; GraalVM version: \`25.4.4.1.1\`.
- Normative global changelog latest: \`0.1.448\`.

Documentation reference at start of this publication: \`guillermomolina/protos-project-docs@363b6ccd84102861be533737abbbf41f92ac49a4\`.

Inspection authority:
- \`AGENTS.md\`, \`AGENTS.work/DESIGN.md\`, \`AGENTS.work/REFERENCE.md\`, \`AGENTS.work/COORDINATION.md\`, \`AGENTS.work/IMPLEMENTATION.md\`, \`spec/AGENTS.md\`.
- \`spec/semantics/MODULES.md\`, \`CALLABLES.md\`, \`EXECUTION_AND_CONTROL.md\`, \`ERRORS.md\`, \`spec/io/PROCESS_IO.md\`, \`IO_CORE.md\`, \`NETWORK.md\`, \`spec/concurrency/ACTORS.md\`, \`spec/PROTOS_SPEC_CHANGELOG.md\`.
- \`src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java\`, \`ProtosLanguageContext.java\`, \`ProtosHostExecutableClosure.java\`, \`ProtosStandaloneHostedSession.java\`, \`ProtosStandaloneHostedExecution.java\`, \`ProtosStandaloneProcessBootstrap.java\`, \`ProtosPolyglotProcessContext.java\`, \`ProtosCoreBootstrap.java\`; \`src/test/java/com/guillermomolina/protos/execution/ProtosPerf033CanonicalCallableInteropTest.java\`; \`pom.xml\`.
- Existing ratified decisions PLAT001, PLAT046, PLAT053; earlier PLAT054-0 evidence; issues #838, #831, #832, #750.

The prior PLAT054-0 evidence inspected \`guillermomolina/protos-benchmarks@e221bb056a208693c9102891df2e13d65e61eefb\`; this research did not modify it. Current timed canonical operations are \`Value.execute()\` in all three languages, while the preparation is \`Context.eval + getBindings\` for JS/Python and \`ProtosStandaloneHostedSession.prepareTopLevel(...).executable()\` for Protos. This is **preparation asymmetry**, not proven graph parity or a demonstrated timing cause.

## Source-confirmed normative fault line

\`ACTORS.md\` §24C says unhandled Errors escaping an Actor turn are fatal, including initialization turns; §32 says an unhandled RootActor fatal failure terminates the minimal Process. \`CALLABLES.md\` §14 makes a late non-local return from an escaped Closure signal InvalidReturn. Yet \`ProtosPerf033CanonicalCallableInteropTest.returnHomeAndControlSemanticsArePreserved()\` currently asserts that after an escaped Value.execute raises a guest exception, another operation in the same session can execute successfully. This test is not normative authority. The approved direct-host RootActor-turn definition resolves the semantic ambiguity without requiring Task allocation; regression expectations must be reconciled only after the new specification revision is published.

\`MODULES.md\` provides source-backed moduleContext identity, Actor-local ModuleKey cache, cache-before-execute, INITIALIZING/READY, cyclic partial access, failed-module eviction, and permanently standalone nonimportable instances. \`PROCESS_IO.md\` places process/filesystem/network bootstrap slots only on the initial RootActor module. A succession of direct Context.eval calls requires explicit application of those existing rules to later host entries, not REPL/global namespace semantics. A READY canonical module has to be returned without reinitialization; the approved host-eval result uses its retained terminal initialization result. Parse-cache identity never substitutes for ModuleKey identity.

\`CALLABLES.md\` §11 specifies a fresh receiver-bound Closure extraction per ordinary Closure-valued member read, preserving receiver and methodHome; §13/14 prescribe return-home and InvalidReturn; parameter/default/rest binding is a single left-to-right algorithm. \`ProtosHostExecutableClosure\` currently admits only zero supplied arguments and source-backed bodies; that is current implementation limitation, not the newly approved public contract. Its existing owner/context/target/DirectCallNode compact path must be reused rather than replaced.

\`ProtosLanguage\` currently creates a language context and parses to a Bytecode CallTarget, but has no ordinary \`getScope\` and \`disposeContext\` override. \`ProtosLanguageContext\` contains Context-owned caches and Env, but the Process for existing hosted sessions is attached by an external wrapper. PLAT054 closes this host integration without redefining PLAT001 physical Context-to-Process topology.

## Truffle/peer evidence, scope and caveats

Applicable public API mechanisms (reviewed against project-declared GraalVM 25.4.4.1.1 when source was available) include TruffleLanguage.parse, TruffleLanguage.getScope, TruffleLanguage.Env and its input/output/application-arguments/environment/internal-resource APIs, Context.eval, Context.getBindings, InteropLibrary and Value.execute, and TruffleLanguage.disposeContext. A language scope is allowed to be retained and reflect changing state; getBindings may initialize the language context before eval, so this may not force Protos Process bootstrap. disposeContext is not a guest-execution callback.

Reference systems: GraalJS (standard interop bindings), GraalPy (standard embedding without copying GIL), TruffleRuby (illustrative persistent interactive binding, deliberately *not* Protos semantics), Espresso (context-owned scope), LLVM/Sulong (global scope and separate lifetime), GraalWasm (language scope), SimpleLanguage (minimal Truffle reference), Apple Pkl (host evaluator and resource boundaries). Non-Truffle V8 and CPython establish the general distinction between host embedding state and language semantic modules. These are comparison precedents, not Protos authorities.

Primary documentation/reference links:
- https://www.graalvm.org/latest/graalvm-as-a-platform/language-implementation-framework/
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.html
- https://www.graalvm.org/sdk/javadoc/org/graalvm/polyglot/Context.html
- https://www.graalvm.org/sdk/javadoc/org/graalvm/polyglot/Value.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/InteropLibrary.html
- https://github.com/graalvm/simplelanguage
- https://github.com/oracle/graaljs
- https://github.com/oracle/graalpython
- https://github.com/oracle/truffleruby
- https://github.com/oracle/graal
- https://github.com/apple/pkl

This evidence does not falsely claim a complete line-by-line peer-runtime source parity audit, performance tests or a measured reduction of Protos Graal graph nodes.

## GITHUB010: full 12-dimension candidate comparison

Candidates:
- A = selected lazy Process per Context + standard language scope and executable interop.
- B = eager bootstrap per Context.
- C = new Process on every eval.
- D = mandatory specialized hosted session.
- E = shared REPL-like namespace or independent export registry.
- F = defer/do nothing.

Scores are 1 (weak) to 5 (strong), with concise *candidate-specific reasons* in every cell.

| Dimension | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| 1 Invariants/correctness | 5 existing identity | 4 same Process | 1 destroys persistent Process | 4 current special contract | 1 second namespace | 4 no change |
| 2 Protos philosophy | 5 ordinary modules | 3 idle work | 2 Process fragmentation | 2 special wrapper | 1 extra globals | 3 avoids work |
| 3 Present-need proportionality | 5 lazy standard surface | 2 bootstrap on idle | 2 repeated startup | 2 wrapper requirement | 1 registry cost | 2 unmet use case |
| 4 Incremental growth | 5 capable boundary | 3 eager to lazy migration | 2 fundamental ownership change | 3 can add bridge | 2 migrate globals | 3 can implement later |
| 5 Future-option resilience | 5 keeps Actor/Process model | 4 keeps same model | 2 hostile to persistence | 3 API-specific | 2 semantic namespace lock-in | 3 no near-term commitments |
| 6 Scalability | 5 independent Contexts | 3 many eager setups | 2 many Process restarts | 3 extra gates | 2 shared namespace bookkeeping | 2 API usability limit |
| 7 Conceptual simplicity | 4 one new host boundary | 3 explicit eager bootstrap | 2 lifecycle multiplication | 3 specialized lifecycle | 1 two module universes | 5 no change |
| 8 Portability/freedom | 4 public Truffle API | 4 public API | 3 host-specific rehosting | 3 Java wrapper | 2 extra host namespace | 5 no new dependency |
| 9 Runtime/resources | 5 pays on use | 2 eager allocation | 1 repeated Core/Process | 3 wrapper allocation | 2 extra registry | 5 no new running code |
| 10 Failure/operability | 4 precise fatal gate | 4 simple lifetime | 1 opaque retries | 4 established close | 2 two mutable states | 2 missing supported API |
| 11 Deferral/migration | 4 reversible physical choices | 3 remove eager later | 1 module identity migration | 3 replace consumers | 1 namespace ABI break | 2 delayed compatibility cost |
| 12 Evidence/implementation risk | 4 public APIs, gap in Protos | 4 known eager shape | 2 normative conflict | 5 existing adapter | 2 invent model | 5 no code risk |

Confidence: A **MEDIUM** until actual Java product regressions and version-specific behavior are validated; B, D and F **HIGH** for their current known operation; C and E **HIGH** for their conflicts. Candidate totals have no normative status and do not override hard disqualifications: C violates approved persistent Process identity; E creates a second namespace. B and D impose unjustified cost/boundary on the required minimum path, and F fails the requested functionality.

Present need: implement only one lazy Process, one existing module cache and one public scope projection, without an extra registry or mandatory scheduler. Incremental growth: retain separate Actor/Process/provider authorities and share immutable code artifacts later if many Contexts demand it. Deferring REPL, auto-restart, global foreign-provider pools, generic multi-caller serialization and speculative task wrappers does not require changing the approved semantic identity/lifecycle model later; adopting C or E *now* would.

Regret case: 10,000 very short-lived Contexts make cold-start/Core cost material. Escape: share immutable Core/code artifacts, cache bounded implementation-only metadata, optimize Context startup, while retaining separate mutable Process/Actor state. Strongest counterargument: the stable scope plus host RootActor Error boundary adds difficult failure handling despite the extremely small \`Value.execute()\` workload. Answer: that contract is required for correct general embedding, but the cost must be outside steady-state minimal execute when unused.

Adversarial checks required from implementation (not reported as already passed): multiple independent Contexts; same Source and different same-named Sources; importable READY and cyclic INITIALIZING; standalone entries and no fake ModuleKey; partially escaped module after failed initialization; handled and fatal Errors; held Value after reassignment, later eval, Context close; Closure receiver/home/default/rest/native body; restricted file environment/stdio; denied network; foreign provider laziness; concurrent same-Actor rejection without global GIL; normal Actor/P concurrency; Native/packaged Core; no-root-Task literal callable; stable namespace projection.

## Exact approved normative publication plan

The accepted text was presented in PLAT054-1 as complete English sections:
- \`MODULES.md\`: **Host-initiated evaluations in an embedded RootActor**. Direct subsequent entry handling, READY canonical reuse and retained terminal result, distinct standalone entries, last completed module scope, local slots, fresh bound Closure extraction, terminated scope.
- \`PROCESS_IO.md\`: **Standard Polyglot embedding bootstrap and authority**. Lazy first valid guest execution; Process/RootActor bootstrap slots only for first entry, Env snapshot/resources, default Network absent, no re-create after fatal error, Context disposal respects semantic termination.
- \`ACTORS.md\` §24C: **Host synchronous execution constitutes an Actor-local turn for unhandled Error fatality independent of physical Task materialization**.
- One new global \`spec/PROTOS_SPEC_CHANGELOG.md\` entry; no duplicated normative owner or unjustified \`ERRORS.md\`/\`CALLABLES.md\` rewrite.

The next slice is **IMPLEMENTATION in guillermomolina/protos**, normative publication first. All sources and validations are human-executed according to AGENTS.md; the prompt must be self-sufficient for a local HEAD checkout and not depend on reading another repository.

~~~text
PLAT054_1_RESULT=OWNER_APPROVED_ARCHITECTURE_RECORDED
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS_8_OF_8
NEW_OWNER_APPROVAL_REQUIRED=NO_FOR_EXACT_APPROVED_CONTRACT
NORMATIVE_PUBLICATION_REQUIRED=YES
NORMATIVE_PUBLICATION_ALREADY_DONE=NO
DESIGN_RATIFICATION=PLATFORM_RECORD_RATIFIED
IMPLEMENTATION_AUTHORIZED=AFTER_NORMATIVE_SPEC_PUBLICATION
NEXT_SLICE_TYPE=IMPLEMENTATION_NORMATIVE_SPECIFICATION
NEXT_REPOSITORY=guillermomolina/protos
NEW_ISSUE_REQUIRED=NO
~~~

## Validation and AI assistance

No code, normative files, benchmark, builds, tests, terminal commands or git validation were run in this publication. It is documentation-only, based on source inspection and owner-approved design. Drafting and comparative analysis were AI-assisted by ChatGPT; no independent human code review is claimed.
