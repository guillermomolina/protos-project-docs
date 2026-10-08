# LM012-C0 — Public lint CLI/CI contract research, comparison and owner approval

**TYPE=INVESTIGATION; COMPLETED 2026-10-08.** **NO PRODUCT IMPLEMENTATION / NO TEST OR BUILD EXECUTION.**
**Consumer:** [LM012/#671](https://github.com/guillermomolina/protos/issues/671).
**Decision:** [D195/#843](https://github.com/guillermomolina/protos/issues/843), [owner-approved canonical contract](../../decisions/tooling/D195_SINGLE_SOURCE_LINT_PUBLIC_CLI_CI_CONTRACT.md), first decision record publication `549bff4a4bc925ebd3c8e8295b7c5018d71a09b3`.
**Product baseline and verified LM012-B1:** [`a8027f6a78fac057f409103d8a79500faead72e2`](https://github.com/guillermomolina/protos/commit/a8027f6a78fac057f409103d8a79500faead72e2) / `0.3.295-SNAPSHOT`.
**Moving product HEAD reviewed:** [`1d6d537d79d69fae3ad1e613cefd76b7a97abe38`](https://github.com/guillermomolina/protos/commit/1d6d537d79d69fae3ad1e613cefd76b7a97abe38). GitHub compare `a8027f6a..1d6d537d` shows precisely `spec/io/PROCESS_IO.md` and `spec/PROTOS_SPEC_CHANGELOG.md` (PLAT054-3E3-SPEC). This does not change analyzed CLI/lint sources; no conflict with the design baseline found in this delta.
**Governing policy:** [D194 ratified modified B](../../decisions/tooling/D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md) and [LM012-B1 owner rule selection](../../work/LM012/LM012_B1_INITIAL_LINT_RULES_OWNER_APPROVAL.md). Formatter: D183, D184, LM011. Static core: PLAT024. Bundled-tool boundaries: `protos:docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md`, `AGENTS.work/TOOL.md`. Research method: `AGENTS.work/DESIGN.md` GITHUB010 and GITHUB021.

## Verified source audit

Files read via GitHub at the exact product baseline:

| Source | Verified findings |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | `SUBCOMMANDS` includes language-server/debug/run/package/test/format, not lint/check. `run()` creates a dedicated `protos-cli-guest` carrier with `GUEST_CALL_STACK_SIZE_BYTES` *before* `dispatchCommand`. `formatDocument` strictly reads one UTF-8 file or stdin, then opens `ProtosPolyglotRuntimeHost`; not an appropriate default for pure AST lint. Usage=2; internal=70; `format` input or parser fail=1. |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java` | `parse(snapshot)` is a real parser-only, editor-neutral `Parsed`/`Failed` result. `lint(snapshot)` reparses and returns no lint findings when parsing fails. |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLint.java` | `check(Parsed)` traverses one Surface AST, does not execute guest code, and sorts by `span.startOffset`, `span.endOffset`, `ruleId`; applies only the two sound approved LM012-B1 rules. |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLintDiagnostic.java` | Finding retains exact snapshot, rule ID, `Severity.WARNING`, span, message. |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticParseResult.java` | `Parsed` holds AST; `Failed` holds message/span/`unexpectedEndOfSource`; not a public diagnostics schema. |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosDocumentSnapshot.java` | Opaque document ID + version + immutable source text; no project path semantics. |
| `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` | One current-snapshot publication per open document: parser Error **or** lint Warnings with `source=protos`, rule `code` for lint, exact UTF-16 ranges; stale snapshots suppressed, close clears findings. |
| `src/main/java/com/guillermomolina/protos/lsp/ProtosLspSourcePositions.java` | Existing exact 0-based UTF-16 and CRLF position conversion; LSP class is package-private, so a shared neutral mapping or carefully reused logic may be needed without importing LSP transport into core. |
| `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticLintTest.java` | Checked source for direct nonlocal-return suffix, strict fresh-object identity, exclusions, deterministic ordering and snapshot validity. |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java` | Checked source for parser failure/recovery, warning ID and severity, close, UTF-16 and CRLF. |
| `src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java` | Existing formatter stdout, stdin, invalid-source echo/fail, I/O and usage assertions; basis for CLI-specific new tests. |

The LM012-B1 commit includes both test source files. **This investigation did not execute tests or receive a maintainer PASS report**; committed tests are not proof of test execution. The source and design show no mandatory guest runtime for pure AST lint; **actual startup/memory cost was not measured**.

## Problem and exact decision boundary

LSP diagnostics operate over active editor buffers and are not an ergonomic contract for CI, shell input or committed saved files. A command could run the **same** existing source-local proof without an editor or language-server process. The decision was whether this limited incremental value justified a public CLI name, output shape and exit codes, NOT whether the current two rules were valid (already approved and implemented). A new check command must not assert whole-program semantic correctness that the Protos static core does not prove.

The candidate set:

- **A, LSP-only:** status quo; zero new implementation and user-facing commitments, but no CI contract.
- **B, single-source lint:** explicit one document; parser+approved lint via one canonical static-core result; text/JSON; opt-in warning failure. Small CLI adapter, no runtime guest.
- **C, general `check`:** might grow to parse/module/type/package analyses; invites unratified completeness semantics and broader contract now.
- **D, official bundled Protos tool:** follows the general toolchain model for higher-level policy but risks starting a guest runtime simply to call a Java-resident parser and already-implemented rules.
- **E, project analysis:** multi-file discovery, recursive scanning, external package versions, aggregation and configuration; lacks demonstrated immediate demand, commits Protos early to project identity and source acquisition policies.

B and future D can coexist if a genuine guest-level orchestration requirement appears; **do not build D now merely to preserve the possibility**. No choice requires a second parser or LSP subprocess.

## Official comparative precedents

1. **ESLint**, official CLI [command line](https://eslint.org/docs/latest/use/command-line-interface): `--max-warnings` independently decides whether warnings trigger nonzero status; output formats and stdin supported. Protos takes only the separation of severity versus CI gate, not plugin framework/config/recursive discovery.
2. **Ruff**, official [linter](https://docs.astral.sh/ruff/linter/): `ruff check` has distinct exit outcomes and machine-facing output; its broad Python file/project capabilities are not justification for Protos' initial scope.
3. **Rust Clippy/Cargo**, [usage](https://doc.rust-lang.org/clippy/usage.html) and [CI](https://doc.rust-lang.org/clippy/continuous_integration/index.html): `-D warnings` (or newer Cargo warnings policy) elevates CI failure without changing the original lint rule. Cargo integration is appropriate for a mature crate graph, not necessary for one Protos source.
4. **Go vet**, [official package](https://pkg.go.dev/cmd/vet): independent CLI over existing analyzers; nonzero on issues/invalid invocation. Vet expressly makes no complete correctness guarantee; Protos should likewise avoid overclaiming.
5. **gopls**, [diagnostics](https://go.dev/gopls/features/diagnostics) and [analyzers](https://go.dev/gopls/analyzers): parser/compiler diagnostics and existing analyzers coexist under one editor diagnostic transport. Protos should share analysis authority, not run LSP itself as a CLI or create a separate editor-side linter.

These represent CLI-first configurable lint, source/toolchain integrated lint and analyzer shared between editor and command-line families. None proves Protos must copy its entire ecosystem.

## GITHUB010 12-dimensional 1–5 comparison

Scores are **qualitative** comparison aids, not benchmark measurements. H=high confidence, M=medium, L=low. Higher is preferred. The most uncertain cost and scalability scores reflect absent measurements.

| # / Dimension | A LSP-only | B lint | C check | D bundled | E project |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness/invariants | **5 H** no new contract | **5 H** canonical parser+rules | **4 M** broader claim risk | **4 M** host bridge required | **3 M** more ownership |
| 2 Protos philosophy | **4 H** minimal | **5 H** direct small mechanism | **4 M** abstract label | **3 M** runtime indirection | **2 H** institutions |
| 3 Present proportionality | **5 H** zero new costs | **5 M** narrow adapter | **3 M** larger promise | **2 M** guest bootstrap | **1 H** infrastructure |
| 4 Incremental growth | **3 H** CI still missing | **5 H** additive | **4 M** expansion potential | **4 M** reusable tool model | **3 M** early burden |
| 5 Future resilience | **4 H** extend later | **5 M** source identity preserved | **4 M** broader namespace | **4 M** existing model | **3 M** baked-in project |
| 6 Scalability | **2 H** editor buffers | **3 M** one source | **4 M** extensible | **4 L** orchestration | **5 L** potential breadth |
| 7 Conceptual simplicity | **5 H** no change | **5 M** one report | **3 M** vague scope | **2 M** extra layers | **1 H** policy pile |
| 8 Portability | **4 H** protocol | **5 M** no guest ABI | **4 M** portable goal | **3 M** runtime hosting | **3 M** filesystem graph |
| 9 Resource cost | **5 H** zero extra | **4 M** on demand | **3 M** variable | **2 M** likely guest startup | **2 M** scan/index |
| 10 Operability | **3 H** editor dependence | **5 M** explicit failures | **4 M** more domains | **3 M** bootstrap failures | **3 M** partial results |
| 11 Deferral/reversibility | **3 H** CI postponed | **5 M** additive later | **3 M** public broad meaning | **3 M** bridge contract | **2 M** complex migration |
| 12 Evidence maturity | **5 H** implemented | **5 M** static core exists | **4 M** ample precedent | **3 L** bridge unproven | **3 M** precedents only |

Qualitative gates: D and E fail **pay for what you need** with their current guest/project overhead; C risks **underdefined completeness semantics**. A would be correct if CI is not yet useful, but defers an explicit, otherwise inexpensive workflow. B is selected for its bounded practical value rather than for a numeric total.

### Counterexamples / failure modes

- Treating an empty `lint(snapshot)` return as "valid" is **wrong**: that API deliberately fails closed after a parser error. Use `parse` outcome first.
- Calling `parse` and then `lint(snapshot)` reparses unnecessarily; use `check(Parsed)`.
- Promoting Warnings to `Error` when `--fail-on-warning` is present changes D194 semantics. Change only exit status.
- Project/package scanner would need source identity, symlink traversal policy, dependency graph/version authority and partial-failure aggregation. Defer; source identity in JSON leaves an additive path.
- `protos format` opens `ProtosPolyglotRuntimeHost` and has D184-specific source echo and input exit=1 behavior. **Do not clone** those properties into lint except the explicitly shared file/stdin and read-only idea.
- Mandatory guest carriers, root Process, Context, Tasks or Actors for static AST would violate pay-as-you-grow. A limited static fast path for `ProtosCli.run()` is a mechanical possibility to validate without weakening guest execution.
- Parse error position must remain canonical; conversion of Java UTF-16 offsets with CRLF must match existing LSP output and be regression-tested with emoji, supplementary characters and final newlines.
- New false-positive rules or imagined dynamic-name/type checks have no authority under LM012-C0.

### Future scenarios and escape paths

- **Many files/CI:** the one-file command may run repeatedly; later add explicit multiple operands and a result envelope keyed by source when demanded. The first-version per-document JSON and stable rule IDs remain reusable.
- **Huge repositories/multicore:** add bounded aggregation, concurrency and indexing only after specifying file/root identity; no requirement to pre-create workspaces/Actors now.
- **Distributed execution:** file-local reports can be aggregated by an external coordinator; no claim of cross-file completeness.
- **Other runtime/backend:** report JSON/text and host-neutral spans, not serialized Java AST/Truffle objects; any backend can reproduce the contract.
- **Cancellation, failures, symlinks and encodings:** fail on malformed UTF-8 or inaccessible input rather than returning empty clean results; do not follow directory trees.
- **Future analyzer precision:** rule additions follow D194 per-rule gates, not automatic onboarding through this CLI.
- **Potential regret in B:** the command may later need whole-project semantic checking and separate `check` namespace; escape is adding that command without making `lint` claim more than it does.
- **Strongest objection:** only two rules exist, so CLI utility might not justify maintaining a new public format. Owner explicitly chose the limited initial CLI/CI value rather than continuing A.

### Smallest sufficient / deferral cost

Only source acquisition, a single canonical parse, existing rules, diagnostic projection and exit status are required. Text and versioned JSON are justified by both interactive and machine consumption. The result schema is the main public compatibility commitment. No plugin/config/rule engine/new AST is required. Deferring multi-file, SARIF, project discovery, bundled guest orchestration and code actions has bounded additive cost; implementing them early would pay permanent interface/runtime costs without demonstrated need.

## Owner approval and selected D195

Immediately following this complete recommendation, the project owner answered **“aprobado”** and requested formal GitHub Issue updates, direct docs publication, and the next slice prompt. The recommendation explicitly listed a nine-point candidate B approval package and the proposed JSON v1 / exit 0,1,2,3,70 contract. Accordingly the [D195 ratified record](../../decisions/tooling/D195_SINGLE_SOURCE_LINT_PUBLIC_CLI_CI_CONTRACT.md) documents the **approved** contract (not merely a speculative proposal). This approval is distinct from the earlier D194 modified-B general policy and the LM012-B1 two-rule approval.

GITHUB021 invariant preservation checked against D194 proof-first Warning rules; LM009-G1 parser authority; D110/D124/D096 static proof limits; D183/D184 formatting separation; PLAT024 zero unused-tool cost; and one public driver with a small host adapter. **No contradiction or silent override** found.

## Routing / what is not evidenced

```text
LM012_C0=RESEARCH_COMPLETE
OWNER_SELECTED=B
PUBLIC_CONTRACT_DECISION=D195
D195_APPROVAL=EXPLICIT_2026-10-08
LM012_C1=READY_AFTER_DURABLE_RATIFICATION
PRODUCT_SOURCE_CHANGED_BY_THIS_WORK=NO
NO_TESTS_RUN=YES
NO_BUILDS_RUN=YES
NO_PRODUCT_GIT_OPERATIONS=YES
D195_NATIVE_PARENT=NOT_VERIFIED
```

D195 is a substantive public-tool approval checkpoint, unlike a mechanical CLI adapter slice; under GITHUB010/GITHUB015 it legitimately owns a separate formal Issue and durable record. `LM012-C1` stays a single grouped **IMPLEMENTATION** slice inside LM012 (no `TOOLxxx` or `CLIxxx` now). Implementer assumes actual product HEAD and reads only its local repository, which contains the authoritative source/design/test context; no external live web or other repo required. The human executor runs builds/tests/git and validates the user-visible outcomes before version/changelog and push.
