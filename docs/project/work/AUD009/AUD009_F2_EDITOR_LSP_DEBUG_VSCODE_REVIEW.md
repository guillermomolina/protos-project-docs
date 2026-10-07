# AUD009-F2 — Editor, static language service, debugger and VS Code architecture review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#653`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`ecf563ed01275929d5b85330e8e6259cc85d73d8`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#653`, issue comment
`5740061730`.

Owner approval provenance: `guillermomolina/protos#653`, issue comment
`5740065131`, 2026-09-19.

Derived implementation route:

```text
I054 / #654 — Retire obsolete GraalVM dynamic-LSP smoke
```

## Explicit scope boundary

By direct project-owner instruction, F2 did **not** audit the Test Tool or its
active rework.

Out of scope:

```text
TOOL002 / Test Tool
TOOL005
AUD014
D152 / D153 and successor Test Tool decisions
protos/tools/test/**
std:test/**
current Test Tool rework
```

F2 also did not classify generic top-level CLI-driver breadth. Only the exact
launcher commands consumed by the editor product were in scope.

## Final classification

```text
STATIC_LANGUAGE_SERVER=KEEP
LSP_STDIO=KEEP
EDITOR_NEUTRAL_STATIC_ANALYSIS=KEEP
REAL_PARSER_REUSE=KEEP

PROJECT_BINDING=KEEP
PER_PROJECT_INDEX=KEEP
OPEN_DOCUMENT_OVERLAY=KEEP
PROTOS_PROJECT_ARTIFACT=KEEP

PROOF_FIRST_DEFINITION=KEEP
PROOF_FIRST_REFERENCES=KEEP

THIN_VSCODE_EXTENSION=KEEP
EXTERNAL_PROTOS_EXECUTABLE=KEEP
RUN_CURRENT_FILE=KEEP

REAL_GRAALVM_DAP=KEEP
PROTOS_DEBUG_LAUNCHER=KEEP
DEBUG_READY_HANDSHAKE=KEEP

TEXTMATE_GRAMMAR=KEEP
STANDALONE_EXTENSION_REPOSITORY=KEEP
PROTOS_SOURCE_REVISION_PIN=KEEP

REPRODUCIBLE_CANONICAL_VSIX=KEEP
GITHUB_RELEASE_DISTRIBUTION=KEEP

GRAAL_DYNAMIC_LSP_TEST_DEPENDENCY=REMOVE_NOW_RECONSIDER_LATER
I026_G_DYNAMIC_LSP_SMOKE=REMOVE_NOW_RECONSIDER_LATER

HOVER=ABSENT_RETAIN_ABSENCE_UNTIL_LM010
COMPLETION=ABSENT_RETAIN_ABSENCE_UNTIL_LM010
SIGNATURE_HELP=ABSENT_RETAIN_ABSENCE_UNTIL_LM010
```

No scoped F2 mechanism is classified `REMOVE_PERMANENTLY`.

## Dedicated static language service

The selected static tooling path remains:

```text
thin editor client
    -> external protos language-server
        -> standard LSP over stdio
            -> editor-neutral Protos static-analysis core
                -> real Protos parser/source model
```

The static-analysis core directly reuses the real parser. It does not duplicate
Protos parsing or language semantics in the editor.

The dedicated language-server process remains independent from guest execution
and live Protos Process/Actor/Task state.

Classification: **KEEP**.

## ProjectBinding, project indexes and live overlays

Canonical ProjectBinding remains the sole project/source authority for
project-wide editor intelligence.

One language-server session may own independent exact project bindings and
separate incremental project indexes. Open-document snapshots override only
their exact canonical source identity.

An editor workspace folder, nearest manifest, URI prefix or recursive
`*.protos` search does not become Protos project identity.

Classification: **KEEP**.

## Package-owned protos.project boundary

The repository-carried `protos.project` projection remains Package Tool owned.

The language-server-side provider mechanically validates the generated
projection and versioned metadata/content witnesses, but does not independently
interpret package manifest/lock semantics.

This preserves:

- no Package Tool guest execution during ordinary static requests;
- no second manifest/lock/package semantic owner;
- exact candidate-root acquisition;
- fail-closed stale/missing/invalid binding;
- a replaceable provider boundary for future BSP/package-daemon/remote
  architectures.

Classification: **KEEP**.

## Static definition and references proof boundaries

Dynamic Protos lookup makes source spelling or workspace-symbol name
insufficient semantic identity.

The current definition/references architecture remains proof-first and fails
closed where exact static evidence is unavailable. Workspace-symbol search
remains exploratory name search rather than a semantic binding authority.

Classification: **KEEP**.

## Thin VS Code product boundary

