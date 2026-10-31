# D228 — REPL receiver-selector Tab completion authority and delegation

**Status: RATIFIED — explicitly owner-approved D228-A1, 2026-10-31.**

- Formal decision: [D228/#1001](https://github.com/guillermomolina/protos/issues/1001).
- Implementation consumer: [I134/#1002](https://github.com/guillermomolina/protos/issues/1002), **one atomic implementation slice**.
- Earlier authority: [D227-A1 revised](D227_REPL_TAB_COMPLETION_CANDIDATE_AUTHORITY_AND_LIVE_SESSION_VISIBILITY.md), [D227/#999](https://github.com/guillermomolina/protos/issues/999).
- [Owner approval and source audit](../../evidence/D228/D228_A1_OWNER_APPROVAL_AND_SOURCE_AUDIT_2026_10_31.md).
- Inspected product HEAD: [`guillermomolina/protos@d555458978c2979a496b36bfa2df19dc964da4c3`](https://github.com/guillermomolina/protos/commit/d555458978c2979a496b36bfa2df19dc964da4c3), **read-only baseline**, not implementation publication.
- Classification: **implementation-independent interactive tooling policy**, not Protos language or Standard Library semantics. Normative authority remains under `spec/`.

## Owner-selected decision and precise D227 delta

The shipped CLI011 completion covers live lexical names and meta-commands, but deliberately not `object.met<TAB>`. On 2026-10-31, after presentation of exhaustive A–F alternatives, source comparisons, failure tests, budgets, and GITHUB021 review, the owner expressly approved **D228-A1**: parser-proven `SurfaceMember` selectors, narrowly read-only admitted lexical receiver references and `this`, syntax-only grouping, local and inherited ordinary slot names without classifying methods versus data, no guest evaluation, no foreign inspection, no deferred-context materialization, coherent owner-Actor/Process admission, budgets/fail-closed behavior and JVM/Native parity. D227-A1 is preserved for bare lexical names; other receiver forms are deferred.

**Newly authorized and bounded:** exactly one transient read of an existing, PRESENT lexical receiver binding's value to identify the object whose *names* will be enumerated. This applies ONLY to the new `SurfaceMember` provider, not the earlier D227 bare-name provider. `this` is read as an existing intrinsic activation reference. Neither the value nor other guest values may be retained after TAB. No arbitrary expression evaluation is authorized.

## Ratified candidate-selection contract

### Parser-proven sites and receivers

A candidate site must be proved by the **real Protos lexer and parser** to be a `SurfaceMember` selector in an ordinary member read or send, with an exact UTF-16 replacement span. This is distinct from D227's `SurfaceName` proof. It may use a collision-free synthetic selector *for parsing only* when completing `obj.`; no syntax is ever evaluated to find candidates. Support provable `obj.met`, `obj.`, `obj.met()`, cursor-in-word and pending multiline input. No suggestions if source is invalid/ambiguous, in a string/comment, splits a surrogate pair, or cannot prove a safe site. Exclude member-creation/assignment targets.

Admit receiver forms ONLY:
1. `name` resolving to a PRESENT **own local binding** in the current REPL activation's genuine lexical chain, nearest scope first, including the frozen prelude only if it is on that chain;
2. the existing `this` intrinsic receiver when safely admitted; and
3. any number of syntax-only parentheses around exactly one otherwise admitted `name` or `this`.

Do not apply bare-name receiver fallback to `this` when resolving a lexical receiver binding. Ordinary delegation of lexical context objects never participates in that resolution. PRESENT semantic `null` is not ABSENT and may be a semantic receiver with inherited names.

Exclude first-release `context` (deferred context concern), `super` message sends, arbitrary literals, computed/called receivers, chained member-value reads (`obj.child.met`), `import()`, indexing, object/array/map construction, foreign or interop inspection and dynamic foreign-member fallback. A *previously imported and bound* source-backed Protos module instance may participate as an ordinary Protos object, with no import executed on TAB.

### Admission, transient value and names-only projection

Enter the correct LIVE Process Polyglot Context and prove coherent **owning RootActor** observation. The existing `ProtosPolyglotProcessContext.persistentRootActorGate` protects persistent evaluation and the D227 snapshot; reuse this narrow admission seam, but **do not assume the gate alone proves no other Actor turn can mutate the inspected graph**. I134 MUST demonstrate the full scheduler/Actor admission and non-overlap; a lifecycle read lock or `context.enter()` alone is insufficient. If the proof is unavailable, fail closed and stop for a new owner decision. Do not introduce a global lock, inspect another Actor, or acquire new permissions.

Resolve the nearest lexical PRESENT binding using its **single authoritative local binding store** without creating a second store, evaluating a selector, materializing deferred contexts (`context()` / `capturedLexicalContexts()`), or falling back to source history. One transient value reference is allowed ONLY during the admitted TAB. Never execute Closure, getter, callback, message, module import, interop, `slotValue()`, constructor, network or filesystem operation. No retained guest reference, cache, journal, background index or LSP daemon.

For the admitted Protos receiver, enumerate **local selector names** using a names-only internal primitive (e.g. `ProtosObjectValue.appendLocalBindingNamesTo`) and follow the **ordinary immutable delegation-parent chain** through to canonical `Object`. Handle `ProtosRepresentedValue` and heterogeneous valid semantic parents consistent with `ProtosValueLookup.delegationParent`; do not rely on `ProtosObjectValue.lookupSlot()` alone or on debugger scope values/representation. A represented receiver with no local ordinary table may still expose names from its parent. Unsupported/foreign representations fail closed; never query foreign fallback.

Include all PRESENT structural selector names, regardless of value kind, including inherited standard behaviors whose invocation may reject the original receiver's semantic family. A selector is not a guarantee of callable method or successful invocation. Closures are the sole executable value kind; "method" is an invocation role, not a distinct selector namespace. Nearest local occurrence shadows inherited ones, even when the local value is data. Deduplicate nearest-first, sort **within each level** in Unicode scalar order. Refresh after each TAB, including mutation, removal, shadowing and re-creation. Open, closed and frozen objects are equally visible for safe reading.

Valid `member-name` spellings follow Unicode 17 XID/NFC and include the six structurally legal reserved names after `.`: `this`, `context`, `super`, `true`, `false`, `null`. The D227 *bare identifier* validator is insufficient for these positions. Reject names not expressible as member syntax. Retain provenance (own versus delegated) in the provider/tests without reading slot values.

### Bounds, failure and editing

| On-demand budget | Value | Required action |
|---|---:|---|
| Source including pending input | 65,536 UTF-16 code units | No candidates if exceeded |
| Display candidate maximum | 256 | Deterministic cap; do not claim full-set completeness |
| Names scanned per TAB | 1,024 | **Fail closed** if exceeded; never emit unmarked partial structural set |
| Delegation nodes per TAB | 64 | **Fail closed** if exceeded, including cycles |
| Guest references retained after TAB | **0** | Mandatory |

Enforce name-scan limits **during enumeration**, not after allocating an unbounded list, including map-backed and frame-backed lexical authorities; add only the smallest backend-private bounded API if necessary. No normal execution or ordinary slot mutation pays for unused TAB.

If access, Process lifetime, Actor exclusivity, representation, parent topology, budget, parser proof or JLine replacement is unsafe, return **no member candidates**, leaving the typed buffer intact and signaling no new guest error. Filter by prefix **and** suffix around cursor; preserve exact parser-authorized span, `COMPLETE_IN_WORD`, `Candidate.complete=false`, no implicit whitespace, no speculative or fuzzy names. JLine is an editor adapter, not language or runtime authority.

Keep command-only context, current D227 lexical completion, history, bracketed paste, multiline, Ctrl-C/Ctrl-D, prompt/error handling, and non-TTY stream mode unchanged. JVM and Native CLI must expose equivalent user-visible behavior.

## Comparative alternatives, scoring and disqualifications (GITHUB010)

A: static-only syntax/member proof; B: direct receiver local names only; C: restricted receiver plus ordinary inherited names (**selected**); D: arbitrary receiver eval/import/getters; E: general debugger/interop/LSP member authority; F: conservative staged feature rollout. **C with the narrow scope discipline of F** is selected *in one I134 publication slice*. A does not know present mutable bindings; B misses prototype inheritance central to the request; F is a strategy, not sufficient behavior. D is disqualified for executing user code; E for incompatible inspection authority and unjustified infrastructure. Scores are comparative engineering judgments (1–5 favorable; H/M/L confidence), not measurements or permission to override hard red flags.

| Mandatory dimension | A static | B local | C inherited | D evaluate | E tooling-wide | F staging |
|---|---|---|---|---|---|---|
| Correctness/invariants | 5H no guest reads | 4M narrow reads | 4M bounded chain | 1H side effects | 2M alien authority | 4M limited |
| Protos alignment | 3H misses live | 4H ordinary slots | 5H delegation | 1H implicit execution | 2H new institution | 4H incremental |
| Pay for present need | 5H minimal | 4H TAB only | 4M inherited methods needed | 1H excess | 1H extra service | 5H bounded |
| Grow incrementally | 4M add runtime | 4H add parents | 5M add future receiver proof | 2M hard to restrict | 3M coupled | 5H controlled |
| Future-option resilience | 4M portable | 4M extends | 5M semantic model | 2M unsafe promises | 2M lock-in | 4M retains options |
| Scalability | 5M local | 4M one receiver | 4M budgeted | 1M unbounded | 2M indices/lifetime | 4M bounded |
| Conceptual simplicity | 5H proof only | 4H local table | 4M one delegation walk | 1H many effects | 2H parallel model | 4H narrow |
| Portability | 5H parser | 4M runtime | 4M dual-target proof | 2M host variance | 2M backend coupling | 4M no new platform |
| Resource costs | 5H parse | 4M TAB-only | 4M capped TAB | 1H unknown effects | 2M fixed cost | 5M deferred |
| Failure/operability | 5H empty safe | 4M fail closed | 4M fail closed | 1H uncontrolled | 2M lifespan issues | 4M explicit |
| Deferral/reversibility | 3M add observer later | 4H add chain | 5M no semantics migration | 1M difficult retreat | 2M integration rewrite | 4H bounded |
| Evidence/risk | 5H LM010 existing | 4M runtime APIs | 4M integration untested | 4H mature but unsafe | 3M known but mismatched | 4M feasible |

**Non-compensating gates:** no score can authorize guest execution, unproven Actor coherence, invalid lexical authority or false member-name claims. Overengineering in D/E is not outweighed by broad future coverage; avoiding inheritance in B is an underengineering risk for a prototype language.

## Source-based prior art and relevance

- [GraalPy `rlcompleter.py`](https://github.com/oracle/graalpython/blob/master/graalpython/lib-python/3/rlcompleter.py): `attr_matches` uses `eval`, `dir`, potentially `getattr`/hooks. **Never copy evaluation.**
- [TruffleRuby IRB `completion.rb`](https://github.com/truffleruby/truffleruby/blob/master/lib/gems/gems/irb-1.16.0/lib/irb/completion.rb): static/type analysis plus dynamic binding and `eval` fallback. **Separate mechanisms, do not import authority.**
- [GraalJS/Node REPL `completion.js`](https://github.com/oracle/graaljs/blob/master/graal-nodejs/lib/internal/repl/completion.js): syntax-aware candidates, own/prototype property enumeration, receiver evaluation, getter/proxy safety warning, import/path completions. **Prototype proximity only.**
- [OpenJDK JShell `ConsoleIOContext`](https://github.com/openjdk/jdk/blob/master/src/jdk.jshell/share/classes/jdk/internal/jshell/tool/ConsoleIOContext.java): separate source analysis/anchor and terminal editing, not a live Protos slot authority.
- [Elixir IEx `autocomplete.ex`](https://github.com/elixir-lang/elixir/blob/main/lib/iex/lib/iex/autocomplete.ex): session-aware variable/dot completion with different module and concurrency semantics.
- [Apple Pkl `ReplServer`](https://github.com/apple/pkl/blob/main/pkl-core/src/main/java/org/pkl/core/repl/ReplServer.java): completion evaluates into temporary REPL state; temporary state does not eliminate external effects.
- [JLine `CompletionMatcherImpl`](https://github.com/jline/jline3/blob/master/reader/src/main/java/org/jline/reader/impl/CompletionMatcherImpl.java): presentation only; Protos currently uses JLine 3.30.6.
- Truffle [Espresso](https://github.com/oracle/graal/blob/master/espresso/docs/how-espresso-works.md), [Sulong](https://github.com/oracle/graal/tree/master/sulong), [SimpleLanguage](https://github.com/oracle/graal/tree/master/truffle/external_repos/simplelanguage): examined for context/representation boundaries, with no equivalent persistent RootActor REPL selector authority established in the examined surfaces.

Precedents are comparative evidence, never normative Protos semantics.

## GITHUB021 — preserved and explicitly widened invariants

| Authority | Decision impact |
|---|---|
| D227 bare lexical-name enumeration is names-only, no receiver values | **PRESERVED for bare names**; **EXPLICITLY WIDENED** only for new proven `SurfaceMember` to one transient admitted receiver value, owner-approved |
| D227 no guest evaluation/getter/import/foreign lookup | PRESERVED |
| D227 no deferred Context materialization, second store, history index or LSP | PRESERVED |
| D227 entered Process **plus** actual Actor coherence | PRESERVED, implementation proof still pending |
| OBJECT_MODEL single parent/ordinary delegation, open/closed/frozen state | PRESERVED |
| CALLABLES one Closure value kind, dynamic receiver and receiver-domain restrictions | PRESERVED; candidates do not certify successful calls |
| EXECUTION_AND_CONTROL genuine lexical contexts distinct from delegation and receiver fallback | PRESERVED |
| MODULES ordinary module instance, explicit `import()` | PRESERVED |
| PROTOS_GRAMMAR Unicode 17/NFC, contextual reserved member spellings | PRESERVED |
| ACTORS / PROCESS_IO Actor isolation and Process lifecycle | PRESERVED; acceptance evidence pending |
| CLI011 released D227 scope | PRESERVED; D228 is a new extension |

**GITHUB021_INVARIANT_CONSISTENCY=PASS_FOR_APPROVED_POLICY**, because the only D227 contradiction is surfaced and explicitly reopened for the bounded *new provider*. If implementation needs a different permission/Actor model, owner approval must be requested again.

## Falsification, incremental design and future stress

I134 must test: local data shadowing an inherited Closure; mutation/removal/recreation; invalid receiver-domain inherited standard selectors; PRESENT `null` versus absent; frozen/represented parents; no guest effects from a would-be called receiver, getter or import; failure of Actor admission/Process termination; cycles and huge name/parent sets; Unicode reserved member spellings, cursor in middle, multiline, invalid source and non-TTY silence.

**Smallest sufficient** is parser-proven receiver reference + admitted single binding-value read + names-only parent-chain walk + existing JLine adapter. Inherited names are justified *now* by the motivating prototype method use case; generic receiver evaluation, literal/`context` receivers, foreign members, LSP and distributed indexes are not. **Pay only for what is used**: no per-mutation bookkeeping, background services or global locks. **Grow as needed**: more receiver proof forms can be added later without redesigning ordinary delegation enumeration. **Deferral cost**: nested receiver support would require a separately approved intermediate-value observation and parser-proof expansion, not a fundamental Protos semantics rewrite. **Future stress**: large object graphs, many Actors/Processes, concurrent schedulers, Native Image, alternate VMs, cancellation, distributed Process boundaries and persistence must remain Actor-local and fail closed.

**Strongest argument against C:** even reading a currently stored receiver value widens D227's authority, and the existing gate may not exclude every concurrent mutating Actor turn. The ratified answer is a **hard proof gate for I134**, not implicit permission to add a global lock or inspect another Actor.

**Intentionally deferred:** `context`, literals, computed/called/indexed/chained receivers, `super` sends, interop/foreign fallback, type or method-callability inference, remote inspection, imports, debugger/LSP authority, and completeness beyond the selected bounds.

## Implementation and closure boundary

I134, **not D228**, owns one coherent implementation in `guillermomolina/protos` current HEAD: parser/selector provider, runtime observer with proven Actor ownership, bounded delegation, safe JLine, focused regression tests, docs/help, and final version/changelog after validation. Human executor runs tests/builds/real JVM + Native PTY acceptance and product commit/push under Protos `AGENTS.md`. No implementation agent needs other repositories or the web: this packet includes the external precedents, and the previously supplied I134 handoff includes file targets/tests.

No changes to normative `spec/` or product source are authorized by **ratification alone**. Design is complete; actual resource, Actor-coherence, parsing and portability proof remains I134 acceptance. Unexpected substantive choices return to the owner.

```text
DECISION=D228-A1
OWNER_APPROVAL=EXPLICIT_2026-10-31
DECISION_ROLE=TOOLING_NON_NORMATIVE
GITHUB021_INVARIANT_CONSISTENCY=PASS_FOR_APPROVED_POLICY
D227_BARE_LEXICAL_AUTHORITY=UNCHANGED
NEW_RECEIVER_VALUE_READ=APPROVED_NARROWLY
PRODUCT_SPEC_CHANGE=NONE
PRODUCT_IMPLEMENTATION_PUBLICATION=NOT_CLAIMED
I134_SLICES=ONE
ACTOR_COHERENCE_AND_JVM_NATIVE_ACCEPTANCE=PENDING_I134
```
