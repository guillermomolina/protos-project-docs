# D228-A1 — Owner approval and source/invariant audit (2026-10-31)

**Role:** durable owner-selection provenance, comparative source audit and GITHUB021 reconciliation. **Not** an I134 implementation, test or runtime acceptance record.

- Formal owner: [D228 #1001](https://github.com/guillermomolina/protos/issues/1001); consuming implementation: [I134 #1002](https://github.com/guillermomolina/protos/issues/1002).
- [Ratified decision](../../decisions/tooling/D228_REPL_RECEIVER_SELECTOR_TAB_COMPLETION_AUTHORITY_AND_DELEGATION.md) and [previous D227-A1 revised](../../decisions/tooling/D227_REPL_TAB_COMPLETION_CANDIDATE_AUTHORITY_AND_LIVE_SESSION_VISIBILITY.md).
- Exact product inspection baseline: [`d555458978c2979a496b36bfa2df19dc964da4c3`](https://github.com/guillermomolina/protos/commit/d555458978c2979a496b36bfa2df19dc964da4c3); accepted CLI011 release baseline: [`b50e256e0f48a20a19a0731116b131871fe2a585`](https://github.com/guillermomolina/protos/commit/b50e256e0f48a20a19a0731116b131871fe2a585).
- **Explicit owner approval on 2026-10-31**, following the exhaustive D228 decision packet. Verbatim original approval:
> Apruebo D228-A1: autocompletado de selectores `SurfaceMember` en el REPL mediante observación restringida de referencias léxicas existentes y `this`, con agrupación sintáctica; nombres de slots locales y heredados siguiendo la delegación ordinaria, sin distinguir métodos de atributos; sin evaluación guest, introspección extranjera ni materialización de contextos; bajo admisión coherente del Actor/Process, presupuestos acotados, fallo cerrado y comportamiento equivalente en JVM y Native. Mantener D227-A1 para nombres léxicos y posponer las demás formas de receptor. Ratificar D228 en `guillermomolina/protos-project-docs` antes de desbloquear I134 como un solo slice.

This is exact D228-A1 approval, including the bounded receiver-value permission and constraints presented before the question, not permission for guest evaluation, new language semantics or arbitrary interop.

## Sources inspected

Project instructions: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, D227 durable approval and D228/I134 Issues.

Normative owners: `spec/PROTOS_GRAMMAR.md` (§2 Unicode 17 XID/NFC and contextual reserved member names; §18 member postfix); `spec/semantics/OBJECT_MODEL.md` (immutable single parent, ordinary delegation, own-local `slotNames()`); `EXECUTION_AND_CONTROL.md` (genuine lexical chain, `this`, receiver fallback); `CALLABLES.md` (Closure-only executable value, extracted-method identity, receiver validity); `MODULES.md` (module context and explicit import); `spec/concurrency/ACTORS.md` (one Actor turn and isolation); `spec/io/PROCESS_IO.md` (Process/RootActor lifetime). Exact product references use the baseline above.

Implementation/source audit: `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`, `ProtosReplSourceCompletion.java`, `ProtosReplJLineCompleter.java`, `ProtosReplCompletion.java`; `analysis/ProtosStaticCompletion.java`, `ProtosStaticDefinitions.java`, `ProtosStaticHover.java`, `parser/ast/SurfaceMember.java`; `runtime/ProtosActivation.java`, `lexical/ProtosLexicalEnvironment.java`, `ProtosObjectValue.java`, `ProtosValueLookup.java`, `ProtosRepresentedValue.java`, `lexical/ProtosLexicalBindingAuthority.java`, `ProtosLexicalBindingAuthorityCalls.java`, `ProtosMapBackedLexicalBindingAuthority.java`, `execution/bytecode/lowering/ProtosFrameLexicalBindingAuthority.java`, `execution/host/embedding/ProtosPolyglotProcessContext.java`, `ProtosPolyglotExecutionContext.java`, `execution/host/tooling/ProtosDebuggerScope.java`; `src/test/java/com/guillermomolina/protos/cli/ProtosReplSourceCompletionTest.java`, `ProtosReplJLineSourceCompletionTest.java`, `ProtosReplJLineCompleterTest.java`; `docs/guide/tools/cli.md` and `pom.xml` (JLine 3.30.6).

Important findings: D227 provider explicitly rejects member sites; a new independent `SurfaceMember` proof is necessary. ProtosObjectValue's names-only append avoids reading every slot value, whereas `localSlotsSnapshot()` retains values. ProtosValueLookup supports heterogeneously represented semantic parents; `ProtosObjectValue.lookupSlot()` alone cannot traverse all of them. Polyglot Context's lifecycle read-lock is not an exclusive Actor admission proof; the persistent RootActor gate needs ownership verification. Debugger scope may materialize activation contexts and read values; it is not TAB authority.

## Comparative and falsification provenance

Research compares A static-only, B local-only, C ordinary delegated names, D evaluation, E debugger/interop/LSP and F conservative rollout, with the complete twelve-dimension 1–5/confidence scorecard preserved in the decision. Source precedents: GraalPy `rlcompleter.py`, TruffleRuby IRB, GraalJS/Node REPL, JShell, Elixir IEx, Apple Pkl, JLine 3; Espresso, Sulong and SimpleLanguage surveyed for relevance. Their dynamic evaluation and inspection permissions are not transferred to Protos.

Falsification requirements: local data shadows inherited Closure, remove/recreate restores delegation precedence, failed/partial mutations refresh on next TAB, semantic `null` remains PRESENT, invalid receiver-domain standard method is only a selector, forbidden calls/getters/imports never run, no foreign or other Actor inspection, no deferred context materialization, no retained guest references, bounded names/depth, Unicode/reserved after-dot selectors, multiline/in-word JLine, JVM/Native PTY and noninteractive silence.

## GITHUB021 and future publication gate

D227's names-only *bare* lookup provider stays unchanged. Its former prohibition on receiver-value reads is **explicitly, narrowly reopened** only for a parser-proven `SurfaceMember` completion request under coherent Actor/Process admission. No guest evaluation, method-value classification, deferred context materialization, context delegation, import or foreign authority is admitted. OBJECT_MODEL, CALLABLES, EXECUTION_AND_CONTROL, MODULES, grammar and Actor isolation remain unchanged. **GITHUB021_INVARIANT_CONSISTENCY=PASS_FOR_EXACT_APPROVED_POLICY**.

This research did not execute local/remote programs, Java tests, `make test`, `make check`, or real JVM/Native TTY validation. The existing gate is not automatically proven to exclude all concurrent mutating turns: I134 must prove it or stop for a further owner decision. Product revision is inspection-only, not implementation evidence.

The role-first durable files under `decisions/tooling/` and `evidence/D228/` establish policy and approval provenance; after their exact docs commit is verified, D228 may close with a closure-evidence Issue comment and I134 may become `status:ready` for **one** implementation slice, not an already-completed change.

```text
WORK_ITEM=D228
OWNER_SELECTED=D228-A1
OWNER_APPROVAL=EXPLICIT_2026-10-31
PRODUCT_INSPECTION_REVISION=d555458978c2979a496b36bfa2df19dc964da4c3
SOURCE_AUDIT=READ_ONLY
DURABLE_RECORD_DECISION=REQUIRED
GITHUB021_INVARIANT_CONSISTENCY=PASS_POLICY
PRODUCT_FILES_MODIFIED=NO
PRODUCT_TESTS_EXECUTED=NO
REAL_TTY_ACCEPTANCE=NOT_EXECUTED
FOLLOW_UP=I134_ISSUE_1002_ONE_SLICE
```