The standalone extension consumes one configured external launcher:

```text
protos.runtime.executable
    -> Run Current File
    -> protos debug
    -> protos language-server
```

The VSIX does not carry a second Protos runtime or language-server distribution.

That keeps editor and command-line users on the same selected Protos toolchain
while allowing independent extension product/version/release evolution.

Classification: **KEEP**.

## Real DAP debugger path

Real GraalVM DAP remains production tooling.

The current boundary:

```text
VS Code
    -> protos debug
        -> real GraalVM DAP on bounded loopback endpoint
        -> versioned PROTOS_DEBUG_READY handshake
        -> VS Code DebugAdapterServer connection
```

preserves runtime authority for breakpoints, stepping, stack frames, scopes and
values while keeping the extension a thin startup/protocol integration layer.

Static LSP does not replace live debugging.

Classification: **KEEP**.

## TextMate grammar and standalone extension repository

The non-normative TextMate grammar remains owned by
`guillermomolina/protos-vscode-extension` under D102 while Protos grammar
semantics remain owned by the canonical specification.

The standalone extension repository remains the correct owner of extension
source, package/VSIX mechanics, releases and extension-local defects.

The exact `protos-source.lock.json` revision pin preserves cross-repository
source provenance without creating a second Protos semantic authority.

Classification: **KEEP**.

## Reproducible VSIX and distribution

The D127/D128 packaging boundary remains:

- committed dependency lock;
- clean deterministic build;
- bundled production JavaScript client;
- bounded VSIX contents;
- external Protos runtime/language-server;
- canonical byte-reproducible VSIX identity;
- GitHub Release as canonical public distribution record.

Marketplace/Open VSX remain optional future mirrors rather than closure
requirements.

Classification: **KEEP**.

## GraalVM dynamic-LSP smoke — approved removal

I026-G was valuable as a historical evidence gate.

It established that:

- the real GraalVM dynamic LSP transport worked;
- real Protos source synchronization worked;
- instrumentation-derived coverage could be observed;
- generic dynamic completion/hover/signature and related intelligence were
  insufficient/empty for the required static IDE experience.

That evidence materially supported PLAT024.

After PLAT024 and LM009, however, the retained mechanism is only:

```text
test-scope org.graalvm.polyglot:lsp dependency
+ experimental-options test setup
+ loopback socket/raw JSON-RPC smoke
```

No production runtime, static language server or VS Code extension path consumes
that generic Graal LSP.

Its ongoing maintenance therefore no longer protects selected product behavior.

Approved classification:

```text
GRAAL_DYNAMIC_LSP_TEST_DEPENDENCY=REMOVE_NOW_RECONSIDER_LATER
I026_G_DYNAMIC_LSP_SMOKE=REMOVE_NOW_RECONSIDER_LATER
```

The removal preserves all historical I026-G evidence and leaves untouched:

```text
Truffle Source / SourceSection
StatementTag / CallTag
real GraalVM DAP
debugger scopes/values
protos debug
ProtosLanguageServer
LSP4J
ProjectBinding
LM009
LM010
```

If future approved tooling genuinely needs runtime-derived dynamic-LSP
augmentation, the capability may be reconsidered against then-current GraalVM
tooling.

Implementation is routed to **I054 / #654**.

## LM010 deferred capabilities

```text
hover
completion
signature help
```

remain deferred under LM010 / #493, not rejected.

The existing editor-neutral analysis -> LSP -> thin client layering remains the
intended growth seam. F2 does not pre-approve future semantic behavior.

## Required routing

Exactly one derived implementation route exists:

```text
I054 / #654
    remove test-scope org.graalvm.polyglot:lsp dependency
    remove ProtosI026GLspTransportTest
    remove only test wiring/comments made dead by that exact deletion
```

I054 must preserve production DAP, Truffle instrumentation and the dedicated
static Protos language server.

No companion VS Code repository implementation change is required by F2.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=ecf563ed01275929d5b85330e8e6259cc85d73d8

STATIC_LSP_ARCHITECTURE=KEEP
PROJECT_BINDING_ARCHITECTURE=KEEP
DAP_ARCHITECTURE=KEEP
THIN_EXTENSION_ARCHITECTURE=KEEP
REPRODUCIBLE_VSIX_ARCHITECTURE=KEEP

GRAAL_DYNAMIC_LSP_TEST_OBLIGATION=REMOVE_NOW_RECONSIDER_LATER
DERIVED_IMPLEMENTATION=I054/#654

TEST_TOOL_AUDITED=NO
TEST_TOOL_EXCLUDED_BY_OWNER=YES

SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_F2_CLASSIFICATION=COMPLETE
```

AUD009-F2 is complete after verified routing of I054.
