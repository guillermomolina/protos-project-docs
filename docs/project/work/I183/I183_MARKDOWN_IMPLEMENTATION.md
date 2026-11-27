# I183 — Markdown parser implementation publication and validation evidence

Date: 2026-11-27.

## Exact identities

- Product work: [I183 #1203](https://github.com/guillermomolina/protos/issues/1203).
- Governing closed decision: [D266 #1194](https://github.com/guillermomolina/protos/issues/1194), [D266-D1 ratified record at 5deb4cdafc02a7be549ce86d98acbacb1d3b6f43](https://github.com/guillermomolina/protos-project-docs/blob/5deb4cdafc02a7be549ce86d98acbacb1d3b6f43/docs/project/decisions/language/D266_D1_REUSABLE_MARKDOWN_PARSER_RATIFIED.md).
- PROTOS_REVISION: [8acc5b07841f815b076143e52d03005a5e1aec32](https://github.com/guillermomolina/protos/commit/8acc5b07841f815b076143e52d03005a5e1aec32).
- Implementation version: 0.4.250-SNAPSHOT, confirmed in the [changelog](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/CHANGELOG.md).
- Publication: `I183: Add CommonMark and GFM Markdown parser with conformance tests`.

## Published implementation

The product revision adds the ordinary Protos `std:markdown` module, with CommonMark 0.31.2 as the default and an explicit CommonMark 0.31.2 plus selected GFM 0.29 extensions profile (tables, task list items, strikethrough, extended autolinks). It returns fresh mutable semantic trees with 22 node kinds, optional original Unicode scalar spans, reference resolution and deterministic input/node/depth/text limits. HTML projection is separately owned by D267. The commit file inventory contains no production Java source changes; Java changes are restricted to test discovery/completeness and progress grouping support.

Revision-bound authorities and evidence:

- [Normative Markdown contract](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/spec/semantics/MARKDOWN.md), [spec changelog](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/spec/PROTOS_SPEC_CHANGELOG.md), [public guide](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/docs/guide/library/markdown.md), [module](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/lib/markdown/module.protos), [example](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/examples/markdown/semantic-tree.protos).
- [Upstream inventory and generation contract](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/official/upstream/README.md), [checksums](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/official/upstream/SHA256SUMS), [provenance](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/official/upstream/PROVENANCE.json), [third-party notices](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/THIRD_PARTY_NOTICES.md).
- [Conformance manifest](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/manifest.tsv), [AST contract tests](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/ast-contract.protos), [resource-boundary tests](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/resource-limits.protos), [adversarial tests](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/adversarial-parsing.protos), [positions tests](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/tests/conformance/library/markdown/source-positions.protos).
- [Corpus and workload contract](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/benchmarks/markdown/README.md), [corpus harness](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/benchmarks/markdown/Corpus.protos), [adversarial workloads](https://github.com/guillermomolina/protos/blob/8acc5b07841f815b076143e52d03005a5e1aec32/protos/benchmarks/markdown/Workloads.protos).

The published generated inventory is 652 official CommonMark examples + 27 selected published GFM extension examples + 2,125 named-entity cases = 2,804 cases, in addition to focused regression tests. This is an inventory count, not a separately observed execution count. Unicode data is pinned to 17.0.0; upstream attribution, retained licenses and checksums are published. No independent exhaustive license audit or generator execution is claimed here.

## Validation provenance

The maintainer explicitly reported after publishing: clean git diff check and all local tests passing. The exact product commit, changelog, file inventory and the above revision-bound documentation were inspected remotely. Builds and tests were not re-executed by the coordinating agent; the reported PASS is maintainer evidence.

| Gate | Evidence |
| --- | --- |
| Product publication | PASS: exact commit verified |
| Product git diff --check | PASS: maintainer report |
| Local test suite | PASS: maintainer report; exact final command list and counts not supplied |
| Official fixture inventory | VERIFIED: published upstream README and source inventory |
| Actual historical 90-document corpus, exact input checksums, remark AST comparison and six renderer fixtures | No independent revision-bound execution results available in this update |
| Adversarial growth measurements | Not established; published benchmark README explicitly states no performance measurements executed for this candidate |
| Explicit packaged JVM/Native parity | No separate parity report or exact command output available in this update |

No missing acceptance evidence is represented as PASS. The implementation is published and local validation reported green. I183's explicit external-corpus, growth-measurement and packaged parity acceptance remains subject to evidence reconciliation before formal closure. D266 remains ratified and closed. [TOOL015 #1216](https://github.com/guillermomolina/protos/issues/1216) is the independent test-selection follow-up, not a Markdown implementation blocker.

## Scope of this record

Non-normative implementation evidence only; no new semantic decision, product modification, priority, milestone or release claim.
