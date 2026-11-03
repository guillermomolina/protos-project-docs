# DOC019 — Accessibility documentation: publication and validation evidence

**Date:** 2026-11-03. **Formal owner:** [DOC019 #1045](https://github.com/guillermomolina/protos/issues/1045). **Evidence kind:** revision-bound product publication and attributed maintainer validation, not a normative language/design decision.

## Immutable product publication

- **PROTOS_REVISION:** [`250b89bf2faf3e580980a67c2d69aa94d10e4df3`](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3).
- **PARENT_REVISION:** `b708e1ed981b2ba151a00210622b486a22a7a908`.
- **COMMIT_SUBJECT:** `DOC019: add Protos-first accessibility documentation`.
- **GITHUB_PUBLISHED_DELTA:** exactly three Markdown paths, 127 added lines, zero deleted lines, verified from the GitHub commit patch.
- **PRODUCT_SCOPE:** new `ACCESSIBILITY.md` (+112), root `README.md` community link (+2), `SUPPORT.md` accessibility section (+13).
- **UNCHANGED:** `spec/**`, runtime, Java/Protos sources and tests, `pom.xml`, and `CHANGELOG.md`. No version or normative semantic change.

## What the published documentation establishes

1. [`ACCESSIBILITY.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/ACCESSIBILITY.md) covers programming-language learning, text-driven CLI/REPL and diagnostic workflows, bundled tools, editor integration, the companion website, and community participation.
2. It records goals and reporting paths rather than claiming accessibility certification, verified WCAG compliance, guaranteed screen-reader compatibility, or dependence-derived GraalVM/Truffle/VS Code capabilities.
3. Report routing distinguishes [core Protos Issues](https://github.com/guillermomolina/protos/issues), [website Issues](https://github.com/guillermomolina/protos-website/issues), [extension Issues](https://github.com/guillermomolina/protos-vscode-extension/issues), and [Protos Discussions](https://github.com/guillermomolina/protos/discussions). Security-sensitive details remain directed to `SECURITY.md`.
4. [`README.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/README.md) and [`SUPPORT.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/SUPPORT.md) make the new statement discoverable through established community/support entry points.

## Validation evidence and attribution

- **Human reported:** final `git diff --check` clean.
- **Human reported:** local integrated `make test` PASS.
- **Human reported:** local integrated `make check` PASS.
- **Human reported:** `DOC019: add Protos-first accessibility documentation` committed and pushed.
- **Coordinator verified independently:** the exact product commit exists on the published repository, has the expected subject and parent, and its patch contains only `ACCESSIBILITY.md`, `README.md`, and `SUPPORT.md`, with the indicated additive changes.

No independent replay of the maintainer's tests or an accessibility conformance evaluation is claimed. No separate end-to-end external link-crawler output was provided; existing documentation paths were inspected for intended routing, not certified as continuously available services. No terminal transcript of the human validation was supplied beyond the maintainer's explicit PASS report.

## Durable and live cross-references

- [DOC019 maintained work record](../../work/DOC019/DOC019_ACCESSIBILITY_DOCUMENTATION_COMPLETION_2026_11_03.md).
- [DOC019 live Issue](https://github.com/guillermomolina/protos/issues/1045).
- [Exact product publication](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3).

The eventual Issue closure comment identifies the separately published, exact `PROJECT_RECORD_REVISION` and `PROTOS_REVISION` to make the cross-repository closure revision-bound.

```text
WORK_ITEM=DOC019
PROTOS_REVISION=250b89bf2faf3e580980a67c2d69aa94d10e4df3
PARENT_REVISION=b708e1ed981b2ba151a00210622b486a22a7a908
PRODUCT_PATHS=3
ADDITIONS=127
DELETIONS=0
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
PRODUCT_PUBLICATION=VERIFIED
INDEPENDENT_ACCESSIBILITY_AUDIT=NOT_PERFORMED
EXTERNAL_LINK_CRAWLER=NOT_ATTESTED
PROJECT_ROUTING=NOT_INDEPENDENTLY_VERIFIED
```
