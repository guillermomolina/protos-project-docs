# BUG022 — TOOL010 document-boundary comments: final publication and closure evidence

Date: 2026-10-20

Non-normative durable closure evidence for
[BUG022 / guillermomolina/protos#860](https://github.com/guillermomolina/protos/issues/860).
The product source, the ratified D183 formatting contract, and PLAT050's
single bundled Protos formatting authority remain unchanged by this record.

## Stable publication identities

```text
WORK_ITEM=BUG022
ISSUE=guillermomolina/protos#860
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=0bceacdfc0127453729f346382152bd9908f3de1
PRODUCT_COMMIT=BUG022: fix TOOL010 document-boundary comments
IMPLEMENTATION_VERSION=0.3.388-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=7
SOURCE_TEST_FILE_COUNT=5
VERSION_AND_CHANGELOG=PUBLISHED
SPECIFICATION_CHANGE=NO
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PRE_FINALIZATION_CHECKPOINT_REVISION=d32524bd1dec045966dc33f3a3f58abc4dcf0278
```

**Final verified source publication:**
[protos@0bceacdfc0127453729f346382152bd9908f3de1](https://github.com/guillermomolina/protos/commit/0bceacdfc0127453729f346382152bd9908f3de1).
This `main` commit directly descends from `60af4eac59a08d31184d517d064878243cdb0914`
and includes seven paths:

```text
CHANGELOG.md
pom.xml
protos/tools/formatter/Structural.protos
src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java
src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java
src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java
src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java
```

The coordinator re-read `pom.xml` and `CHANGELOG.md` on GitHub at the final
product `main`: `pom.xml` declares `0.3.388-SNAPSHOT`; the root
changelog opens with a matching BUG022 section describing file-leading and
file-trailing comment preservation, TOOL010's routing and regression tests.

## Product defect, implementation, and invariant preservation

Original defect: `protos format` exited 70 with
`bundled TOOL010 structural formatter did not return String; state=FAILED`
for a leading `/* probe */` and the checked-in `slots.protos` APL license.
The host source-layout projection now marks comments before the first code
token `DOCUMENT_PREFIX` and comments after the last code token
`DOCUMENT_SUFFIX`. Bundled `Structural.protos` emits them at the
document boundaries with source-preserved raw comment payload and canonical
whitespace, retaining the fail-closed guard on unsupported internal cases.
CLI and LSP remain on the same Protos-owned TOOL010 formatting authority.

The five source/test file blob SHA identities in the final `0bceacdf`
commit were independently checked against the historical `d32524bd`
commit and **all five match exactly**. The forced update replaced the branch
tip/commit identity, not the tested source or regression bytes; `0bceacdf`
also adds the required version and changelog metadata. `d32524bd` is a
retained historical checkpoint, not the final current product identity.

## Maintainer-reported acceptance

The human maintainer reported on 2026-10-20:

- Focused suite initially **39 tests**, all PASS.
- Expanded focused suite **41 tests; 0 failures, 0 errors, 0 skipped**,
  covering source-layout classification, bundled TOOL010 preservation,
  deterministic/idempotent formatting, structural and semantic source
  invariants, shipped `slots.protos` license, and real LSP formatting.
- `mvn package -DskipTests` **BUILD SUCCESS**, exit 0, used to refresh
  the checkout JAR without running the full suite.
- Fresh checkout `bin/protos format` with leading
  `/* probe */\nx: 10\n`: exit **0** and preserved comment/code.
- Fresh checkout `bin/protos format protos/examples/basics/slots.protos`:
  exit **0**, preserving the printed APL license header.
- Final integrated `make test`: **PASS** as reported by the maintainer.
- No tests rerun after the final version/changelog-only publication, as
  explicitly required by the maintainer.

The coordinator did not execute local tests or receive a full retained
`make test` transcript. The statements above are maintainer-reported
acceptance; the source/content identities and metadata were independently
verified through GitHub. No Native Image release acceptance is claimed.
The previously observed CLI exit 70 after the source patch used a stale
checkout JAR, and disappeared after the JAR was rebuilt.

## Publication reconciliation and closure

The first source-only publication
[protos@d32524bd1dec045966dc33f3a3f58abc4dcf0278](https://github.com/guillermomolina/protos/commit/d32524bd1dec045966dc33f3a3f58abc4dcf0278)
contained five source/test paths while Maven and the root changelog still
advertised 0.3.387. That discrepancy was documented at
[the historical BUG022 checkpoint](BUG022_TOOL010_DOCUMENT_BOUNDARY_COMMENTS_PUBLICATION_CHECKPOINT_2026_10_20.md)
in docs publication
[`7b046959663591dcd2b42e23d0cb1eeb20b5bce0`](https://github.com/guillermomolina/protos-project-docs/commit/7b046959663591dcd2b42e23d0cb1eeb20b5bce0).

The maintainer subsequently published `main` with a forced ref update to
`0bceacdfc0127453729f346382152bd9908f3de1`, combining the same five source/test blobs and the two required
metadata files into one product commit. The prior publication gap is
**resolved**, not hidden or rewritten from the historical evidence.

`PRODUCT_PUBLICATION=PASS` and `VERSION_CHANGELOG_INVARIANT=PASS`.
This durable record satisfies the requested cross-repository project evidence,
subject to verification of this record's own exact documentation commit.
The final GitHub Issue comment and `state=closed / completed` are separate
live-coordination steps to be performed after record publication is confirmed.

`DURABLE_RECORD_DECISION=REQUIRED`, as the maintainer explicitly requested
project-docs evidence. `SPECIFICATION_CHANGE=NO`. No separate design
ratification or additional implementation issue is required.
