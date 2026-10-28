# DOC012-A — HTTP/1.1 learning materials publication and closure evidence

**Owning formal work:** [DOC012 — HTTP/1.1 Server, Host and Client documentation](https://github.com/guillermomolina/protos/issues/976) (top-level `DOC012`; one implementation slice `DOC012-A`).

**Record date:** 2026-10-28.

**Role:** durable, non-normative documentation-publication and validation evidence. This record does not define HTTP or Protos semantics and does not own live Issue status, priority, or assignment.

## Exact published product identity

- **Repository:** `guillermomolina/protos`
- **PROTOS_REVISION:** [`65d0e76303d09229f036e99ed5f29fe2c5367958`](https://github.com/guillermomolina/protos/commit/65d0e76303d09229f036e99ed5f29fe2c5367958)
- **Commit subject:** `DOC012-A: add HTTP/1.1 Standard Library guide, tutorials and examples`
- **Publication:** maintainer reports commit pushed; GitHub independently exposes the exact commit on `main` with the 16 paths below.
- **Specification changed:** no. There are no changes to `spec/`, the HTTP Standard Library implementation under `protos/lib/`, runtime Java, `pom.xml`, or `CHANGELOG.md`.

The product commit is the authority for the delivered content. The HTTP normative owner remains `spec/io/HTTP.md` (Server/D215, Host/D216, Client/D218-B), with `NETWORK.md`, `BYTE_IO.md`, `IO_CORE.md`, and Futures/Tasks as relevant supporting contracts.

## Published deliverables

1. **HTTP guide:** `docs/guide/library/http.md` describes when to use `std:http/Server`, `std:http/Host`, or `std:http/Client`; explicit authority and module imports; Bytes-oriented Request/Response values; borrowed and owning lifecycles; body callback boundaries; Host Workers and shutdown; limits, errors, and unsupported facilities. The guide is indexed from the maintained guide and library navigation.
2. **Progressive lessons:** `protos/tutorials/13-http/` provides one-response serving, incremental Client response-body consumption, and TCP Host+Client over loopback, together with supporting in-memory duplex and handler modules. The separate `memory-duplex-contract.protos` asserts the teaching duplex's byte-I/O and close contracts.
3. **Cookbook examples:** `protos/examples/io/` provides a minimal HTTP response, self-contained loopback GET, bounded loopback hosting, and the Worker-local echo handler.
4. **Execution coverage:** `ProtosHttpLearningMaterialIntegrationTest.java` evaluates the three TCP-dependent learning programs unchanged under Java Polyglot embedding with socket authorization; thread creation is authorized only for programs spawning Host Workers. The print-discovered CLI teaching programs remain separate from Network-dependent programs.
5. **Navigation:** the guide/library, tutorials, and examples README indexes point to the new material and identify environment requirements.

Exact changed paths in the published commit:

```text
docs/guide/README.md
docs/guide/library/README.md
docs/guide/library/http.md
protos/examples/README.md
protos/examples/io/http-echo-handler.protos
protos/examples/io/http-get.protos
protos/examples/io/http-loopback-hosting.protos
protos/examples/io/http-minimal-response.protos
protos/tutorials/13-http/01-serve-one-response.protos
protos/tutorials/13-http/02-client-response-body.protos
protos/tutorials/13-http/03-host-and-client-over-tcp.protos
protos/tutorials/13-http/hello-handler.protos
protos/tutorials/13-http/memory-duplex-contract.protos
protos/tutorials/13-http/memory-duplex.protos
protos/tutorials/README.md
src/test/java/com/guillermomolina/protos/execution/integration/http/ProtosHttpLearningMaterialIntegrationTest.java
```

## Validation provenance

The maintainer explicitly reported the following **local validation results on the published DOC012-A candidate** after the final cleanup repair:

| Gate | Result | Provenance |
| --- | --- | --- |
| `git diff --check` | PASS (clean) | Maintainer-reported |
| `make test` | PASS | Maintainer-reported |
| `make check` | PASS | Maintainer-reported |
| Product commit pushed | YES | Maintainer-reported; exact SHA and 16-file commit independently observed on GitHub |

The narrow `ProtosHttpLearningMaterialIntegrationTest` / `ProtosCliLearningMaterialsTest` set was reported PASS **on an earlier candidate** before subsequent repairs; that earlier result is not presented as a separate focused-test PASS for the final candidate. The final full-suite results above are the reported final validation.

No independent agent-executed build, program, test run, or local diff-check is claimed. No remote CI PASS is inferred from the local results.

## Scope and retained limitations

- The standalone CLI does not grant the `network` capability. Real TCP learning sources require an authorized Java Polyglot embedding; the in-memory-duplex lessons can run in the CLI.
- HTTP/1.1 examples do not add URL parsing, DNS, redirect handling, pooling, keep-alive, HTTPS/TLS, or implicit timeouts. The distinct TLS workstream is outside DOC012.
- The TCP GET example's connect, exchange, origin, and listener-close **failure paths do not have dedicated deterministic fault-injection tests**. Its successful execution path is covered by the published embedding fixture; cleanup semantics were reviewed statically against cancellation, listener close, and `ensure` rules.
- A 413 example over real TCP was deliberately omitted because a client write racing an early server close could make the demonstration nondeterministic. The guide explains the limit behavior.
- The new guest source files and Java test carry the reported complete APL Part 5 notice. Historical unrelated abbreviated notices were not changed.

## Closure interpretation

DOC012-A delivered the one combined documentation slice in the owning Issue. There is no independent DOC012-B phase, API redesign, specification change, release, or website publication implicitly included in this closure.

Under the DOCxxx first-publication record rule and GITHUB020, this owner-specific record is required and is published **after** the exact product revision. The final Issue comment must cite both `PROTOS_REVISION` and this project's exact record-publication revision (`PROJECT_RECORD_REVISION`). Only after those cross-references are verified may the coordinating agent close [DOC012 #976](https://github.com/guillermomolina/protos/issues/976) as completed.

## AI-assistance disclosure

This record was materially prepared with ChatGPT assistance using the published product commit, the owning Issue, repository governance, the implementation-agent handoff, and maintainer-reported validation. It does not claim independent human review or agent-executed tests.
