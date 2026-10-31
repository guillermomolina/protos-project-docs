# DOC013 — Protos-first documentation catch-up: completion evidence

**Owning work:** [DOC013 #1003](https://github.com/guillermomolina/protos/issues/1003), top-level documentation work in `guillermomolina/protos`.

**Record date:** 2026-10-31. **Role:** durable, non-normative documentation-publication and validation evidence. Live Issue status remains in GitHub; normative language and library semantics remain in `guillermomolina/protos/spec/`.

## Exact product publication

- **Repository:** `guillermomolina/protos`.
- **Frozen scope baseline:** [`PROTOS_BASE_REVISION=02e32411ac614ae69b974757430d8a29d291619d`](https://github.com/guillermomolina/protos/commit/02e32411ac614ae69b974757430d8a29d291619d).
- **Published revision:** [`PROTOS_REVISION=3bce21c2b6e97a03f666e71f47717393384c302b`](https://github.com/guillermomolina/protos/commit/3bce21c2b6e97a03f666e71f47717393384c302b), independently observed at `main`.
- **Commit subject:** `DOC013: complete Protos-first networking, TLS and linear algebra learning material`.
- **Published delta:** 20 paths, including three new guides, four standalone tutorials, two new examples, one corrected existing Resolver example, maintained navigation and an exact source-style exception.
- **No normative or implementation modifications:** no changes in `spec/`, `src/`, `protos/lib/`, `pom.xml`, `CHANGELOG.md`, release artifacts, or the website.

The product commit, maintained guides and executable Protos sources are the authoritative delivered artifacts; this record does not duplicate their prose or introduce new API obligations.

## Coverage reconciliation at the frozen baseline

| Current-HEAD capability | Documentation disposition | Maintained locations |
| --- | --- | --- |
| HTTP/1.1 Server, Host and single-exchange Client (DOC012/D215/D216/D218-B) | Existing guides and learning programs preserved and linked, not rewritten wholesale | `docs/guide/library/http.md`; existing `protos/tutorials/13-http/` |
| Router, UrlClient, Host Worker factory and RootActor service (LIB026/I133/D225) | HTTP module-choice clarification; new standalone routing lesson; existing application-service examples and Java embedding documented by cross-reference | `docs/guide/library/http.md`; `protos/tutorials/13-http/04-router-dispatch.protos`; `docs/guide/embedding/java.md` |
| Explicit name resolution and numeric TCP authority (I132/D223) | New practical guide; existing manual Resolver example updated with scoped cleanup | `docs/guide/library/networking.md`; `protos/examples/io/hostname-resolution.protos` |
| TLS client and direct HTTPS composition (I130/D220) | New guide: explicit trust, SAN-only identity, SNI, limits, errors, borrowing/owning lifecycle and JVM/Native limits; new manual TLS-over-TCP HTTP example | `docs/guide/library/tls.md`; `protos/examples/io/tls-client-over-tcp.protos` |
| Standard Library Vector/Matrix and numeric behavior (LIB024/D214) | New guide and three self-checking tutorials plus transformation example; covers Hermitian dot, strict shapes, Float/Complex and reduction order | `docs/guide/library/linear-algebra.md`; `protos/tutorials/14-linear-algebra/`; `protos/examples/numbers/vector-matrix.protos` |
| CLI/REPL completion, numeric rendering and Test Tool/corpus registration (CLI011/I134/TEST010) | Existing current instructions retained and made discoverable; no CLI/Tool code or API changes | `docs/guide/tools/cli.md`; `docs/guide/tools/test-tool.md`; navigation |
| Native installer, prefixes, rollback and local SNAPSHOT workflow (I131/DIST019) | Existing installer and first-use instructions retained; no release or runtime-version upgrade claim | `dist/README.md`; `docs/guide/00-try-protos.md` |
| Internal Truffle/compiler performance work | Deliberately excluded where no shipped user-visible contract changed | No public contract inferred |

## Exact changed paths

```text
README.md
docs/guide/05-values-identity-equality-and-collections.md
docs/guide/11-process-io-filesystems-and-authority.md
docs/guide/README.md
docs/guide/embedding/java.md
docs/guide/library/README.md
docs/guide/library/http.md
docs/guide/library/linear-algebra.md
docs/guide/library/networking.md
docs/guide/library/tls.md
protos/examples/README.md
protos/examples/io/hostname-resolution.protos
protos/examples/io/tls-client-over-tcp.protos
protos/examples/numbers/vector-matrix.protos
protos/tutorials/13-http/04-router-dispatch.protos
protos/tutorials/14-linear-algebra/01-vector-operations.protos
protos/tutorials/14-linear-algebra/02-matrix-operations.protos
protos/tutorials/14-linear-algebra/03-complex-and-shapes.protos
protos/tutorials/README.md
scripts/source_style_exceptions.json
```

The one exact source-style exception is `family=indexing`, `max_count=2`, confined to `protos/tutorials/14-linear-algebra/02-matrix-operations.protos`; the Matrix protocol requires two-dimensional `at(row, column)` and does not offer an equivalent bracket form. No broader exception or language syntax change was introduced.

## Validation provenance and execution boundaries

| Check | Result | Evidence provenance |
| --- | --- | --- |
| Focused tests | PASS | Maintainer reported all focused tests PASS before final publication |
| `git diff --check` | PASS (clean) | Maintainer reported clean local check |
| `make test` | PASS | Maintainer reported local integrated gate PASS |
| `make check` | PASS | Maintainer reported local integrated gate PASS |
| Differential source-style guard on published commit | **PASS** | Maintainer ran `python3 scripts/source_style_guard.py --base HEAD^ --head HEAD`; output: `SOURCE_STYLE_GUARD_SCANNED_CHANGED_FILES=17`, `SOURCE_STYLE_GUARD_EXCEPTION_ENTRIES=1`, `SOURCE_STYLE_GUARD_ALLOWED_INCREASES=1`, `SOURCE_STYLE_GUARD: PASS`. Exact exception reported `base=0 head=2`. |
| GitHub product publication | **PUBLISHED** | Exact `PROTOS_REVISION` and 20-path commit independently verified on GitHub |

These are human-executed local test/validation reports, not agent-executed test results. Raw focal logs and an external CI PASS are not independently claimed.

The four new tutorials and `protos/examples/numbers/vector-matrix.protos` are standalone-CLI learning sources and signal an Error on a failed self-check rather than merely printing `false`. The new `tls-client-over-tcp.protos` and the existing `hostname-resolution.protos` are **manual, host-dependent** examples: they need a Java embedding that explicitly provisions `network` and `resolver`, and the TLS client uses explicitly chosen trust. They reach external origins and are not run verbatim by the automated CLI learning-material discovery. Controlled-fixture HTTP/Resolver/TLS integration tests validate the protocol facilities instead.

The installed runtime, release artifacts, public site, and external certificate-store freshness were not changed or validated as part of this documentation-only publication. No specific remote-origin run of the manual examples is claimed.

## Closure

DOC013 delivered the one coherent current-HEAD documentation catch-up, preserving prior DOC011/DOC012 work. The first-publication work-record requirement for `DOCxxx` and GITHUB020 make this revision-bound record appropriate. Close [DOC013 #1003](https://github.com/guillermomolina/protos/issues/1003) only after this record is published in `guillermomolina/protos-project-docs`, its exact `PROJECT_RECORD_REVISION` is re-read, and the final Issue comment identifies both immutable revisions and the maintainer's validation evidence.

This record does not claim a new Protos release, website deployment, design approval, or separate implementation deliverable.

## AI assistance

This record was prepared with ChatGPT assistance from the authoritative published commit, the DOC013 Issue, repository policies, and maintainer-provided validation results. It does not claim independent human review or tests performed by the agent.
