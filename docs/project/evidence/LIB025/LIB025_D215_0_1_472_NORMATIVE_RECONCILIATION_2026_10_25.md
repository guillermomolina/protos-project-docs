# LIB025 / D215 — HTTP best-effort error-response precedence published in spec 0.1.472

Date: 2026-10-25. Revision-pinned, non-normative follow-up to the [LIB025 0.3.404 publication checkpoint](LIB025_404_SERVER_PUBLICATION_AND_CLOSURE_GATE_2026_10_25.md). This document records that the previously pending normative wording reconciliation has now been published. It is **not** a replacement for the specification or proof of an independently executed test gate.

## Exact revision chain

```text
WORK_ITEM=LIB025
NORMATIVE_DECISION=D215
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_REVISION=01535c8609831bcca50da481583190b657242eda
NORMATIVE_CORRECTION_REVISION=205420a03d4ebb1c5c125ca891c04147bd0c752c
GLOBAL_SPEC_REVISION=0.1.472
IMPLEMENTATION_VERSION=0.3.404-SNAPSHOT
NORMATIVE_CHANGED_PATH_COUNT=2
D215_DESIGN_ISSUE=guillermomolina/protos#961 (CLOSED)
LIB025_IMPLEMENTATION_ISSUE=guillermomolina/protos#962 (REVIEW)
INTEGRATED_MAKE_TEST_GATE=AWAITING_EXPLICIT_MAINTAINER_CONFIRMATION
```

GitHub independently confirms the [normative commit `205420a0`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c) on `main`, with exactly two changed files: `spec/io/HTTP.md` and `spec/PROTOS_SPEC_CHANGELOG.md`. The global changelog adds `0.1.472`. No runtime, guest/Java test, `pom.xml`, or implementation `CHANGELOG.md` file changes in this correction. It follows the public server [implementation `01535c86`](https://github.com/guillermomolina/protos/commit/01535c8609831bcca50da481583190b657242eda).

## Published owner-ratified precedence

| Service outcome / attempted final error write | Precedence if that best-effort write fails |
| --- | --- |
| Initial request rejection (400/413/414/417/431/501/505) | The write I/O Error supersedes the initial rejection `HttpError`. |
| Handler failure or `invalidResponse`; optional empty 500 attempted | The write I/O Error supersedes the original handler / `invalidResponse` error. If no 500 attempt occurs, the original error remains. |
| Late body-framer failure (§6.2); best-effort 400/413 attempted | Original framer `HttpError` remains, even if the write fails. |
| Owning `close()` in `ensure` fails | General cleanup-error precedence remains authoritative and may replace the preceding service outcome. |

§7.6 now explicitly forbids another final response after initiating a final-response write, including a failed one, without assuming zero emitted octets. The guard does not change physical commitment in §7.7 or `BYTE_IO.md` §6.2. The §12.2 general duplex-I/O rule retains only the narrow late-framer exception. §§6.2, 6.3, 7.7, 9, 10 and 12.1 remain unchanged.

These normative edits implement an explicit owner decision already adopted by the 0.3.404 runtime. No new public API, error kind, Task machinery, transport privilege or HTTP feature is introduced. The [D215 design issue #961](https://github.com/guillermomolina/protos/issues/961) remains closed; no new design decision is implied.

## Remaining LIB025 close gate

- The maintainer previously reported all local tests passing and a clean `git diff --check` for the 0.3.404 implementation before publication; these claims are recorded in the earlier evidence file.
- The 0.1.472 correction is a normative-document-only commit. This documentation agent has not run tests or builds, and the user's publication message does not state an exact `make test` invocation.
- [LIB025 #962](https://github.com/guillermomolina/protos/issues/962) expressly requires one human-executed canonical `make test` integrated gate. **The exact gate remains unconfirmed in the available handoff**, so the issue stays open in Review pending that confirmation. No redundant rerun is requested here.

## Source references

- [Normative publication `205420a0`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c)
- [Implementation publication `01535c86`](https://github.com/guillermomolina/protos/commit/01535c8609831bcca50da481583190b657242eda)
- [Original D215 decision publication `9dc4e508`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450)
- [Prior 0.3.404 implementation checkpoint](LIB025_404_SERVER_PUBLICATION_AND_CLOSURE_GATE_2026_10_25.md)
