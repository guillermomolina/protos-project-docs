# DOC019 — Protos-first accessibility documentation completion

**Date:** 2026-11-03. **Owner:** [DOC019 #1045](https://github.com/guillermomolina/protos/issues/1045). **Classification:** independent non-normative public documentation. **Product publication:** [`guillermomolina/protos@250b89bf2faf3e580980a67c2d69aa94d10e4df3`](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3).

## Scope and authority

DOC019 adds repository-root accessibility guidance for Protos as a programming language and its surrounding documentation, CLI, REPL, language tooling, editor integration, companion website, and community workflows. It documents accessibility priorities and reporting routes without treating the accessibility behavior of GraalVM, Truffle, Java, VS Code, or other dependencies as verified Protos capability.

The published guidance distinguishes canonical Protos language/docs/tooling concerns from website-specific and VS Code extension-specific ownership. It avoids unverified WCAG conformance, screen-reader compatibility, support guarantees, new contact channels, and requirements to disclose medical information. It routes vulnerability reports to the existing security policy. This is descriptive project documentation only: it does not amend normative Protos specifications, runtime/Standard Library behavior, implementation, or any other repository.

## Published product delta

The immutable [DOC019 product commit](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3) has parent `b708e1ed981b2ba151a00210622b486a22a7a908` and changes exactly three Markdown files:

| Published file | Purpose |
| --- | --- |
| [`ACCESSIBILITY.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/ACCESSIBILITY.md) | New Protos-first statement: scope, priorities, honest limitations, component-specific report routing, privacy and best-effort handling |
| [`README.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/README.md) | Add Accessibility after Support in the Community navigation |
| [`SUPPORT.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/SUPPORT.md) | Link the accessibility policy and explain reporting without a complete technical reproduction |

The published GitHub patch adds 112 lines to `ACCESSIBILITY.md`, two lines to `README.md`, and 13 lines to `SUPPORT.md`: **127 additions, zero deletions, three changed files**. No product source/test files, `spec/**`, `pom.xml`, or `CHANGELOG.md` changed; no implementation or normative specification version bump was needed.

## Validation provenance and limits

**Maintainer attestation, after the DOC019 edits:** `git diff --check` clean; local `make test` PASS; local `make check` PASS; commit and push completed. The coordinator independently inspected the published GitHub commit and exact three-file patch, not the maintainer's local command output. The coordinator did not rerun the suites or independently assert a complete website, editor, or assistive-technology accessibility audit.

The [immutable publication and validation evidence](../../evidence/DOC019/DOC019_ACCESSIBILITY_PUBLICATION_AND_VALIDATION_2026_11_03.md) records the exact verified product revision and distinguishes maintainer-reported validation from GitHub-observed publication. Live state and closure belong to [DOC019 #1045](https://github.com/guillermomolina/protos/issues/1045), not to this durable record.

```text
WORK_ITEM=DOC019
PROTOS_REVISION=250b89bf2faf3e580980a67c2d69aa94d10e4df3
PARENT_REVISION=b708e1ed981b2ba151a00210622b486a22a7a908
PRODUCT_CHANGED_PATHS=3
PRODUCT_ADDITIONS=127
PRODUCT_DELETIONS=0
GIT_DIFF_CHECK=CLEAN_REPORTED_BY_MAINTAINER
MAKE_TEST=PASS_REPORTED_BY_MAINTAINER
MAKE_CHECK=PASS_REPORTED_BY_MAINTAINER
PRODUCT_PUBLICATION=INDEPENDENTLY_VERIFIED
NORMATIVE_SPEC_CHANGED=NO
PRODUCT_IMPLEMENTATION_CHANGED=NO
VERSION_BUMP=NO
```
