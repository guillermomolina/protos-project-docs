# PLAT054-3E3-FS0 — Filesystem base provisioning failure: owner-approved A

**Date:** 2026-10-08
**Type:** INVESTIGATION + EXACT OWNER DECISION (GITHUB010/GITHUB021)
**Decision:** PLAT054 / [guillermomolina/protos#838](https://github.com/guillermomolina/protos/issues/838)
**Implementation owner:** I086 / [guillermomolina/protos#840](https://github.com/guillermomolina/protos/issues/840)
**Blocker:** B011
**Scope:** only the fail-closed outcome when effective Polyglot File I/O authorization exists but a safely representable/confined Context working-directory base cannot be established.

## Baseline and provenance

Read-only repository verification at the decision checkpoint:

- Product: [`guillermomolina/protos@f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2`](https://github.com/guillermomolina/protos/commit/f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2), `0.3.294-SNAPSHOT`, normative specification `0.1.450`.
- Prior durable project records: `guillermomolina/protos-project-docs@dd8afd5d0fcef7ffe073a05d7bfc8a6e4ff53351`; no B011 outcome had yet been selected.
- Existing normative owners: [`PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2/spec/io/PROCESS_IO.md) (Embedding filesystem grant/base), [`FILESYSTEM.md`](https://github.com/guillermomolina/protos/blob/f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2/spec/io/FILESYSTEM.md) §20/20.1; `IO_CORE.md`, `MODULES.md`, `ACTORS.md` retain their own existing rules.
- In the active conversation, immediately following the complete FS0 GITHUB010 research report and its summary of A versus B, the project owner expressly wrote: **“evidentemente apruebo A”**. This is an exact candidate choice, not generic permission to continue.
- An assistant's intervening suggestion to prefer B was not an owner approval and is superseded by this explicit owner selection.

**OWNER_APPROVAL=A_BOOTSTRAP_ABORT — PASS.** The research/owner selection is complete. **Normative publication is not**: specification `0.1.450` still states that both outcomes are open until a human-executed product change is committed and published.

## Exact selected outcome

**A — BOOTSTRAP ABORT.** If the Context effectively authorizes File I/O, but its configured provider and effective working-directory base cannot support a safely representable and confined initial Filesystem capability, the first Process bootstrap **fails before any source expression in the initial guest module executes**. The host receives an explicit embedding/bootstrap provisioning failure. No `filesystem` slot or partially initialized guest module is exposed.

- Do **not** downgrade this particular granted-but-unprovisionable case into successful bootstrap with an absent `filesystem` slot (rejected B).
- When effective File I/O authorization is **absent**, successful bootstrap with an **absent** (not `null`) `filesystem` slot remains the approved normal outcome.
- When authorization exists and the base is safe, the initial module receives the already-defined bootstrap-local `filesystem` capability, bounded by the configured provider.
- No ambient NIO root fallback, wider authority, provider bypass, or invented guest-visible Error category.
- The host-facing existing `ProtosEmbeddingException` is an appropriate mechanism: it already represents failures before guest entry. This is a recommended implementation mapping, not a new public Error semantic.
- Bootstrap failure must clean up any partially acquired host resources and retain the existing Context/Process retry distinction. A failed bootstrap leaves no established Process; a terminated established Process is not recreated.
- A virtual provider needs a safely defined **provider-relative logical base**, not a real host path. Ordinary open/access/permission failures after successful provisioning remain ordinary I/O outcomes rather than a reason to abort Process bootstrap.
- PAY AS YOU GROW: no FS work without permission; with permission, establish safety at bootstrap but do not eagerly start heavy operation machinery (workers, pollers, file descriptors, scheduler, wrapper graphs) if the guest never uses FS.

## GITHUB021 invariant check

| Existing owner-approved invariant | Result |
| --- | --- |
| One lazy Process per Context; Context construction and pre-eval scope query do not bootstrap | PRESERVED: failure considered only on initial valid host evaluation |
| Core default/override and internal Core resources carry no guest File authority | PRESERVED |
| Truffle Env authorization and configured provider bound all guest IO | PRESERVED, no NIO bypass |
| Persistent Process, ordinary initial module identity, bootstrap-only slots | PRESERVED, abort happens before guest module expressions |
| Java read-only bindings and ordinary guest mutation | PRESERVED |
| Reject unsafe concurrent RootActor entry; no universal gate | PRESERVED |
| Retained host Value captures original Closure | PRESERVED |
| PAY AS YOU GROW; no mandatory Task/RootTask/scheduler | PRESERVED for unused FS machinery; acknowledged limitation: granted-but-invalid FS causes first eval failure even for a trivial guest program |

Previously ratified HOST-FS-1/FS-2, HOST-NET-1/2, HOST-FUT-1, Filesystem §20/20.1 confinement and Actor/Process lifecycle are unchanged. The last PAY AS YOU GROW limitation is an **explicitly considered consequence of A**, not an accidental cost hidden in the design.

**DECISION_INVARIANT_CONSISTENCY=PASS**: the selected observable failure policy refines the open branch without changing an already approved invariant.

## GITHUB010 alternatives and comparison

A = granted authority + unsafe/unrepresentable base **aborts first bootstrap**. B = same condition **completes bootstrap without `filesystem`**. Both must fail closed with no privilege amplification. Score 1–5; H/M confidence = high/medium. These scores are decision aids, not measured performance or authority.

| Required dimension | A | B | Key reason |
| --- | --- | --- | --- |
| Correctness/invariants | 5 H | 4 M | A makes invalid provisioning observable; both confine authority |
| Protos philosophy | 4 H | 3 H | A follows “fail where invariant is violated”; B favors unrelated progress |
| Present proportionality | 3 H | 5 H | A can reject an FS-free guest on a granted-but-broken host |
| Incremental growth | 4 M | 4 M | Neither requires a new public FS API |
| Future-option resilience | 4 M | 4 M | Both allow new virtual/backends |
| Scalability | 4 M | 5 M | B keeps many broken-permission contexts alive |
| Conceptual simplicity | 5 H | 3 M | B conflates “not granted” and “granted but broken” |
| Portability | 4 M | 4 M | Neither requires physical CWD |
| Runtime/resource cost | 4 M | 5 M | A validates safe provisioning but can defer operation resources |
| Failure/operability | 5 H | 2 H | A surfaces misconfiguration at boundary |
| Reversibility/deferral cost | 4 M | 3 M | Silent absence could become an application compatibility dependency |
| Evidence maturity/implementation risk | 4 M | 3 M | Existing host bootstrap exception/retry boundary supports A |

**Strongest counterexample against selected A:** a host grants File I/O to 10,000 short-lived Contexts using a restrictive virtual provider whose effective CWD cannot safely be represented, while the guest code only computes `1`. A rejects those otherwise FS-independent programs; B would permit them to run. The owner explicitly selects A anyway: this is a failure to provision a **granted** bootstrap capability, not a requirement that an FS-free Context construct unused adapters.

**Strongest counterexample against B:** the host intends to supply a required configuration filesystem, but the provider/base setup is broken. B presents successful boot with a missing slot indistinguishable from deliberate no-permission mode and can hide deployment defects.

**Adversarial implementation cases:** nonexistent/illegible bases only when they actually prevent safe provisioning; virtual base with no physical path; concurrent symlink/reparse/mount redirection; restricted/partial providers; mixed FS/Network grants; Context close/cancellation and resource cleanup; distinct Contexts with different providers; retry only after failed bootstrap and never after fatal Process termination. Avoid probe-then-use races: a provider-respecting `TruffleFile` adapter alone is not proof of every stronger Protos FS operation invariant.

Reference documentation:
- [GraalVM `IOAccess`](https://www.graalvm.org/sdk/javadoc/org/graalvm/polyglot/io/IOAccess.html): host IO grant and custom filesystem.
- [Truffle `TruffleLanguage.Env`](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.Env.html): `isFileIOAllowed()`, `getCurrentWorkingDirectory()`.
- [GraalVM `FileSystem`](https://www.graalvm.org/sdk/javadoc/org/graalvm/polyglot/io/FileSystem.html): custom and virtual providers, no implied host-root access.
- [GraalVM `Context.Builder`](https://www.graalvm.org/25.0/javadoc/sdk/org/graalvm/polyglot/Context.Builder.html): `allowIO` and `currentWorkingDirectory`.
These APIs show the relevant boundaries; none dictates Protos's guest-visible A/B choice.

## Mandatory normative publication before backend

Next slice: **`PLAT054-3E3-SPEC` / TYPE=IMPLEMENTATION / REPOSITORY=`guillermomolina/protos` / HUMAN_EXECUTOR=YES**.

Patch only the narrow open paragraph under `spec/io/PROCESS_IO.md` → “Embedding filesystem base” and `spec/PROTOS_SPEC_CHANGELOG.md` (next *current-HEAD-derived* global revision). Normative substance:

> If effective file authorization is granted but a safe provider-confined Context working-directory base cannot be established, initial Process bootstrap fails before executing any initial module source expression. Report an explicit host bootstrap provisioning failure. Do not substitute a missing `filesystem` slot, create a guest capability, invent a public Error category, or widen host authority. No-authorization still means absent slot; valid base means capability provisioned. Preserve existing bootstrap cleanup/retry and Process terminality, provider-relative virtual semantics and pay-as-you-grow resource laziness.

Do not edit `FILESYSTEM.md`, `IO_CORE.md`, `MODULES.md`, `ACTORS.md` absent an actual normative contradiction. Do not change `pom.xml`, `CHANGELOG.md`, Java runtime, implementation tests, benchmarks, releases or other repositories as part of this **spec-only publication**. Check current `spec/PROTOS_SPEC_CHANGELOG.md` immediately before assigning the next revision.

Only **after** the product specification commit is actually published and verified may B011 transition **BLOCKED → READY**, and a separately bounded `PLAT054-3E3` Filesystem runtime implementation be released to the human executor. **CLOSED** additionally requires published implementation. B012 remains CLOSED. I086 remains OPEN for Filesystem and final portable/Native Image acceptance. I087/#841 and PERF032/#831 remain independent.

## Non-claims and assistance disclosure

This record captures research and the owner's exact choice, **not** a product specification amendment, test PASS, implementation or Native Image result. The decision/evidence was prepared with ChatGPT AI assistance and explicitly approved by the project owner. No builds, tests, product repository mutations or local Git commands were executed in this decision/publication step.

```text
ISSUE=I086/#840
DESIGN=PLAT054/#838
SLICE=PLAT054-3E3-FS0
TYPE=INVESTIGATION_OWNER_DECISION
PRODUCT_REVISION=f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2
NORMATIVE_REVISION=0.1.450
OWNER_APPROVAL=A_BOOTSTRAP_ABORT
DECISION_INVARIANT_CONSISTENCY=PASS
SPEC_PUBLICATION=PENDING
B011_STATUS=BLOCKED_PENDING_NORMATIVE_PUBLICATION
B012_STATUS=CLOSED
RUNTIME_IMPLEMENTED=NO_FOR_FILESYSTEM
NEXT_SLICE=PLAT054-3E3-SPEC
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
HUMAN_EXECUTOR=YES
```
