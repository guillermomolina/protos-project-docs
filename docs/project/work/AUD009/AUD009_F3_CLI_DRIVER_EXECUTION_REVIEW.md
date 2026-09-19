# AUD009-F3 — Public CLI driver and direct execution architecture review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#655`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`ecf563ed01275929d5b85330e8e6259cc85d73d8`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#655`, issue comment
`5740092781`.

Owner approval provenance: `guillermomolina/protos#655`, issue comment
`5740095940`, 2026-09-19.

Derived implementation routes: **NONE**

## Explicit scope boundary

By direct project-owner instruction, the Test Tool and its active rework were not
audited by F3.

```text
TEST_TOOL_AUDITED=NO
TEST_TOOL_EXCLUDED_BY_OWNER=YES
```

F3 therefore makes no statement about TOOL002 internals, TOOL005, AUD014,
D152/D153 or successor Test Tool work.

Package Tool internals were already audited by F1. Editor/LSP/debugger internals
were already audited by F2. F3 considered only their public dispatch coexistence
under the single `protos` driver.

## Final classification

```text
PUBLIC_PROTOS_DRIVER=KEEP
HOST_TOP_LEVEL_DISPATCH=KEEP
HELP_VERSION=KEEP

PERSISTENT_REPL=KEEP
JLINE_REPL_TERMINAL=KEEP
MULTILINE_REPL=KEEP

ONE_SHOT_EVAL_E=KEEP
DIRECT_FILE_EXECUTION=KEEP
DIRECT_FILE_LOCAL_RESOLUTION=KEEP

WORKSPACE_RUN=KEEP
DIRECT_AND_PACKAGE_RESOLUTION_SEPARATION=KEEP

CLI_INITIAL_CONTEXT_PRINT=KEEP
DIAGNOSTIC_INSPECTOR=KEEP
CLI008_C_GUEST_STACK_DIRECTION=KEEP

APPLICATION_ARGUMENT_PROJECTION=KEEP
PROCESS_STANDARD_STREAM_PROVISIONING=KEEP

EXACT_BUNDLED_TOOL_DISPATCH=KEEP
DEBUG_DRIVER_ROUTE=KEEP
LANGUAGE_SERVER_DRIVER_ROUTE=KEEP

OUTER_DRIVER_GUEST_COMMANDLINE_PARSER=ABSENT_RETAIN_ABSENCE
CWD_SEARCH_PATH_RESOLUTION_HEURISTICS=ABSENT_RETAIN_ABSENCE
ERROR_OBJECT_STACK_MUTATION=ABSENT_RETAIN_ABSENCE
ALWAYS_ON_STACK_CAPTURE=ABSENT_RETAIN_ABSENCE
```

No scoped F3 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## One public protos driver remains

One installed `protos` executable continues to own the outer bootstrap/dispatch
boundary for:

- direct source execution;
- no-argument REPL;
- `-e`;
- package-backed `run`;
- exact bundled Tool launch;
- `debug`;
- `language-server`;
- help/version.

This is intentionally small host-side bootstrap logic. It must select what kind
of environment to start before ordinary guest/library policy can run.

Using `std:cli/CommandLine` for the outer dispatch would invert this bootstrap
boundary by requiring guest execution to decide whether guest execution, a Tool,
a debugger host or the static language server should start.

Separate public executables would add distribution/PATH/version-selection surface
without removing the underlying mechanisms.

Classification: **KEEP**.

## Persistent REPL and terminal UX remain

The REPL retains one top-level context across evaluations and therefore provides
a capability distinct from one-shot `-e`.

Current maintained behavior includes:

- persistent bindings/state;
- recovery after input errors;
- result display;
- line editing/history;
- bracketed paste;
- Ctrl-C/Ctrl-D terminal behavior;
- parser-backed multiline accumulation.

JLine is a real runtime dependency and carries ongoing cost, but the dependency
is isolated to the deliberately retained interactive terminal experience.

Classification: **KEEP**.

## Multiline input remains parser-owned

The CLI does not maintain a second grammar to decide input completeness.

It accumulates input only when the canonical parser reports unexpected
end-of-source and executes once the same parser recognizes a complete unit.

This preserves ordinary multiline Protos source in the REPL without editor-like
syntax duplication.

Classification: **KEEP**.

## -e remains a separate locationless one-shot surface

`protos -e <source>` remains useful for scripting, smoke checks and automation.

It is intentionally:

- one-shot;
- locationless;
- non-persistent;
- without direct local `./` / `../` source resolution;
- able to receive explicit application arguments.

Its implementation reuses the standalone Process/session/root-task machinery, so
its incremental maintenance cost is small.

Classification: **KEEP**.

## Direct file execution remains

D137/CLI009 established a canonical direct-file domain with:

