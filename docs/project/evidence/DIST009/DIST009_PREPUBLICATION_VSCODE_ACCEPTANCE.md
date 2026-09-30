# DIST009 — prepublication VS Code acceptance checkpoint

Status: **FUNCTIONAL PREPUBLICATION PROOF / RELEASE STILL PENDING**

This durable, non-normative record retains the editor-integration checkpoint that
preceded DIST009 publication work. It records what was actually demonstrated
against a BUG012-fixed Native runtime and what remains unproven until an exact
release candidate is built.

## Ownership

```text
DATE=2026-09-30
WORK_ITEM=DIST009
GITHUB_ISSUE=guillermomolina/protos#743
CONSUMER=DIST006-B / guillermomolina/protos#734
BUG012=guillermomolina/protos#742
BUG012_REPAIR_REVISION=417e44a8c60eaba1db7859d78bbb1e61c5dace64
```

DIST009 owns publication of the next current `JVM_PLUS_NATIVE` prerelease that
contains the BUG012 repair. It is not a historical BUG012-only patch release.
The release baseline is the current intended Protos release when DIST009 starts;
once a concrete candidate enters validation, that candidate is frozen for the
rest of the release attempt.

At this checkpoint, Protos `main` had already advanced beyond BUG012 through
I062, I073, I074 and I075-D. The observed repository state was:

```text
PROTOS_MAIN=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
PROTOS_MAIN_VERSION=0.3.125-SNAPSHOT
```

This is historical checkpoint identity, not an advance selection of the eventual
DIST009 release baseline.

## Why the first VS Code retries were misleading

The extension acceptance driver originally resolved its runtime through
`install_locked_runtime()`, which installs the runtime named by
`protos-source.lock.json`. That lock still targets public `v0.3.116`, whose
Native artifact predates BUG012.

Therefore setting only:

```text
PATH=/tmp/protos-bug012-runtime/bin:$PATH
```

did not change the runtime used by `make -C test`. Real Debug was repeatedly
exercising the old published Native runtime and reproduced the original BUG012
failure shape.

A bounded local acceptance override was then added:

```text
PROTOS_ACCEPTANCE_RUNTIME=/tmp/protos-bug012-runtime/bin/protos
```

so the real VS Code acceptance could exercise the intended unreleased
BUG012-fixed runtime directly.

The temporary runtime reported:

```text
Protos 0.3.120-SNAPSHOT
```

because it had been built before later publication metadata/version updates. It
contained the BUG012 implementation and had already passed the direct Native DAP
repair gates. It is functional prepublication evidence, not the final DIST009
release candidate.

## Direct DAP discriminators

Before the runtime-selection mismatch was found, several direct DAP probes were
used to exclude editor-specific protocol hypotheses against the same
BUG012-fixed Native executable.

The runtime passed:

```text
breakpoint line 2 -> stopped                         PASS
stackTrace line 2 -> one frame                       PASS
VS Code-style initialize capabilities                PASS
setBreakpoints/launch ordering                        PASS
two threads requests before stackTrace                PASS
concurrent VS Code-style threads requests             PASS
VS Code-style launch arguments and working directory  PASS
```

Representative result:

```text
LINE2_BREAKPOINT_STOPPED=PASS
VSCODE_THREADS_1=PASS
VSCODE_THREADS_2=PASS
LINE2_STACK_FRAMES=1
LINE2_TOP_FRAME line=2
VSCODE_ORDER_STACKTRACE=PASS
VSCODE_ORDER_DAP_PROBE=PASS
```

These probes confirmed that the repaired Native runtime itself could serve the
real line-2 debugger path required by the editor acceptance.

## Acceptance-harness corrections

Two independent harness effects were also removed before the final acceptance
result was considered reliable.

First, the harness had polled DAP with its own `threads` / `stackTrace`
requests while VS Code was already making the same requests automatically. This
created a second DAP client behavior on top of VS Code's own debugger UI flow.
The harness was changed to observe the stack response already requested by VS
Code instead of issuing a duplicate `stackTrace`.

Second, the harness kept its source breakpoint installed while testing
`next`. The debugger correctly continued and then immediately stopped again on
the same line-2 breakpoint, making the harness report that stepping had not
advanced. The harness was changed to remove the breakpoint before `next` and
wait for the corresponding `setBreakpoints([])` acknowledgement.

Neither correction changes Protos runtime or language behavior.

## Real VS Code acceptance result

With the BUG012-fixed runtime selected explicitly and the harness no longer
interfering with the debugger flow, the full local acceptance passed:

```text
LM009_I3C_REAL_DEBUG=PASS
PACKAGED_EXTENSION_FOUND=YES
PACKAGED_EXTENSION_ACTIVATED=YES
SOURCE_BREAKPOINT=PASS
STOP_LOCATION=PASS
STACK_FRAMES=PASS
ACTIVATION_LOCAL_SCOPE=PASS
REPRESENTATIVE_SCALAR_VALUE=PASS
STEP_NEXT=PASS
CONTINUE=PASS
CLEAN_TERMINATION=PASS
EXTENSION_DEVELOPMENT_PATH_USED=NO
LOCAL_REAL_DEBUG_ACCEPTANCE=PASS

LOCAL_VSCODE_ACCEPTANCE=PASS
LOCAL_VSCODE_ACCEPTANCE_PROTOS_BUILD=NO
```

The successful debugger trace also established:

```text
breakpoint verified at line 2
stopped(reason=breakpoint, threadId=1)
stackTrace -> one frame at line 2
scopes -> Block
variables -> x=41
next -> continued -> subsequent stop
continue -> terminated
```

Real Run had already passed in the same acceptance flow.

## What this does and does not prove

Established:

```text
BUG012_FIXED_NATIVE_REAL_VSCODE_RUN=PASS
BUG012_FIXED_NATIVE_REAL_VSCODE_DEBUG=PASS
BREAKPOINT_STACK_SCOPES_VARIABLES_NEXT_CONTINUE=PASS
EDITOR_SIDE_BUG012_WORKAROUND_REQUIRED=NO
DIST009_RELEASE_PATH_TECHNICALLY_VIABLE=YES
```

Not yet established:

```text
DIST009_EXACT_RELEASE_CANDIDATE_BUILT=NO
DIST009_EXACT_RELEASE_CANDIDATE_VSCODE_GATE=NOT_YET_RUN
DIST009_PUBLICATION=NO
DIST006_B2_PUBLISHED_RUNTIME_ACCEPTANCE=NOT_YET_RUN
```

The local runtime was deliberately treated only as a functional proof. DIST009
must still build the current intended release candidate, run the ordinary
Native/JVM/multi-asset admission, run the same real VS Code Run/Debug gate
against that exact candidate before publication, and only then publish.

After publication, DIST006-B2 updates the extension lock to the new public
runtime and repeats the installed/public Run/Debug acceptance.

## Routing

```text
DIST009_STATUS=READY
NEXT_OWNER=DIST009/#743
NEXT_SLICE=DIST009-A
NEXT_SLICE_TYPE=INVESTIGATION
DIST006_B_STATUS=BLOCKED_ON_DIST009_PUBLICATION
```

The next DIST009 investigation should identify the current intended release at
the time it runs, reuse the established DIST005/DIST001 multi-asset machinery,
and avoid pinning the release to the historical BUG012 commit merely because
BUG012 triggered the publication need.
