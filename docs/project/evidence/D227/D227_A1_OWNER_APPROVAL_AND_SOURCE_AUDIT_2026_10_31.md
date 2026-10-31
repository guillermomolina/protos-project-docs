# D227-A1 — Owner approval and source-level audit (2026-10-31)

**Classification:** bounded design approval, source inspection and counterexample evidence; **not** product implementation/validation evidence.

- Formal design owner: [D227/#999](https://github.com/guillermomolina/protos/issues/999).
- Consumer: [CLI011/#988](https://github.com/guillermomolina/protos/issues/988).
- Ratified decision: [D227-A1 revised](../../decisions/tooling/D227_REPL_TAB_COMPLETION_CANDIDATE_AUTHORITY_AND_LIVE_SESSION_VISIBILITY.md).
- Investigation log: [D227 investigation comment #6538204270](https://github.com/guillermomolina/protos/issues/999#issuecomment-6538204270).
- Exact published product baseline inspected: [`guillermomolina/protos@0ff5c9828e64b9a127d945f4e2cb98053dcb7c23`](https://github.com/guillermomolina/protos/commit/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23), not a new D227 or CLI011 publication.
- Approval: explicit project-owner **"si apruebo D227-A1"**, 2026-10-31, following a report which recommended the **revised** candidate with the names-only, deferred-context, Actor-observation and conservative-incomplete-source constraints. The owner additionally authorized GitHub Issue coordination and bounded direct publication in `guillermomolina/protos-project-docs`.

## Repository evidence and precise constraints

The source-level investigation read governing `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/IMPLEMENTATION.md`, `spec/AGENTS.md`, `src/AGENTS.md`, `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/PROTOS_GRAMMAR.md`, `spec/semantics/EXECUTION_AND_CONTROL.md`, `MODULES.md`, `OBJECT_MODEL.md`, `CALLABLES.md` and `spec/io/PROCESS_IO.md`.

Targeted implementation/test surfaces (links fixed at product revision):

| Source | Observed evidence / constraint |
|---|---|
| [ProtosCli.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java#L1295-L1415) | JLine is constructed for interactive input only; separate `streamRepl`; pending multiline buffer; persistent evaluation; no published public JLine completer at this baseline |
| [ProtosObjectValue.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java#L300-L370) | `hasLocalSlot` and `localSlotsSnapshot` exist; latter returns `Map<String,Object>`, **not** an efficient names-only snapshot |
| [ProtosFrameLexicalBindingAuthority.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosFrameLexicalBindingAuthority.java#L480-L562) | Snapshot traverses present frame bindings and **reads their values**; direct reuse would violate the names-only requirement |
| [ProtosActivation.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java#L575-L748) | Deferred authority and captured environment exist; compatibility `capturedLexicalContexts()` materializes every context; avoid it for TAB |
| [ProtosLexicalEnvironment.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/lexical/ProtosLexicalEnvironment.java) | Captured chain is by reference and may be deferred; `contextsOf()` calls `context()` which materializes |
| [ProtosLexicalBindingAuthority.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/lexical/ProtosLexicalBindingAuthority.java) and [Calls.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/lexical/ProtosLexicalBindingAuthorityCalls.java) | Single binding authority, snapshot/append APIs; need bounded names-only, no guest-value reads or second store |
| [ProtosDebuggerScope.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDebuggerScope.java#L122-L160) | Debugger scope offers a read-only presentation but appends receiver/delegation names; not exact REPL lexical authority |
| [ProtosPolyglotExecutionContext.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotExecutionContext.java#L190-L245) and [ProtosPolyglotProcessContext.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotProcessContext.java) | Context `callEntered` takes a lifecycle **read** lock, and overlapping execution remains possible; separately prove owning Actor's coherent observation |
| [ProtosStaticCompletion.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/analysis/ProtosStaticCompletion.java) and [ProtosStaticDefinitions.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitions.java) | LM010 proof is tied to a real parse/lexer position, not to all live REPL slots |
| [ProtosStaticCompletionTest.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/test/java/com/guillermomolina/protos/analysis/ProtosStaticCompletionTest.java) and [ProtosReplTest.java](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/test/java/com/guillermomolina/protos/cli/ProtosReplTest.java) | Existing static-completion tests intentionally decline many unfinished fragments, strings/comments and unsafe cursor sites; REPL tests cover persistence/continuation, not actual TAB real-TTY acceptance |

The two untracked local CLI011 files `ProtosReplCompletion.java` and `ProtosReplJLineCompleter.java` were **not** in published `main` and were not audited as published evidence. Maintainer-reported CLI011 targeted JUnit batches are not product publication and do not establish real terminal acceptance.

## Cross-system survey and rejected mechanisms

D227/#999 records GraalPy, TruffleRuby/IRB, GraalJS/Node, OpenJDK JShell, Elixir IEx and JLine 3 (at least three technical families: dynamic guest/introspection, syntax/compiler-aware, and session-state/editor separation). Relevant direct-source confirmations: GraalPy `rlcompleter.py` evaluates receiver text for attribute matching; Node REPL completion covers imports, FS operations and context-dependent sources; IEx completion queries shell state. These mechanisms document the design trade-offs but **do not grant** Protos permission to evaluate code or inspect extra authority. The D227 packet explicitly screened Espresso, Sulong, SimpleLanguage and Pkl for relevance rather than claiming they have no completion features.

## Counterexamples and acceptance evidence to obtain

1. Successful `x: 1` yields a current live name; ordinary `context.removeSlot("x")` removes it; a subsequent TAB cannot use stale history. Recreate the slot and verify recovery; a PRESENT `null` slot is not treated as absent.
2. A failed/partially executed input can still mutate the current context; read the actual post-failure presence, not the statement's success/failure flag.
3. A nearer lexical slot shadows prelude; removal can reveal an outer name. Receiver fallback and context delegation must not be mislabeled as proven lexical names.
4. TAB cannot invoke `import()`, evaluate a getter or arbitrary expression, read guest values, materialize deferred Contexts or retain guest objects.
5. Closing Process/Context or losing Actor admission must fail closed and preserve typed input; no global synchronization or cross-Actor name collection.
6. Test non-ASCII identifiers, NFC, comments, strings, cursor-in-middle, multiline pending source and syntactically invalid/incomplete buffers. The approved baseline allows no candidate when the parser cannot prove a safe site.
7. JLine command completion does not insert unintended spaces; no-match/no-interactive output is stable; brackets/paste/history/Ctrl-C/Ctrl-D remain unaffected.
8. Human executes focused tests and **real TTY** acceptance for JVM and Native CLI, separate from this decision ratification.

## Approval invariant reconciliation

`DECISION_APPROVAL_PROVENANCE=PASS`: exact revised proposal was presented to and approved by the owner, 2026-10-31.

`DECISION_INVARIANT_CONSISTENCY=PASS_POLICY`: already-approved Protos lexical context lookup, prelude, module import, object mutation, Closure capture, Actor isolation, and exact static-provenance rules are retained; D227 adds an interactive-tool candidate selection contract **without changing their meaning**. No normative spec change is claimed. Full runtime coherence and real terminal behavior remain an **implementation evidence gate** owned by CLI011, not an assumption licensed by D227 ratification.

```text
WORK_ITEM=D227
OWNER_SELECTED=D227-A1_REVISED
APPROVAL_DATE=2026-10-31
PRODUCT_BASELINE=0ff5c9828e64b9a127d945f4e2cb98053dcb7c23
SOURCE_AUDIT=READ_ONLY
DURABLE_RATIFICATION_RECORD=decisions/tooling/D227_REPL_TAB_COMPLETION_CANDIDATE_AUTHORITY_AND_LIVE_SESSION_VISIBILITY.md
PRODUCT_FILES_MODIFIED=NO
PRODUCT_TESTS_EXECUTED=NO
REAL_TTY_ACCEPTANCE=NOT_EXECUTED
FOLLOW_UP=CLI011_ISSUE_988
```
