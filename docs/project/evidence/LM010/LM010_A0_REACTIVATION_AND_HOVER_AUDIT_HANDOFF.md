# LM010-A0 — activation and hover authority audit handoff

Evidence date: **2026-10-08**

Owning work item: [LM010 / guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493)

Nature: snapshot-like **coordination and investigation handoff**, not design ratification, product implementation, or validation evidence.

## Exact reference baseline

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REFERENCE_HEAD_AT_ACTIVATION=3bb1278d91ee5cea98031462be2a5c4dd3c89019
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
PROJECT_RECORD_HEAD_BEFORE_PUBLICATION=eb73241405b61365fc07a21de28c966a483e61d4
LM010_ISSUE=493
LM010_STATUS=IN_PROGRESS
LM010_PRIORITY=P2_NORMAL_ACTIVE_WORK
NEXT_SLICE=LM010-A0
TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO
TESTS_EXECUTED=NO
BUILDS_EXECUTED=NO
PRODUCT_CODE_CHANGED=NO
SPECIFICATION_CHANGED=NO
DESIGN_CANDIDATE_SELECTED=NO
```

The product reference revision is a **snapshot**, not permission to assume a later HEAD has not moved. No code, test, release, specification, or decision-publication outcomes are claimed by this record.

## Project-owner direction

The owner selected LM010 over LM012 for work to start on 2026-10-08. LM010 was already an independently tracked formal work item and therefore does not need a duplicate Issue for this bounded initial investigation. LM010 is not a dependency of LM009 closure. LM012 remains distinct.

The first focus is the smallest trustworthy hover capability; later LM010 completion and signature-help scopes are not thereby approved. The intent is to avoid micro-slices: after audit and any necessary design approval, prefer one coherent implementation slice spanning the semantic adapter, standard LSP wiring, tests and bounded editor acceptance where feasible.

## Existing authorities to re-check at current HEAD

- [LM010 #493](https://github.com/guillermomolina/protos/issues/493) explicitly defers hover/completion/signature help and forbids invented nominal types, runtime-derived guesses, and duplicated TypeScript semantics.
- [PLAT024 #342](https://github.com/guillermomolina/protos/issues/342) supplies the thin standard-LSP process/hosting boundary for source analysis without executing guest code.
- [D124 #491](https://github.com/guillermomolina/protos/issues/491), informed by D110, governs exact statically provable references/definition identity and limits unsound name-based fallbacks.
- [LM009-H #489](https://github.com/guillermomolina/protos/issues/489) records bounded same-snapshot identity proof and fail-closed references acceptance.
- `src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java` currently documents LM010 as deferred; definition, references and formatting providers already exist.
- `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` reuses immutable editor document snapshots, definition/references static analysis, canonical source authority and UTF-16 position conversion.
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java` is the static-session implementation surface to examine.
- `src/main/java/com/guillermomolina/protos/lsp/ProtosLspSourcePositions.java` owns existing LSP source-position conversion.

These are **audit entry points**, not exhaustive evidence of what hover can currently prove. The subsequent investigation must also read the current normative specification owners, the full affected static analysis/resolver code and its tests, the relevant decision records and current head, without substituting old issue prose for current source truth.

## LM010-A0 investigation contract

Read `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, and `AGENTS.work/DESIGN.md` only if a substantive design checkpoint arises. Respect the current governing specification and durable blocker ledger. This task is **investigation only**: no commands, builds, tests, programs, state-changing Git actions, file edits, repository mutations or speculative code implementation.

Produce an evidence table for each potential hover datum: source position category; exact static authority; proof of validity and completeness; ambiguity/barrier behavior; staleness and source ownership; UTF-16/range mapping; and counterexample. Include at least lexical/Closure parameter, match Binder/Alias, named source origin, literals, exact callable shape, module/package identity, receiver/delegation/dynamic member, invalid source, unsaved buffer, and no-project/out-of-domain cases. Unsupported cases must be explicitly unsupported, not filled with guesses.

Separate all mechanically determined work from substantive public editor behavior choices (including partial/ambiguous hover text, documentation provenance, signature or type claims, range, unknowns, and absence behavior). If a real public contract is unsettled, propose the appropriate new Dxxx gate following allocation rules rather than selecting it; if durable hosting/index/lifetime architecture must change, propose PLATxxx. Reuse already-ratified decisions rather than reopening them.

Recommend a bounded follow-up implementation **only if** the audited source, tests and decisions already determine the behavior. That implementation should operate solely in `guillermomolina/protos` from the human's actual checkout at HEAD, with no repository changes to `guillermomolina/protos-vscode-extension` unless a later, independently justified need is found. The editor remains a thin standard LSP client. Runtime execution without a language server must pay no static-intelligence overhead.

## Completion criteria for this handoff

```text
LM010_A0_STATUS=READY_FOR_INVESTIGATION
D_OR_PLAT_DECISION_APPROVAL=NOT_GRANTED
LM010_IMPLEMENTATION=NOT_RELEASED
LM010_PARENT_CLOSED=NO
NEW_FORMAL_ISSUE_REQUIRED_NOW=NO
NEXT_HUMAN_EXECUTOR_ACTION=NONE_FOR_INVESTIGATION
```

## Materially inspected to prepare this activation

- `guillermomolina/protos:AGENTS.md`
- `guillermomolina/protos:AGENTS.work/COORDINATION.md`
- `guillermomolina/protos:AGENTS.work/IMPLEMENTATION.md`
- `guillermomolina/protos:AGENTS.work/DESIGN.md` (decision workflow sampled, not full decision audit)
- `guillermomolina/protos:AGENTS.work/REFERENCE.md` (role-first paths)
- `guillermomolina/protos#493`, `#489`, `#342`, `#491`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java` (targeted review)
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` (targeted review)
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/lsp/ProtosLspSourcePositions.java` (targeted review)
- `guillermomolina/protos-project-docs:AGENTS.md`, `docs/project/README.md`, `docs/project/evidence/README.md`, and the LM011-D1 sample evidence

No complete normative/source hover audit was performed here; that is exactly the scope of LM010-A0. In particular, no performance, test, Git diff, or compiled behavior was validated.
