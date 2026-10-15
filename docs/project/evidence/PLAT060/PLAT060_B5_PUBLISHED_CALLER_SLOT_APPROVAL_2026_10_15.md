# PLAT060-B5 — Owner-approved published-activation caller-slot clarification

**Evidence date:** 2026-10-15.
**Authority:** [PLAT060/#895](https://github.com/guillermomolina/protos/issues/895), [I102/#896](https://github.com/guillermomolina/protos/issues/896), and the [canonical B5 decision](../../decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md).
**Product readback:** `guillermomolina/protos@7a79068642333542c0102f3698ad6196c58406d3` (moving `main` inspected; its tree `017701f191f16f59863b5fb30e03f735c29133cd` matches previously published I102-A `1f23f43aaf82a95306731180c82823ae046d560e`).
**Approval provenance:** after an explicit presentation of the exact conditional caller-slot interpretation, the project owner replied **“si apruebo”** in this conversation. This is a new, specific architectural clarification—not an inference from green tests, historical approval, or an Issue status change.

## Approved physical contract: one B5 ABI, two publication states

The ordinary positional CallTarget argument header remains exactly **five fixed slots** for every ordinary Closure entry, in both NORMAL and DEBUG:

```text
[0] invoked ProtosClosureValue, or this invocation's published ProtosActivation
[1] method receiver, or private DIRECT_CLOSURE_CALL marker
[2] methodHome, or explicit owning Task / null
[3] exact caller ProtosActivation while [0] is an unpublished Closure;
    after [0] is an already-published ProtosActivation, original caller if retained,
    otherwise null when the exact dynamic caller is unavailable
[4] exact per-invocation ReturnHome (including sanctioned markers)
[5...] supplied user arguments
```

**Entry-state rule, explicitly approved:**
1. **Compact/unpublished entry** (`arguments[0] instanceof ProtosClosureValue`): `arguments[3]` **MUST** be the exact non-null caller activation. Materialization consumes this identity to establish the callee's inherited provenance and, when it publishes the activation in slot 0, **does not rewrite slot 3**.
2. **Already-published entry** (`arguments[0] instanceof ProtosActivation`): `arguments[3]` **MAY** be `null` when the exact dynamic caller was not carried into construction. Every activation/provenance-aware consumer **MUST** read the already-published activation itself in this state and **MUST NOT** dereference or interpret a nullable slot 3 as a compact-call caller.
3. The slot remains reserved at index 3 in **both** states, including when null. It never disappears, is never moved, and is never replaced with a manufactured caller or another metadata role. The same physical layout, parameter offset 5, semantics, return-home identity, nullable Task policy, and NORMAL/DEBUG profile-independent ABI remain in force.

This is **not** a second ABI, a profile-specific ABI, or permission to invent a caller, synthesize a `ProtosActivation`, eagerly create guest Context/Task, or weaken lexical/nested-control semantics. No new `ProtosActivation` field is required. A future requirement that must recover the exact dynamic caller even from a rich-entry activation that never retained it requires its own justified authority/evidence and cannot be silently inferred from this clarification.

## GITHUB021 invariant/delta reconciliation

| Prior ratified B5 statement | Approved clarification | Impact |
|---|---|---|
| Slot 3 is "caller activation" for every header | Slot 3 is **mandatory exact caller** only in unpublished Closure state; in already-published activation state it is retained exact caller or `null` if unavailable | **Explicit owner-approved precision** to a formerly unconditional invariant |
| A rich entry uses the same five-slot header and must not invent provenance | Rich entry remains B5 with `arguments[0]` published and optional `null` in its reserved slot 3 | Preserved, no fabricated caller |
| Materializing an unpublished compact call publishes only slot 0 | Original non-null slot 3 remains intact across that publication | Preserved |
| Lazy activation, exact ReturnHome, captured lexical/context identity, optional Task, NORMAL/DEBUG identical ABI | No other layout, identity, profile or semantic changes | Preserved |
| Former MINIMAL / NORMAL-only headerless / ordinary one-element rich ABI retirement | No restoration | Preserved |

**Source evidence:** [`ProtosFrameArguments.publishedInvocation`](https://github.com/guillermomolina/protos/blob/7a79068642333542c0102f3698ad6196c58406d3/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java) creates B5 with the already-published activation in slot 0, writes receiver or marker, methodHome, exact ReturnHome and user arguments, and deliberately leaves caller slot 3 `null`. `materializeCompactActivation` reads slot 3 **before** publication and replaces only slot 0. `compactCaller`, `unmaterializedFullHeaderCaller`, and inherited-provenance fast paths are compact/unpublished-only. `ProtosActivation` owns the already-materialized effective provenance but has no universal dynamic-caller reference to reconstruct. The accepted approach avoids introducing fake caller identity.

## Evidence and execution separation

The owner also supplied a focal implementation-validation result for the uncommitted I102-B test candidate:
`mvn -Dtest=ProtosPerf025H1LazyRootActivationTest test` → **13 tests, 0 failures, 0 errors, 0 skipped**. The local delta at handoff was **one modified test file / 151 insertions**, `git diff --check` clean. The first compile attempt had an omitted test import, corrected before the successful run.

These results are **maintainer reported**, not tests run by this recording agent. They do **not** certify the entire I102 work item, a full `make test` run, benchmark stabilization, Tier-2 compilation quality, or a push of the 151 test lines. This approved clarification is a durable architectural record; the code already follows it for the inspected paths, so **no runtime or build-file mutation is required merely to reflect the approval**. I102 remains open for independent final closure conditions, and `pom.xml` and `CHANGELOG.md` must not be changed before the human's final executable validation.

## GITHUB015 / closure boundary

PLAT060 remains **CLOSED / RATIFIED** with this explicit owner-approved addendum; the previously ratified five-slot ordinary ABI remains the authority. I102 is a separate implementation issue and does not close from approval alone. Record the final exact `PROJECT_RECORD_REVISION` through readback of the published docs commit and cross-reference that revision in the owning GitHub Issues. The preexisting PLAT060 research snapshot and old approved B5 alternatives remain historical evidence, not a substitute for this explicit approved delta.
