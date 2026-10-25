# D215 — normative HTTP/1.1 server publication evidence (2026-10-25)

**Formal owner:** [D215/#961](https://github.com/guillermomolina/protos/issues/961).
**Non-normative decision:** [D215 HTTP/1.1 single-request server](../../decisions/language/D215_HTTP_1_1_SINGLE_REQUEST_SERVER.md).
**Product publication:** [`9dc4e50875252508e2afecf91541fb50f756d450`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450), **global specification `0.1.471`**.

## Remote-verified product facts

- Repository: `guillermomolina/protos`; branch: `main`.
- Exact commit: `9dc4e50875252508e2afecf91541fb50f756d450`.
- Commit subject: `D215: publish Standard Library HTTP/1.1 single-request server normative contract (spec 0.1.471)`.
- Product changed paths (**exactly seven**):
  1. `spec/io/HTTP.md` — new 1,007-line normative HTTP Standard Library owner;
  2. `spec/concurrency/FUTURES_AND_TASKS.md` — Actor-incarnation root asynchronous scope, Task-backed ownership, continuation routing and abnormal owner/child termination;
  3. `spec/concurrency/ACTORS.md` — no implicit Task from synchronous host RootActor entry, preserved exclusivity;
  4. `spec/io/NETWORK.md` — separate HTTP facility reference, no new Network authority;
  5. `spec/semantics/ERRORS.md` — module-local `HttpError` directly under `Error`;
  6. `spec/PROTOS_SPEC_CHANGELOG.md` — new global revision `0.1.471`, 2026-10-25;
  7. `AGENTS.md` — HTTP normative-ownership map entry, not new semantics.
- The exact commit, changed-path list, new HTTP specification and global changelog were read from published GitHub repository data rather than inferred from local status.

## Approval and invariant trail

- Option A and bounded A3-minimal / Q0 approval preceded [P6's full F3–F8 packet](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153) and explicit [P7 ratification](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385790888).
- Owner explicitly approved [F9 A–F in P9](https://github.com/guillermomolina/protos/issues/961#issuecomment-6388054314); F9-A deliberately replaced P6's synchronous-validation result with a failed Future after successful `serveOne` dispatch.
- Owner subsequently ratified **F10 A–C**: rejection/status precedence, logical first body-read framer error versus subsequent lifecycle error, and exactly one service Task.
- Owner subsequently ratified **F11 A–C**: Actor-incarnation root structured scope for Task-backed work started without current Task, abnormal owner/child cancellation and cleanup sequencing, and ownership/cancellation-before-start for owning HTTP service. The final §26 editorial correction limits scope ownership to *Task-backed* work; non-Task-backed Futures remain governed by their producer.
- Invariant/delta reconciliation retained explicit Core Network authority, byte-duplex borrowing, Actor nontransferability, RootActor exclusivity, existing Future producer cancellation semantics, and the strict trust boundary. See the [durable decision](../../decisions/language/D215_HTTP_1_1_SINGLE_REQUEST_SERVER.md) for concise matrix. No unauthorized runtime or API extension was added.

## Validation provenance and limitations

- **Maintainer-reported:** `git diff --check` clean; all local tests PASS; D215 commit pushed.
- **Independently verified here:** published product commit SHA, message, exactly seven changed paths, global spec revision and normative file presence/content.
- **Not independently established:** exact local test command, test count, captured full stdout/stderr, standalone HTTP runtime execution or deployed server. A spec-only publication does not supply the latter.
- **Not part of D215:** HTTP runtime, `pom.xml` version bump, root product `CHANGELOG.md`, listener or HTTP client, TLS, deadlines, package-system integration, HTTP/2, keep-alive.

## Closure/publication boundary

**PROTOS_REVISION=`9dc4e50875252508e2afecf91541fb50f756d450`**
**SPEC_REVISION=`0.1.471`**
**DECISION_APPROVAL_PROVENANCE=PASS** (explicit owner approval, with F9–F11 refinements)
**DECISION_INVARIANT_CONSISTENCY=PASS** (published normative owners and recorded ratification)
**DURABLE_RECORD_DECISION=REQUIRED** (D215's own Issue contract requires the separate durable decision)

The required durable-publication postcondition is satisfied only when this record and its decision companion are actually committed and readable on `guillermomolina/protos-project-docs:main`; the resulting **exact documentation commit SHA** is recorded on the closing D215 Issue comment. These non-normative records never replace `spec/`, and D215 design closure must not be mistaken for completed runtime implementation.
