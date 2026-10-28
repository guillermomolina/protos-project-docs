# D218-B — HTTP/1.1 explicit-duplex Client normative ratification (spec 0.1.476)

**Recorded:** 2026-10-28. **Role:** exact-revision normative-publication and decision-closure evidence; the normative contract itself lives in `guillermomolina/protos/spec/`.
**Decision issue:** [D218 #969](https://github.com/guillermomolina/protos/issues/969). **Implementation:** [I127 #970](https://github.com/guillermomolina/protos/issues/970).
**Owner approval:** [exact Candidate B approval](https://github.com/guillermomolina/protos/issues/969#issuecomment-6460926536), grounded in [comparative research §5](D218_EXPLICIT_DUPLEX_HTTP_CLIENT_COMPARATIVE_RESEARCH_2026_10_28.md).
**Exact normative/product revision:** [`guillermomolina/protos@6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba), global specification `0.1.476`, product `0.3.410-SNAPSHOT`.
**Decision record:** [D218 explicit-duplex HTTP Client](../../decisions/language/D218_HTTP_1_1_EXPLICIT_DUPLEX_CLIENT.md).

## Ratification and publication chain

1. The owner expressly approved D218 Candidate B for a single HTTP/1.1 exchange over an explicit caller-provided duplex, with incremental response consumption scoped to a callback; approval provenance is the linked D218 Issue comment.
2. The already-published D218 decision record compared its exact semantics against approved D215 Server, D216 Host, D217 TcpAcceptor, explicit Network authority, Core byte I/O/Future/Task/Actor semantics and Protos-first library placement. No conflicting prior invariant was reopened (**GITHUB021 invariant/delta consistency: PASS**).
3. I127 integrated the approved contract into normative `spec/io/HTTP.md` §16 and into `spec/PROTOS_SPEC_CHANGELOG.md` entry **0.1.476**. `spec/io/NETWORK.md` and `spec/semantics/ERRORS.md` register the independent optional Client and `ClientError` without adding Network authority or changing D215/D216/D217.
4. The same [product commit](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba) publishes the operational Protos-first Client, its five guest test suites, Java real-TCP integration tests, and the necessary private module registration. There is no Java HTTP implementation.
5. The Human Executor reports clean `git diff --check`, **`make test` PASS**, **`make check` PASS**, and commit/push to `main`. GitHub confirms `main` resolves to the exact product SHA above at this evidence checkpoint. The coordinating agent did not run local tests or infer success from source inspection.

## Selected boundary preserved

- Independent `std:http/Client` exports `Client` and `ClientError`; `exchangeOne` borrows the supplied byte duplex, while `.owning` closes it once after admitted terminal paths.
- Admission, request snapshots, explicit Host and forbidden caller framing headers; no implicit URL/network acquisition.
- One cooperative service Task, bounded incremental HTTP/1.1 head and body parsing, informational and final response distinction, strict CL/TE conflict handling, callback-local `ByteReadable`, retired-body lifecycle and Future-result adoption.
- Close-delimited EOF handling, chunked/trailer budgets, no-body statuses, response/body limit errors, ordinary I/O and cancellation identity and mandatory cleanup precedence.
- No host HTTP engine, ambient Network, DNS/TLS/redirect/pool/retry/HTTP2–3 semantics, reusable connection or cross-Actor transfer.

The final source/spec clarification permits outer Host SP/HTAB while keeping exact output bytes, treats syntactically valid parameter BWS in `Transfer-Encoding` as unsupported when outside the strict `chunked` subset, and retains agreed HEAD/304 representation-length and 1xx/204/205 framing checks. No new independent decision identifier or expansion beyond D218-B is established.

## Closure postconditions

| Requirement | Outcome |
| --- | --- |
| Exact owner approval and auditable provenance | PASS |
| Applicable prior invariant preservation | PASS |
| Normative Client contract published at a fixed global specification revision | PASS — `0.1.476` at `6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba` |
| Operational Protos-first client, tests and release metadata in one product publication | PASS — `0.3.410-SNAPSHOT` |
| Human integrated validation and static diff confirmation | PASS — owner reported |
| Durable revision-bound ratification/evidence | This record, committed in `guillermomolina/protos-project-docs` |
| Live Issue D218 closure | Final coordination action after re-reading the published record |

This completes the documentation prerequisites for D218 completed closure without treating project records as normative authority. The exact documentation revision is identified in the closing D218/I127 Issue comments, where cross-repository links and `CROSS_REFERENCES=PASS` are verified.