- physical source identity;
- canonical initial ModuleKey;
- explicit importer-relative `./` / `../` local imports;
- selected entry-directory confinement;
- no CWD fallback;
- no implicit extension;
- no directory-index lookup;
- no search path;
- fail-closed imported symlink/reparse traversal;
- exact component spelling.

This remains the lightweight package-free composition path for examples, scripts
and standalone source trees.

Classification: **KEEP**.

## Package-backed run remains distinct

`protos run <entry>` consumes package/project authority rather than direct-file
authority:

```text
exact project root
+ package metadata
+ canonical non-stale lock
+ package-backed resolver
+ explicit root-package logical entry
```

Direct-file and package-backed execution are therefore complementary rather than
duplicate CLI spellings.

Merging them behind path/CWD/search fallbacks would weaken identity and authority
boundaries.

Classification: **KEEP**.

## CLI-local print convenience remains

The initial CLI context may expose `print` as a convenience without making it
Core syntax, a Core binding or an ambient host-stream escape.

The facility writes through Process stdout plus its selected Encoding and
preserves current suspension/Future behavior.

Removing it would impose low-level TextWriter boilerplate on the primary
learning/interactive path for little architectural benefit.

Classification: **KEEP**.

## Diagnostic inspection remains distinct from print and serialization

D063/CLI008-B already fixed:

```text
inspect != print != serialize
```

The diagnostic inspector remains bounded, cycle-aware, deterministic and
non-evaluating. It enriches REPL/error presentation without changing ordinary
program output or serialization semantics.

Classification: **KEEP**.

## CLI008-C direction remains

CLI008-C / #416 remains valid active work.

The retained direction is:

- guest-only stack/source presentation;
- no JVM/Truffle/scheduler-frame leakage;
- no Error-object mutation;
- no fabricated async causal relation;
- no always-on stack cost on successful execution.

F3 does not select its implementation mechanism. CLI008-C remains responsible
for stopping at a PLATxxx gate if its implementation re-audit exposes a new
durable platform choice.

Classification: **KEEP direction**.

## Application arguments and standard streams remain explicit

The CLI continues to project only application arguments into
`process.args()`. Launcher spelling, physical source name, `run`, and the
logical entry name are not falsely represented as guest application arguments.

stdin/stdout/stderr are provisioned through the Process capability model with
host-selected Encoding associations rather than giving guest code ambient Java
stream authority.

Classification: **KEEP**.

## Bundled Tool dispatch remains narrow

The public driver may select an exact bundled Tool and provision approved
mechanisms/capabilities, while higher-level Tool policy remains owned by ordinary
bundled Protos code.

F3 does not move Package Tool policy into Java and makes no classification of
Test Tool internals.

Classification: **KEEP dispatch boundary**.

## Debug and language-server routes remain under the same executable

F2 already classified their internal architectures KEEP.

F3 only confirms that exposing them through the same selected external
`protos` launcher remains a coherent distribution/toolchain boundary.

Classification: **KEEP**.

## Implementation-shape note

`ProtosCli.java` is physically large, but source-file size is not itself a
semantic or architectural institution.

Much of its size is integration/bootstrap plumbing for separately owned
mechanisms. Internal extraction/refactoring may be useful later if it lowers
maintenance risk without changing behavior, but F3 opens no refactor route,
especially while Test Tool internals are being actively reworked.

## Required routing

No F3 removal/redesign route is required.

```text
REMOVAL_ROUTES=NONE
DERIVED_CLIXXX=NONE
DERIVED_IXXX=NONE
DERIVED_DXXX=NONE
DERIVED_PLATXXX=NONE
```

## AUD009-F partition closure

The intended Tooling partition is now:

```text
F1 Package Tool architecture              COMPLETE
F2 editor/LSP/debug/VS Code architecture  COMPLETE
F3 public CLI/direct execution            COMPLETE
Test Tool                                 OWNER-EXCLUDED / NOT AUDITED

AUD009_F_PARTITION=COMPLETE
```

The partition is complete only in this explicitly bounded sense. It must not be
cited as an AUD009 review of TOOL002.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=ecf563ed01275929d5b85330e8e6259cc85d73d8

PUBLIC_CLI_ARCHITECTURE=KEEP
REPL_ARCHITECTURE=KEEP
DIRECT_FILE_ARCHITECTURE=KEEP
PACKAGE_RUN_PUBLIC_ROUTE=KEEP
PRINT_DIAGNOSTIC_SEPARATION=KEEP

TEST_TOOL_AUDITED=NO
TEST_TOOL_EXCLUDED_BY_OWNER=YES

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_F3_CLASSIFICATION=COMPLETE
AUD009_F_PARTITION=COMPLETE
```

AUD009-F3 is complete.
