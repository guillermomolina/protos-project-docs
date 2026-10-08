# D194 — Canonical lint diagnostics, rule and fix policy

**Status: RATIFIED — modified Candidate B selected by the project owner on 2026-10-08.**

- Decision issue: [D194/#842](https://github.com/guillermomolina/protos/issues/842).
- Consumer: [LM012/#671](https://github.com/guillermomolina/protos/issues/671).
- Owner approval provenance: the project owner replied **"aprobado"** to the immediately preceding, explicit six-part proposal entitled **"modificar D194 antes de aprobarlo"** on 2026-10-08. This was approval of **modified B**, not of the earlier research B unchanged.
- Decision research: [D194 comparative research](../../evidence/D194/D194_LINT_POLICY_COMPARATIVE_RESEARCH.md), first published at `d376d00168c87b6b9456a03281361457bb2e07e2`.
- Source authority audit: [LM012-A0](../../evidence/LM012/LM012_A0_DIAGNOSTICS_LINT_AUTHORITY_AUDIT.md), first published at `739ece903a64bcd01d32229f61153d449510c207`.
- Product baseline inspected during research: `guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`.
- Nature: **non-normative, implementation-independent public tooling policy**. This record does **not** change Protos language syntax, evaluation, runtime semantics, or Standard Library semantics.

## Selected contract: modified Option B

The selected design is **small, proof-first, source-local and incremental**, with a strict separation between correctness rules, optional style checks, parser diagnostics, formatting, and potential explicit corrections. It is not a commitment to build a lint engine before a useful rule has been demonstrated.

### 1. Correctness rules and default enablement

A correctness rule is **eligible** for default enablement *only if* its claimed source fact is supported by a sound static proof over the rule's explicitly bounded domain **and** the warning provides justified, distinct practical value. Both conditions are necessary, **not sufficient**: a specific rule's inclusion, default behavior and severity still require an explicit, reviewable individual rule contract.

- A rule must give its proven proposition, exact applicable source form/scope, positive examples, negative/false-positive counterexamples, invalid-source handling and required proof authorities.
- Where semantics permit dynamic lookup, delegation, mutation, reflection, effectful invocation or matching, **absence of source-local proof is not proof of absence at runtime**. Omit speculative results.
- A rule must not claim a class-wide "unused"/"undefined"/"unreachable"/"redundant" fact from textual pattern similarity when soundness cannot be established.
- Do not create an empty default correctness profile solely to claim lint functionality. No initial correctness rule is selected by this decision.

Default activation is *per approved rule*, never by automatically promoting an untested category or by inferring owner approval from this policy.

### 2. Style rules and formatter separation

Style lint checks are **opt-in** and must offer distinct utility beyond the canonical formatter. Rules that merely report differences from D183/LM011 formatter output are not justified as lint by that difference.

Formatting, syntax validity, user style preferences and semantic correctness are separate authorities. This decision does not introduce a second style engine, autoformat-on-lint, global suppression/configuration rules, or an implicit editor style policy. Any concrete style check still needs its own rule-specific proof, ID, level and enablement contract.

### 3. Canonical diagnostic evidence and identity

Every approved lint diagnostic must have:

- a **stable, distinct rule identifier** and source/provenance identifiable as canonical Protos lint (separate from parser error identity);
- an **explicit severity** selected for that rule, never inferred from a generic label such as "correctness" or "style";
- a precise **source location/range**, computed from the exact immutable document/source snapshot with the canonical SourceSpan/UTF-16 position mappings where projected through LSP;
- a statement that is justified by the bounded proof and does not overclaim completeness beyond the analyzed source;
- deterministic reporting, with outdated/closed document results invalidated instead of published over a newer revision.

Parser/compiler errors remain owned by their existing authority (including LM009-G1). They are not relabeled or duplicated as new lint rules. A failed/incomplete source analysis must not produce conjectured lint warnings. Future editor, CLI or other consumers project the same editor-neutral rule identity and evidence; no TypeScript/editor-local semantic reimplementation is authorized.

This is a *rule identity and reporting contract*, not approval of a particular identifier namespace, serialized schema, severity mapping for a first rule, user configuration format or suppression syntax.

### 4. Quick fixes: conditional capability, not implicit edits

A quick fix is **permitted** only where the concrete transformation has an independently demonstrated proof that it preserves the approved semantics in the exact applicable source context, with an exact snapshot/version and edit-range applicability check. The user must **explicitly choose** the action before a document is edited.

- No automatic edits, background rewrites, default fix-on-save, indiscriminate `fixAll` or "safe" label without the proof.
- Every concrete proposed rule and rewrite must respect grammar-significant whitespace/newlines, literal/comment payloads, source-form preservation, Unicode offsets, and stale edits; conflicts/overlaps must be handled fail-closed.
- Providing a possible future safe-action seam is **not** approval of any first quick fix, any particular LSP `CodeAction`, or an editor capability advertisement.

A failed proof means no edit action, not an unsafe fallback.

### 5. Runtime cost and static architecture

Reuse PLAT024's editor-neutral static analysis core, real parser, immutable exact document snapshots, source spans and existing on-demand source-layout facts as sufficient for proven, local rules. A thin LSP server projects approved diagnostic outcomes, and the VS Code extension stays a client.

**Pay as you grow:** an ordinary Protos execution that does not invoke editor/lint tooling pays no new lint analysis, startup, context, Task, Actor, Process, thread, daemon, registry or indexing cost. No mandatory hosted guest runtime, speculative project-wide index or second TypeScript compiler is authorized. Extend storage/host lifetime architecture only on independently demonstrated need and under the appropriate PLAT decision gate.

### 6. CLI/CI remains a separate contract

D194 does **not** select `protos lint`, `protos check`, stdout/stderr format, exit codes, CI failure threshold, workspace discovery, public suppression/configuration files, new `TOOLxxx` ownership, or a mandatory invocation model. Such behavior requires its actual public-tool/CLI contract when the feature is justified and reaches that boundary. An LSP diagnostic alone is not an implicit CI error.

Likewise, this decision neither defines D096 matching exhaustiveness/redundancy nor reopens ratified language semantics, D110/D124 proof identity, LM009-G diagnostics or D183 formatting.

## Deliberately unresolved: the first useful lint rule

**No first lint rule has been approved**. This is intentional, not an unfinished policy selection.

Before implementing LM012-B, evaluate candidate checks for:
1. **distinct user value** not already delivered by the parser, formatter, existing editor navigation or other independent feature;
2. source-local **soundness** and explicit applicability/exclusion domain, with positive and adversarial counterexamples grounded in the current grammar/semantics;
3. stable rule ID and description, exact range and diagnostic severity;
4. explicit enablement choice under the approved correctness/style policy;
5. whether any proposed fix is semantically preserving in its precise context (otherwise no fix);
6. requested validation and regression evidence against the affected source/transport surfaces.

If no candidate meets both usefulness and soundness requirements, defer implementation of new lint diagnostics; do not pad the ruleset with trivial trailing-whitespace duplication of LM011 or speculative semantic claims. A selected first rule may be specified in a bounded LM012 rule-admission record and implementation handoff; any new substantive public-contract choice that exceeds D194 routes to its own approval gate.

In particular, **implementation readiness is conditional**: D194 removes the *general policy* blocker, but cannot by itself authorize a source-visible rule, default warning, quick fix or CLI mode that is not concretely selected.

## Invariant/delta consistency check (GITHUB021)

The following already-ratified or owner-selected constraints were checked against the **exact six-part modified B** approved on 2026-10-08. This ratification adds no hidden override of those constraints.

| Applicable authority/invariant | Modified B consequence | Check |
| --- | --- | --- |
| PLAT024: editor-neutral static core; thin client; no ordinary-runtime tooling cost | On-demand source-local analysis only; no new always-on guest/index | **PRESERVED** |
| LM009-G1: parser-derived standard LSP diagnostics | Parser errors not relabeled or duplicated as lint; exact version and clearing | **PRESERVED** |
| D110/D124: only proven static origin claims; dynamic ambiguity is not an exact negative fact | Every new rule requires its own sound bounded proof; no guessed symbol negatives | **PRESERVED** |
| D183/LM011: formatter is canonical for presentation and source-form preservation | Style lint is opt-in, nonduplicative; no implicit formatter-to-lint gate | **PRESERVED** |
| D096: matching exhaustiveness/redundancy remains independently unresolved | No matching-coverage lint selected | **PRESERVED** |
| Grammar/language/matching/closure semantics remain normative in `spec/` | Tool checks cannot redefine them or change guest evaluation | **PRESERVED** |
| Explicit owner approval does not automatically approve further public contracts | First rule, exact severity, default activation and CLI/CI behavior deferred | **PRESERVED** |

**D194-specific previous owner-approved invariants:** none in the initial research stage. The owner's six explicit modifications to option B are the current selected invariants. Compared with the earlier recommendation, the **material deltas** are: (a) sound and useful correctness rules can eventually become **default** after individual selection instead of a globally opt-in starting premise; (b) style remains opt-in; and (c) conditional **explicit, proven** quick fixes are permitted instead of blanket postponement/no edits. All were visible in the exact proposal to which the owner replied "aprobado"; no separate previously ratified D194 invariant is silently reopened.

## Candidate alternatives and rationale

The completed comparison includes ESLint, Rust/Clippy, Ruff, Go vet, gopls and LSP, with A (defer), B (conservative opt-in), C (default conservative core), D (full configurable engine), and rejected E (speculative inferred semantic lint), scored across all 12 GITHUB010 criteria with uncertainties in the linked research packet.

**Why modified B:** it retains B's correctness and minimal on-demand cost while allowing the eventual first *demonstrably useful* correctness warning to be enabled by default when individually approved. It avoids premature default noise, a sprawling configuration institution, and a misleading first rule that merely repeats formatting.

**Strongest objection:** there is no identified rule today that unquestionably adds value; a policy without rules can appear to deliver no user-facing feature. That is accepted deliberately: forcing an unsound or redundant check solely to justify tooling is worse, and later rule admission is incremental.

## Downstream state and non-actions

- D194 may be closed after exact durable publication and re-read of this owner-approved policy record and live coordination state.
- LM012 can return to **rule discovery/admission**, not direct implementation of unspecified lint. Its first useful rule must be independently evidenced and selected within this already-approved policy.
- Do not create a new Dxxx or PLATxxx for every small rule; promote independent design work only if a genuinely new substantive choice appears.
- No product source, normative specification, test, build, runtime, VS Code extension or product Git state is modified by this ratification.
- Native GitHub parent/sub-issue and `blocked by` mutations for D194/LM012 may remain pending if the connector cannot express them; text links are **not** native graph edges.

```text
D194_SELECTED=MODIFIED_B
OWNER_APPROVAL=2026-10-08_EXPLICIT
GENERAL_LINT_POLICY=RATIFIED
INITIAL_RULE_SELECTED=NO
FIRST_RULE_IMPLEMENTATION_AUTHORIZED=NO
QUICK_FIX_SELECTED=NO
CLI_CI_PUBLIC_CONTRACT_SELECTED=NO
SPECIFICATION_CHANGE=NO
```

## GITHUB015/GITHUB020 formal closure reconciliation — 2026-10-08

The earlier *native hierarchy pending* statements above are historical snapshots. GitHub's current native issue endpoints now independently verify both directions:

- `GET /repos/guillermomolina/protos/issues/842/parent` identifies [LM012/#671](https://github.com/guillermomolina/protos/issues/671).
- `GET /repos/guillermomolina/protos/issues/671/sub_issues` includes [D194/#842](https://github.com/guillermomolina/protos/issues/842).
- These actual native relations satisfy GITHUB015; the previous inability to mutate the relation using a connector is no longer a closure blocker.

**Decision complete:** Modified Candidate B's owner approval, GITHUB010 comparative record, GITHUB021 invariant checks and exact durable ratification have already been published. The general source-local, proof-first correctness/style/fix/snapshot/PLAT024 policy is settled. A new owner decision, normative language spec revision, or product edit is not required.

**Separate downstream work:** The owner separately approved two default-on Warning rules under [LM012-B1's exact rule admission](../../work/LM012/LM012_B1_INITIAL_LINT_RULES_OWNER_APPROVAL.md), and the initial source and LSP diagnostics were published as [Protos `a8027f6a78fac057f409103d8a79500faead72e2`](https://github.com/guillermomolina/protos/commit/a8027f6a78fac057f409103d8a79500faead72e2). That publication is not the D194 policy itself. [D195/#843](https://github.com/guillermomolina/protos/issues/843) separately ratified the public CLI/CI policy and is now closed. LM012/#671 remains the open, separately validated implementation owner, including LM012-C1; **closing D194 does not declare LM012 implemented, tested or complete.**

**Evidence provenance:** Existing D194 owner approval and ratification, published D194 comparison, the B1 rule selection, and the live native GitHub hierarchy were inspected. No agent-run builds, tests, benchmarks or product Git operations are asserted, nor is a new PASS inferred for B1 tests.

For GITHUB020, the coordinator must reread this exact published record and the owning issue, post a compact final issue comment with this repository revision, then close D194 as `completed` while retaining its formal D-family identity and LM012 native parent. GitHub Project state is derived by automation; no independent Project synchronization proof is asserted.

```text
ISSUE=D194/#842
OWNER_APPROVED_DECISION=MODIFIED_B_RATIFIED
GITHUB021_INVARIANTS=PRESERVED
GITHUB010_COMPARATIVE_RESEARCH=COMPLETE
NATIVE_PARENT=PASS_LM012_671
NATIVE_SUBISSUE=PASS_D194_842
DURABLE_RECORD_DECISION=REQUIRED
CLOSURE_EVIDENCE_IDENTIFIED=PASS
IMPLEMENTATION_OWNER=LM012_671_INDEPENDENT
D195_843=CLOSED_COMPLETED
NO_NEW_PRODUCT_CHANGES=YES
FINAL_NEXT_ACTION=GITHUB020_COMMENT_THEN_CLOSE_D194_COMPLETED
```
