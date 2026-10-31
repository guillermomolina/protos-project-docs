# INC002 — External-process capability and Protos-first command-library work split (2026-10-31)

> **Status: owner-directed coordination and source-evidence snapshot only.** This is non-normative historical evidence of the formal split of future work from INC002. It is **not** a selected D229 candidate, an approved public API, a ratified specification, a published implementation, or an authorization to implement shell syntax. Live work state remains in GitHub.

## Revision identity and primary sources

- **Protos source inspection baseline:** [`guillermomolina/protos@73bd9fe913421ff493146b546869f9a1f0d2b34e`](https://github.com/guillermomolina/protos/commit/73bd9fe913421ff493146b546869f9a1f0d2b34e) (`main` at the read-only review on 2026-10-31).
- **Owning incubator:** [INC002 — Protos as an interactive Unix shell and Bash alternative, #1004](https://github.com/guillermomolina/protos/issues/1004).
- **New design investigation:** [D229 — External OS process execution capability and lifecycle contract, #1008](https://github.com/guillermomolina/protos/issues/1008).
- **New library work item:** [LIB027 — Protos-first external commands and byte-pipeline composition, #1009](https://github.com/guillermomolina/protos/issues/1009).
- **Owner instruction:** create separate issues for steps **1** (external-execution capability contract) and **2** (Protos-first command/pipeline library), and retain steps **3** (shell syntactic sugar) and **4** (shell/Protos grammar/template integration) in INC002 **until 1 and 2 are resolved**. This instruction authorizes issue allocation and coordination, not a particular implementation or architecture.
- **Applicable process rules:** [`AGENTS.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/AGENTS.md), [`AGENTS.work/DESIGN.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/AGENTS.work/DESIGN.md), [`AGENTS.work/LIBRARY.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/AGENTS.work/LIBRARY.md), [`AGENTS.work/INCUBATOR.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/AGENTS.work/INCUBATOR.md), and [`AGENTS.work/COORDINATION.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/AGENTS.work/COORDINATION.md).

## Source-backed need for a separate process capability

1. [`spec/io/PROCESS_IO.md` §22](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/io/PROCESS_IO.md) defines `Process` as a Protos **execution domain**, not an operating-system process. Its explicit bootstrap grants do not confer subprocess authority. `process.environment()` is a snapshot; a mutable system-shell environment or working directory cannot be inferred from this API.
2. The same normative document, **§28**, explicitly excludes external-process creation, host-process control, PIDs, `kill`, OS signals, process groups and sessions. It anticipates potential child stdin/stdout/stderr streams through existing I/O protocols but does not define how to launch children.
3. [`spec/io/IO_CORE.md` §1](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/io/IO_CORE.md) excludes OS-process control and pipe creation/pipe-specific semantics. Byte I/O and `Future` facilities therefore provide reusable protocols, **not** an existing external-execution API.
4. [`spec/PROTOS_GRAMMAR.md` §21](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/PROTOS_GRAMMAR.md) fixes the Core operator surface. Ordinary `ls -l` must not be silently reclassified as a shell command inside unchanged Protos grammar. A future sugar frontend needs its own expressly approved syntactic boundary; that decision is retained in INC002.
5. [`docs/guide/interop/README.md`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/docs/guide/interop/README.md) distinguishes explicit host-registered foreign providers from general native host authority; importing arbitrary Java or reaching `ProcessBuilder` is not a standardized Protos subprocess facility.

This is a **document/source inspection and coordination record**, not a proof from executable conformance. It does not assert that all conceivable host-specific escape hatches or external providers are impossible.

## Formal separation and readiness gates

| Step | Formal owner | Scope | Recorded coordination state | Gate |
| --- | --- | --- | --- | --- |
| 1 | [D229 #1008](https://github.com/guillermomolina/protos/issues/1008) | Compare and select explicit external-OS-execution authority and child lifecycle/streams contract | `family:D`, `status:inbox`, open, unapproved | Exact design packet, owner selection and required durable ratification; **then** independent authorized capability implementation |
| 2 | [LIB027 #1009](https://github.com/guillermomolina/protos/issues/1009) | Optional Protos-first commands and streaming byte pipeline composition | `family:LIB`, `status:blocked`, open | Approved D229 semantics, working verified execution backend, resolution of any additional observable library contract |
| 3 | [INC002 #1004](https://github.com/guillermomolina/protos/issues/1004) | Explore `ls -l`-simple command and pipeline desugaring into ordinary Protos calls | Incubator only; no syntax chosen | Return after steps 1 and 2 |
| 4 | [INC002 #1004](https://github.com/guillermomolina/protos/issues/1004) | Compare explicit parser modes versus embedded Protos blocks/template-style grammar integration | Incubator only; no parser mode or delimiter chosen | Return after steps 1 and 2 |

A thin syntax layer and template-like delimiters are **not competing backend execution mechanisms**: one determines lowering to ordinary Protos library calls; the other determines the language/grammar boundary. Neither is an approved change.

D229 must not be equated with implementation. D229's issue allocation does not approve or expose an executable OS primitive; LIB027 is not ready merely because D229 eventually closes. Any separate host/native implementation work and its ownership will be allocated after exact D229 approval, if required.

## Authority, safety and portability constraints retained for research

- Explicit host-granted process execution, independent of Protos `process` and `Filesystem` authority; avoid ambient execution and authority amplification.
- Executables and argument **vectors**, not implicit `sh -c` string interpretation; preserve spaces, bytes and native representation checks where applicable.
- Separate child cwd/environment from the immutable Protos bootstrap snapshot and restricted filesystem/path namespace; do not promise confinement that a host child can bypass.
- Distinguish spawn failure, nonzero child exit, stream error and asynchronous cancellation/termination; preserve process lifetime and custody when Protos execution terminates.
- Streaming byte stdin/stdout/stderr, backpressure, EOF/broken pipe, ownership and failure cleanup before adding optional rich data pipelines.
- Portability requires an explicit Linux/POSIX/Windows policy; PTY, signals, job control and Bash compatibility are not silently included.
- Use Protos-first composition and reuse ordinary objects, Closures, Futures and byte I/O instead of a parallel private command runtime.

These are **investigation requirements/open choices**, not their resolutions.

## Coordination observations and limitations

- The owner directed **two independently tracked new issues**, D229 and LIB027, with references to INC002. The latter retains the unscheduled shell idea: no new issues were allocated for steps 3 or 4.
- The independently reusable design/library items are **not** INC002 native sub-issues by mere provenance. No native parent relation was created.
- LIB027's intended exact blocker is **D229 #1008**; native GitHub `blocked_by` was inspected and was empty at the initial read. The connected GitHub interface used for creation does not provide a native dependency mutation action. **Native blocker edge remains pending**, not reconciled by textual cross-reference alone.
- D229 remained **unselected** and LIB027 remained **blocked** at this record's observation. This document does not duplicate live GitHub status as continuing authority; any subsequent transition must be read from those issues.
- No owner approval of a D229 candidate, LIB027 API, parser grammar or shell syntax was supplied. No specifications or product sources were modified; no implementation, subprocess runtime tests, performance benchmarks, `make test`, or `make check` were run as part of this coordination task.

## Audit / evidence classification

```text
SOURCE_BASELINE=73bd9fe913421ff493146b546869f9a1f0d2b34e
D229_ISSUE=1008
LIB027_ISSUE=1009
INC002_ISSUE=1004
DESIGN_SELECTED=NO
DESIGN_RATIFIED=NO
EXTERNAL_EXECUTION_IMPLEMENTED=NO
COMMAND_LIBRARY_IMPLEMENTED=NO
SHELL_SUGAR_APPROVED=NO
SHELL_GRAMMAR_APPROVED=NO
NATIVE_LIB027_BLOCKED_BY_D229=PENDING
PRODUCT_REPOSITORY_MODIFIED=NO
TESTS_EXECUTED=NO
EVIDENCE_ROLE=NON_NORMATIVE_COORDINATION_SNAPSHOT
```

Do not cite this snapshot as normative semantics or as a completion/approval artifact. It exists because the owner expressly requested durable evidence for this new coordination split; no general obligation to archive every incubator discussion is implied.
