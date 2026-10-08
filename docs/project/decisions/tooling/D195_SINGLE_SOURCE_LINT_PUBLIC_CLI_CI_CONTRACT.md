# D195 — Single-source lint public CLI and CI contract

**Status: RATIFIED — candidate B, owner-approved on 2026-10-08.**

- Decision issue: [D195/#843](https://github.com/guillermomolina/protos/issues/843).
- Consumer: [LM012/#671](https://github.com/guillermomolina/protos/issues/671), implementation slice LM012-C1.
- Owner approval provenance: in the active conversation following the completed **LM012-C0** report, the project owner said **"aprobado"**. This was explicit approval of the proposed candidate B and its nine contract points, including `--output json`, `--fail-on-warning`, exit code `3` and host-only static dispatch. The owner also authorized publishing project-docs records and updating GitHub coordination. This does not ratify other rules, code actions, format/check commands or project analysis.
- Governing decisions: [D194 policy](D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md), LM012-B1 [approved rule scope](../../work/LM012/LM012_B1_INITIAL_LINT_RULES_OWNER_APPROVAL.md), D183/LM011 formatter separation, [D184](D184_CANONICAL_FORMATTER_PUBLIC_CLI_CONTRACT.md), PLAT024 and the selected common toolchain architecture.
- Comparative and moving-HEAD evidence: [LM012-C0 research](../../evidence/LM012/LM012_C0_PUBLIC_LINT_CLI_CI_RESEARCH_AND_APPROVAL.md).
- Protos static implementation baseline: `guillermomolina/protos@a8027f6a78fac057f409103d8a79500faead72e2`, `0.3.295-SNAPSHOT`. Later inspected HEAD `1d6d537d79d69fae3ad1e613cefd76b7a97abe38` changes only `spec/io/PROCESS_IO.md` and `spec/PROTOS_SPEC_CHANGELOG.md` (PLAT054-3E3-SPEC), so no conflicting LM012/CLI delta in that interval.
- Scope: **public tooling transport/policy, not normative language semantics**.

## Exact owner-selected minimal public surface

```text
protos lint [--output text|json] [--fail-on-warning] [<file>]
```

Zero source operands mean read **one** source from standard input; one operand names **one** explicit source file. Accept UTF-8 text only; reject malformed input rather than silently substituting Unicode characters. Read-only explicit symlinks resolving to regular files are eligible. A directory, missing or unreadable file, or wrong text encoding is an input failure; multiple source operands, unknown options, invalid flag values and incompatible invocations are usage failures. No directory or workspace discovery; no implicit user package/module resolution. The operand spelling is transported as `source`, with `<stdin>` for stdin. This is not semantic Protos module identity.

Lint uses the existing parser and exactly the **two currently owner-approved default-on Warning rules**:
- `protos/unreachable-after-nonlocal-return`;
- `protos/always-different-fresh-object`.

Build a single immutable `ProtosDocumentSnapshot`, call `ProtosStaticAnalysisCore.parse(snapshot)` **once**, and use `ProtosStaticLint.check(parsed)` only on `ProtosStaticParseResult.Parsed`. Do not call `parse(snapshot)` and then the convenience `lint(snapshot)` (the latter reparses). On `Failed`, report only the canonical parser error, not an empty clean result or speculative lint warnings. The parser controls invalid/incomplete source; lint remains separate. No new rule, suppressions, rule configuration, inferred type/lookup/match claims, autofix, CodeAction or source edits.

### Diagnostics and projection

A diagnostic carries:
- origin `parser` or `lint`, never rebranding parser errors as a lint rule;
- stable canonical lint `code` (or JSON null when the existing parser error has no public code);
- canonical `error` or `warning` severity, not modified by an exit-code flag;
- original canonical message and exact SourceSpan over the acquired immutable source;
- zero-based `line`/`character` UTF-16 positions with exclusive `range.end`, matching the LSP projection for Unicode, CRLF and EOF;
- deterministic ordering by range start, range end, then code (define a stable null-code tie break; parser failures yield one parser error).

Output only to **stdout** for the diagnostic report, including parser errors and lint warnings; operational errors (usage/input/internal) go only to **stderr** and must leave stdout empty. Ordinary successful clean text output may be empty; `--output json` always emits the contracted structured report for successfully acquired input even when zero diagnostics exist. User module code is **never executed**.

Baseline JSON contract:

```json
{
  "schemaVersion": 1,
  "source": "example.protos",
  "status": "valid",
  "diagnostics": [
    {
      "origin": "lint",
      "code": "protos/always-different-fresh-object",
      "severity": "warning",
      "message": "diagnostic message supplied by the canonical analyzer",
      "range": {
        "start": {"line": 0, "character": 0},
        "end": {"line": 0, "character": 14}
      }
    }
  ]
}
```

`status` is `valid` for a parsed document, including documents with warnings, and `invalid` for a parser failure. Such invalid parsed-input diagnostics appear on stdout with exit 1; operational source-input/CLI/internal failures are not valid diagnostic JSON results and appear only on stderr with their respective exit. `schemaVersion` is numeric `1`; `diagnostics` is always an array. JSON string escaping, UTF-16 coordinates, a single top-level JSON value, and deterministic ordering must be regression-tested. Human-oriented text must make source, location, severity, origin/code (where available) and canonical message intelligible; exact ornamental text punctuation is **not** a new independent language/tooling semantic and should not be frozen beyond the approved fields.

### Exit statuses

| Status | Meaning |
| --- | --- |
| `0` | Source successfully parsed; no blocking findings; default warnings are allowed |
| `1` | Parser failure, **or** at least one Warning with explicit `--fail-on-warning` |
| `2` | CLI usage/options/operand count error |
| `3` | Source acquisition failure (including missing/not-regular/unreadable file or malformed UTF-8 stdin/file) |
| `70` | Unexpected internal/host fault |

`--fail-on-warning` only affects the process status; it does not change the emitted severity, code, message, ordering, stdout/stderr discipline or eligibility of warnings. Parser invalidity always fails regardless of that flag. The newly selected input-error status `3` is **specific to `protos lint`**: D184's already-published `protos format` contract retains its original input error exit `1`, and existing general commands are not silently modified.

### Host/runtime boundary and pay-as-you-grow

This is a thin public driver/static-analysis **adapter**, not a second compiler or an independently governed guest tool:
- no Truffle Context, Polyglot runtime host, guest Process, Task, Actor, guest library or execution of user modules merely to inspect an AST;
- no separate TypeScript analyzer or LSP protocol subprocess; use existing editor-neutral Java parser/analysis;
- no mandatory new thread/carrier merely for a static CLI; current `ProtosCli.run()` starts the guest carrier before dispatch, so allow a **mechanical dispatch refactor** for static paths while preserving existing guest-execution stack safety, stdout/stderr and command behavior;
- no extra work in ordinary Protos program execution when lint is not selected;
- no speculative index, runtime registry, plugin/config mechanism, project discovery or multi-file orchestration.

The shared toolchain design's preference for official Protos bundled tools is preserved as an **option**, not an instruction to force host-resident static analysis behind a guest runtime. No `TOOLxxx` or `PLATxxx` is required for this bounded adapter; `LM012-C1` is the product implementation slice; `CLIxxx` only if genuinely independent driver work later emerges.

### Deliberately deferred

`protos check`; multi-file and recursive project scanning; dependency/module resolution; project/config files; rule suppression; SARIF/JUnit/GitHub Actions-specific annotations; alternate structured formats; automatic edits; source formatting; arbitrary rule engines; generalized style warnings; new lint rules; guest-bundled TOOL ownership. Extending from one source to multiple is **additive** if every result retains source identity and the versioned diagnostic schema.

## GITHUB021 invariant/delta check

| Ratified authority / owner-approved invariant | D195 consistency |
| --- | --- |
| D194 proof-first lint; separately admitted two B1 rules, Warning default-on and no autofix | **PRESERVED**; no third rule or severity change |
| LM009-G1 parser-derived errors and source snapshot freshness | **PRESERVED**; one parser result, invalid fails closed |
| PLAT024 editor-neutral static core, on-demand, no execution overhead | **PRESERVED**; pure static on-demand adapter |
| D183/D184/LM011 formatting authority and D184 observable CLI outcomes | **PRESERVED**; no `format` behavior changed |
| D110/D124/D096 proof limits | **PRESERVED**; no speculative negative facts |
| Common toolchain one public driver, tool policy/mechanism boundary | **PRESERVED**; narrow host adapter without duplicated analysis |
| Explicit owner approval for new public interfaces | **SATISFIED** by `aprobado` to the preceding exact candidate B |

No language/Standard Library/specification change is approved or required. Future change of schema, exit codes, or analyzed source scope needs its own compatibility review.

## Publication and coordination gates

```text
D195_SELECTED=B_SINGLE_SOURCE_LINT
OWNER_APPROVAL=EXPLICIT_2026-10-08
PUBLIC_COMMAND=protos lint [--output text|json] [--fail-on-warning] [<file>]
JSON_SCHEMA_VERSION=1
WARNING_DEFAULT_EXIT=0
WARNING_CI_EXIT=1
PARSER_ERROR_EXIT=1
USAGE_EXIT=2
SOURCE_INPUT_EXIT=3
INTERNAL_EXIT=70
USER_SOURCE_MUTATION=NONE
GUEST_EXECUTION=NONE
NEW_LINT_RULES=NONE
NEXT=LM012-C1_IMPLEMENTATION_IN_guillermomolina/protos
```

Native D195-to-LM012/#671 parent/sub-issue linkage cannot be established with the current available GitHub connector. Its **textual parent is not proof** of the native edge. Although owner-approved and durably ratified here, keep D195 **open** and report `NATIVE_PARENT=PENDING` until that structural postcondition is verified under GITHUB015. A separate native exact issue dependency edge may be needed; do not falsely claim it exists.
