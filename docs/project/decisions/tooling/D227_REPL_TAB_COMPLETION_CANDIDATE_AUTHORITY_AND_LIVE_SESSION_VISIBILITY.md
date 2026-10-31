# D227 — REPL Tab completion candidate authority and live session visibility

**Status: RATIFIED — owner-approved D227-A1 (revised), 2026-10-31.**

- Formal decision: [D227/#999](https://github.com/guillermomolina/protos/issues/999).
- Implementation consumer: [CLI011/#988](https://github.com/guillermomolina/protos/issues/988).
- Owner approval provenance: in the active 2026-10-31 conversation, immediately following the investigation and revised proposal, the project owner stated **"si apruebo D227-A1"** and authorized GitHub coordination plus publication in `guillermomolina/protos-project-docs`. This approves the **revised A1** packet, including the names-only and ownership conditions and the limited first-release incomplete-source behavior. It does not approve speculative receiver/import completion, a new language-server integration, public code publication, or a new language semantic rule.
- Investigation and invariant/delta review: [D227-A1 approval and source audit](../../evidence/D227/D227_A1_OWNER_APPROVAL_AND_SOURCE_AUDIT_2026_10_31.md).
- Reviewed product source: [`guillermomolina/protos@0ff5c9828e64b9a127d945f4e2cb98053dcb7c23`](https://github.com/guillermomolina/protos/commit/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23). This is an **inspection baseline**, not a D227/CLI011 implementation commit.
- Type of authority: **implementation-independent public interactive-tool completion policy**; it is non-normative for the Protos language. Existing lexical, module and grammar semantics retain authority under `spec/`.

## Problem and decision

An interactive REPL retains a live activation/context across successful and failed submitted units. Textual declarations in previously submitted source or command history are not reliable evidence that a local slot is currently present: ordinary mutation and removal can change it. Conversely, conservative document-static completion does not know every present REPL binding. TAB must suggest useful names without evaluating guest expressions, importing modules, silently inspecting dynamic receivers or inventing extra Process authority.

**Selected D227-A1 (revised):** combine a read-only, on-demand snapshot of **currently present lexical binding names** from the exact REPL activation with **independent, current-cursor, parser/lexer-authoritative static proofs**. JLine is solely the interactive editing/presentation adapter. Keep runtime names, proven static names and syntactic suggestions distinguishable in internal tests and diagnostics. No persistent journal, background index or language-server process is authoritative.

## Selected contract

1. **Interactive scope only.** TAB is active only on the real terminal REPL path, never the stream/piped branch or external shell-completion surface. REPL meta-commands (including `:help`, `:quit`, `:exit`) are candidates **only in meta-command context**; they are not Protos identifiers or keyword suggestions.
2. **Live name authority.** On each TAB request, enumerate *present local names* in the exact current REPL execution context and its genuine captured lexical-parent chain, including the frozen standard prelude **only if it belongs to that chain**. Traverse lexical scopes nearest-first, deduplicate by spelling, and apply lexical shadowing. Do not follow ordinary delegation of any context object. Presence of a local binding with value `null` is still presence; removal is absence, including after failed or partially executing submitted units.
3. **Names only, no materialization.** The runtime-private observation boundary enumerates names directly from the **single existing lexical-binding authority**, without reading/retaining guest slot values, constructing a second binding store, materializing deferred execution contexts, or creating any guest-visible state. Reusing `ProtosObjectValue.localSlotsSnapshot()` as-is is insufficient: its `Map<String,Object>` snapshot can read/retain frame-backed values. Reusing `ProtosActivation.capturedLexicalContexts()` as-is is insufficient: it may materialize deferred contexts. The exact internal API and representation remain implementation choices consistent with these constraints.
4. **Actor ownership and coherence.** Acquire the snapshot only under both (a) the correct entered, live Process Polyglot Context and (b) the actual **owner-Actor-local state observation/admission discipline**. Entering a Polyglot Context with its lifecycle *read* lock is not proof of exclusive ownership: its contract allows overlapping execution. Do not introduce a global lock or inspect another Actor's state merely for completion. If a coherent read is unavailable, closed, unsafe, or fails, **fail closed** without guest evaluation and preserve the edit buffer.
5. **Static proof authority.** Cursor-specific candidates from the current pending multiline source plus current editable text require the real Protos lexer/parser and existing D110/LM010 proofs. These static candidates do not create/verify session-local slots by themselves. Unevaluated declarations, source history, module cache guesses, strings and comments are not live-name authority. Only grammar-valid intrinsic/reserved syntax is offered at a proven site; bare `super` is not a standalone expression.
6. **Conservative incomplete input.** The first release **may return no candidates** when incomplete/lexically invalid/ambiguous input does not yield a sound cursor-site proof under the existing analyzer; it is not required to implement tolerant syntax recovery or complete every nested/incomplete expression. Later parser-authoritative recovery may expand coverage under a separately scoped change, but neither JLine's shell-style tokenization nor unchecked regex may become language syntax authority.
7. **Boundary exclusions.** No guest expression execution, getter/call/receiver evaluation, module import for side effects, imported-module member enumeration, delegated member-name guessing, type inference, filesystem/network scanning, capability acquisition, LSP daemon, or cross-Actor/global index. A module value bound in a current lexical slot may contribute **its binding name**, but not inferred members or implicit imported names.
8. **Presentation.** Deterministic candidate policy: meta-commands only in command context; in Protos source, current/nearest live lexical names first, remaining visible prelude names next, independently proven static names where not duplicated, then grammar-valid syntax candidates. Preserve distinct provenance in the provider/test layer even after deduplicating display spellings. Honor Unicode 17 XID and NFC, exact source offsets, cursor-in-middle/prefix/suffix replacement and multiline positioning. No implicit appended argument space for commands; no match leaves input unchanged, ambiguity may list candidates without erasing the buffer.
9. **On-demand and bounded.** Refresh at each TAB instead of maintaining a mutable history/mutation journal. Minimize allocations, bound work/retention, and do not charge normal evaluation paths for unused completion. A future cap/truncation must remain deterministic and must not falsely assert candidate-set completeness. Do not retain Process contexts or guest values beyond a TAB request.
10. **Existing REPL behavior preserved.** History, paste, multiline continuation, `Ctrl-C`, `Ctrl-D`, error recovery, prompt text, and non-interactive streaming semantics remain unchanged. JVM and Native CLI use equivalent user-visible behavior where both are supported.

This is **tooling behavior**, not a change to the normative lexical-lookup algorithm in `spec/semantics/EXECUTION_AND_CONTROL.md`, module semantics in `MODULES.md` or token grammar in `PROTOS_GRAMMAR.md`.

## Alternatives and comparative rationale

| Option | Assessment | Disposition |
|---|---|---|
| A — live lexical names + parser-backed static proof | Exact mutable-session presence, opt-in cost only on TAB; needs careful Actor admission and names-only API | **SELECTED with conditions above** |
| B — history/source journal with invalidation | Removal, recreation, escaped captured contexts, failed units and host bootstrap make history stale | Rejected as authoritative; historical text is not proof |
| C — static analyzer alone | Sound, protocol-neutral and cheap, but does not represent dynamic persistent REPL top-level names | Allowed as limited fallback, not complete CLI011 |
| D — evaluate receiver/import/member expressions | Side effects, unknown user code and amplification of authority | Rejected as a hard constraint |
| E — embedded LSP/workspace index | Extra lifecycle/index infrastructure without Process-local mutable-slot truth | Rejected as disproportionate |
| F — meta-command-only completion/defer | Reversible, but cannot meet CLI011's full persistent-name outcome | Safe incomplete alternative |

Source-level precedents examined in D227 include GraalPy `rlcompleter.py` (dynamic evaluation: **not** imported), TruffleRuby IRB/Reline (separation of editor/providers, not evaluation), GraalJS/Node REPL (broader runtime/package/filesystem completion: **not** granted), OpenJDK JShell (editing vs compiler-aware completion), Elixir IEx (session-aware bindings) and JLine 3 (editor transport). Truffle Espresso/Sulong/SimpleLanguage and Apple Pkl were relevance-screened; their examined surfaces do not establish a comparable Process-local REPL lexical-binding authority. These are comparative precedents, never normative Protos authority.

### GITHUB010 comparative matrix

Scores are 1–5, higher is more favorable; H/M/L indicate confidence. D/E are disqualified on non-compensating constraints and F does not meet the requested outcome. Explanations are concise engineering judgments, not measurements.

| Criterion | A live+static | B journal | C static-only |
|---|---|---|---|
| Correctness/invariants | 4/H: actual presence; admission required | 2/H: stale mutations | 4/H: sound, partial |
| Protos alignment | 5/H: ordinary lexical contexts | 2/M: parallel authority | 4/H: existing analysis |
| Pay for present need | 4/H: on TAB only | 2/M: ongoing tracking | 5/H: existing static path |
| Grow incrementally | 5/M: add proof categories later | 2/M: invalidation redesign | 4/M: live path can be added |
| Future-option resilience | 4/M: no LSP/process index | 2/M: journal obligations | 4/M: protocol-neutral |
| Scalability | 4/M: local bounded snapshot | 2/L: history grows/drifts | 5/M: document-local |
| Conceptual simplicity | 4/M: two justified sources | 2/M: tracking corner cases | 5/H: single source |
| Portability | 4/M: Process-local host API | 4/M: text-based | 5/H: parser-only |
| Runtime/resource cost | 4/M: demand allocation | 2/M: every mutation tracked | 5/H: demand parse |
| Failure/operability | 4/M: fail-closed | 2/H: stale positives | 4/H: conservative absence |
| Deferral/reversibility | 4/M: editor/provider separated | 2/M: retire journal | 3/M: later runtime integration |
| Evidence maturity/risk | 4/M: runtime read APIs; TTY pending | 2/M: no invalidation proof | 5/H: LM010 already tested |

Scoring does not compensate for unauthorized guest execution or claims of nonexistent bindings.

## Invariant/delta reconciliation (GITHUB021)

| Existing authority / approved boundary | Effect of selected A1 | Check |
|---|---|---|
| EXECUTION_AND_CONTROL §§4,6,7: only actual lexical local slots before receiver fallback | Enumerate exact present local names; **do not** turn context delegation or receiver fallback into lexical authority | **PRESERVED** |
| MODULES: ordinary per-module contexts, explicit `import(specifier)`, frozen prelude | No synthesized exports, implicit namespace, module loading or prelude mutation | **PRESERVED** |
| OBJECT_MODEL / CALLABLES: removal, creation, captured-context by-reference semantics | Refresh snapshots after mutations/failures instead of using history | **PRESERVED** |
| PROTOS_GRAMMAR §2: Unicode 17 NFC/XID and exactly six reserved words; member names differ from bare reads | Parser/lexer proof and no inferred dotted members | **PRESERVED** |
| Existing Process/Actor isolation and Polyglot lifecycle | Actor-local admission required in addition to entered Context; no global synchronization/capability gain | **PRESERVED, implementation proof pending** |
| D110/LM010: exact-source, conservative static proof | No promotion of static guesses into live authority; incomplete source can fail closed | **PRESERVED** |
| CLI011 requested interactive-only, no evaluation/import effects | JLine remains editor; stream mode intact; only approved candidate classes | **PRESERVED, acceptance pending** |

**DECISION_INVARIANT_CONSISTENCY=PASS for the exact approved policy.** No preexisting approved invariant is reopened. The implementation must demonstrate that the chosen actor ownership seam actually satisfies this contract; a discovered semantic conflict requires a new explicit checkpoint.

## Implementation / validation boundary and future evolution

CLI011, **not D227**, owns implementing the provider, names-only enumeration and snapshot admission, JLine wiring, tests, REPL help and `docs/guide/tools/cli.md`, plus product version/changelog/publication. Local preparatory CLI011 batches remain **unpublished** at the reviewed baseline; this ratification does not assert that they passed independent validation.

Acceptance must cover creation/removal/recreation, shadowing and frozen prelude; failed and partially executed units; strings/comments/Unicode/NFC/cursor mid-token; nested/multiline input; command whitespace; no-retained-guest-values; closed Context; Actor-safe access; and noninteractive silence. A real TTY on both portable JVM and Native CLI is needed separately from JUnit. Human-executor rules in `guillermomolina/protos/AGENTS.md` govern validation and product publication.

**Pay only for what is used:** the REPL pays only at interactive TAB; ordinary execution and noninteractive mode incur no continuous tracking. **Grow as needed:** proven parser recovery, more provider kinds and alternative terminal adapters can be added without changing the lexical authority model. **Cost of deferral:** omitting runtime snapshots forces adding a provider later but not changing Protos semantics; recording mutable history now creates avoidable consistency maintenance. **Strongest argument against A:** coherent Actor-local names-only enumeration over frame/deferred authority is more work than command-only completion; that cost is warranted by CLI011's persistent-name requirement and must not be hidden by premature public wiring.

**Intentionally deferred:** tolerant parsing of arbitrary unfinished source, completion of dynamic receiver members/imports/modules/paths, global/workspace semantic indexes, LSP lifetime, cross-Process discovery, heuristically inferred names, and shell completion for the external `protos` command. Any extension of completion authority or externally observable guarantee outside this packet requires its own review.

## Ratification and closure evidence

```text
DECISION=D227-A1_REVISED
OWNER_APPROVAL=PASS_2026-10-31
GITHUB021_INVARIANT_CONSISTENCY=PASS_POLICY
DURABLE_RECORD_DECISION=REQUIRED
DECISION_ROLE=TOOLING_NON_NORMATIVE
PRODUCT_SPEC_CHANGE=NONE
PRODUCT_IMPLEMENTATION_PUBLICATION=NOT_CLAIMED
CLI011_IMPLEMENTATION_OWNER=ISSUE_988
VALIDATION_OR_REAL_TTY_ACCEPTANCE=NOT_RUN_FOR_THIS_DECISION
```
