# DOC010-A — Published feature documentation coverage audit

Date: 2026-10-08
Source issue: https://github.com/guillermomolina/protos/issues/848
Historical prompt identifier: DOC006-A (collided with closed #524)
Canonical identifier: DOC010-A
Decision: owner approved recommendation to reconcile identity first, then implement grouped embedding/application-modules/foreign-provider documentation.

## Read-only evidence pins

- Product observed HEAD: guillermomolina/protos@d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae
- Project records observed HEAD: guillermomolina/protos-project-docs@c48ca3e2375dbce0d35896424311426b49a8e1ce
- Website observed HEAD: guillermomolina/protos-website@1834b97c4f2e208bb6335f90383d61ac791bb8b6
- VS Code extension observed HEAD: guillermomolina/protos-vscode-extension@c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a
- Website Protos source lock: 4a6df50e8577ebb0f3fc352df13dcbab368f955a
- Website VS Code grammar lock: 28c572560d51bbda33e6bd0a98f71fafbf2b2526
- Website public release lock: v0.3.0

Commits are observed recent commits, not simultaneous transactional ref snapshots.

## Authority and inspected inputs

Read: product AGENTS.md, AGENTS.work/DOCUMENTATION.md and COORDINATION.md, spec/AGENTS.md and spec/semantics/MODULES.md; product README.md, CHANGELOG.md, docs/README.md, docs/guide/README.md, docs/guide/00-try-protos.md, 06-modules-and-imports.md, 11-process-io-filesystems-and-authority.md, docs/guide/tools/README.md, docs/guide/tools/test-tool.md, protos/examples/README.md, protos/tutorials/README.md and docs/news/README.md; companion website AGENTS.md, README.md, package.json, astro.config.mjs, scripts/prepare-protos.mjs, source/release/grammar locks, homepage and learning/reference/community indices; records AGENTS.md and docs/project/README.md.

Governance: spec remains normative; project implementation/acceptance defines availability, not changelog alone; project records do not override live GitHub coordination. Human executor runs product/website validation and commits. The present approval expressly allows direct project-record publication; do not infer approval of new semantic contracts.

## Coverage findings

| Capability | Status at reviewed product HEAD | Maintained source / gap | Planned response |
|---|---|---|---|
| Core syntax, objects, closures, collection semantics, matching | Guide chapters 01–05 and 12 already exist | Web manual chapter index omits chapter 12 | Reconcile navigation, avoid duplicating spec |
| Modules and imports | Guide 06 exists | New app:/foreign:/java: workflows missing | Extend module guide and add workflows |
| Futures, Actors, isolated parallel | Guide chapters 08–10 and runnable examples exist | Task examples can be improved | Targeted links and examples |
| Process/Filesystem/Network authority | Guide 11 exists | Practical supported-host workflows missing | One authority HOWTO, explicit grants |
| Standard Library datetime/logging/text | Source implementations/tests present; release maturity needs gate verification | No corresponding task guides identified | Grouped library guides |
| Polyglot embedding | PLAT054/I086 published source; distributed scope must be verified | No dedicated Java embedding HOWTO identified | Add embedding guide |
| app: catalogs | I087-2 in 0.3.307-SNAPSHOT, accepted tests and portable smoke reported | No app: howto | Add application module HOWTO |
| generic foreign provider | I082/I085 contracts and implementation present; exact public distribution not assumed | No custom provider HOWTO | Add provider HOWTO |
| Java provider scheme | I088-A in 0.3.312-SNAPSHOT switches jdk: to java: (not alias) | No Java fixture/provider guide | Document java: with clear availability gate |
| Test Tool / Package Tool | Tools overview and Test Tool chapter present; package operations still evolving | Need current command inventory | Update tools and conditional package guide |
| lint | LM012-C1 command implemented in HEAD | No tool guide | Create lint guide with D194/D195 exits/options |
| LSP, diagnostics, editor | LM010 implemented; LM011/LM012 final gates need reconciliation | No precise capability/extension matrix | Editor/LSP guide, no invented CodeActions |
| Native and portable distributions | Public artifacts exist, current version/platform not verified here | README and Try Protos are pinned to 0.3.116 | Update only after artifact verification |
| Website/news | Website consumes exact Protos revision and separate release lock | Source lock 4a6df..., release lock v0.3.0, missing chapter 12 | Update source-derived navigation after guides |

Audit limitation: not an exhaustive review of every normative section, full artifact inventory, or live deployment. Any remaining gap must be verified at slice time, not silently counted as accepted.

## Publication architecture

Product README, docs/guide, examples/tutorials and docs/news are canonical in guillermomolina/protos. Astro/Starlight site under guillermomolina/protos-website materializes these via scripts/prepare-protos.mjs from protos-source.lock.json; generated content is not an independent authoritative copy. API pages are derived at build time. Home page and manual nav indices live in website. GHCR production image and deployment are separately governed; successful static build is not proof of live deployment.

## Grouped implementation work

- DOC010-B (PRODUCT): integration/embedding: docs/guide/06-modules-and-imports.md, new docs/guide/embedding/{java,application-modules}.md and docs/guide/interop/{README,custom-provider,java-provider}.md, documentation indexes and focused example(s). Respect I086/I087/I082/I085/I088 authority/gates. Avoid promises that java: fixture is universally bundled.
- DOC010-C (PRODUCT): libraries and tools: new library family guides, lint and editor/LSP guides, reconcile package/test-tool docs.
- DOC010-D (PRODUCT): README, getting started, distributions and authority HOWTO, guide/examples navigation after release verification.
- DOC010-E (WEBSITE): regenerate/reconcile source lock, site navigation/pages; human builds/checks.
- DOC010-F (PRODUCT and PROJECT RECORD): one dated news article only once published capabilities and links are verified, then final evidence and coordination.

Each product/website implementation is edited by agent; human executor executes validation, git diff/check, commit/push. Project record repository is the only authorized agent-publication destination in this user turn.

## Risks and explicit STOP gates

- Former DOC006 collision with #524: corrected to DOC010 at #848; historical prompt name retained as provenance only.
- Source 0.3.312-SNAPSHOT is not evidence that java: is shipped in a release.
- No generic Java reflection claim or unbounded host authority.
- Native CLI is not native Java embedding.
- LM012 final, LM011 editor and Package Tool acceptance must be checked prior to stable claims.
- No assumed newest GitHub release and no fabricated article date.
- DOC010-A investigation performed read-only; no builds/tests or local commands.

Next recommended: DOC010-B, implementation in guillermomolina/protos, self-contained from local HEAD, grouping integration and Java provider documentation.