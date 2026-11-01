# DOC016 — Protos-first contextual for: publication and validation evidence

**Date:** 2026-11-01. **Owning Issue:** [DOC016 #1032](https://github.com/guillermomolina/protos/issues/1032). **Evidence kind:** immutable product identity, exact source inspection, and attributed human validation.

## Verified product revision

- PROTOS_REVISION: [fadb184409fc9b985e19db4a75ca76461592ff6b](https://github.com/guillermomolina/protos/commit/fadb184409fc9b985e19db4a75ca76461592ff6b); commit subject: DOC016: add Protos-first contextual for tutorial, example and guide links.
- GitHub-reported commit timestamp: 2026-11-01T12:44:58Z.
- Parent product commit: 1c774003a2c697b7c2adae9967b562872b8d652a (I141).
- Exact published change: **7 paths, 432 insertions, 41 deletions**.

| Role | Exact revision-locked path |
| --- | --- |
| Main guide | [docs/guide/04-control-flow-through-protocols.md](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/docs/guide/04-control-flow-through-protocols.md) |
| Range guide | [docs/guide/05-values-identity-equality-and-collections.md](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/docs/guide/05-values-identity-equality-and-collections.md) |
| Guide navigation | [docs/guide/README.md](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/docs/guide/README.md) |
| Cookbook navigation | [protos/examples/README.md](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/protos/examples/README.md) |
| New cookbook source | [protos/examples/algorithms/stride-sampling-summary.protos](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/protos/examples/algorithms/stride-sampling-summary.protos) |
| New tutorial source | [protos/tutorials/04-control-flow/04-contextual-for.protos](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/protos/tutorials/04-control-flow/04-contextual-for.protos) |
| Tutorial navigation | [protos/tutorials/README.md](https://github.com/guillermomolina/protos/blob/fadb184409fc9b985e19db4a75ca76461592ff6b/protos/tutorials/README.md) |

Both new Protos sources include the full APL Part 5 notice, use the shipped contextual for syntax and contain print calls. The chapter 04 guide links those sources and the conformance/Range material. No spec, parser, runtime, library/tool, test-harness, version or changelog file appears in the product delta.

## Human-reported validation (not coordinator-executed)

Owner statement on 2026-11-01: "DOC016: add Protos-first contextual for tutorial, example and guide links pushed, el git diff check esta limpio. make test y make check pasan en local."

| Gate | Result |
| --- | --- |
| GitHub commit and exact changed paths | VERIFIED by coordinator from published GitHub |
| New Protos content, APL notice, chapter 04 link targets | VERIFIED by coordinator from exact published source |
| git diff --check | PASS — human-reported |
| make test | PASS — human-reported |
| make check | PASS — human-reported |
| CLI learning-material discovery | SOURCE VERIFIED — unchanged ProtosCliLearningMaterialsTest discovers print-dependent programs under tutorials/examples; overall PASS is human-reported |
| Byte-for-byte comparison against expected stdout | NOT separately attested; sample expected output for cookbook is 2, 55, true (three lines) |
| Native Image or separate CI run | NOT attested |
| Coordinator local command/test execution | NONE |

## Scope and cross-references

DOC016 completes educational documentation for already-shipped [I141 #1030](https://github.com/guillermomolina/protos/issues/1030) and the ratified [D231 #1013](https://github.com/guillermomolina/protos/issues/1013). It does not imply closure or mutation of [I142 #1033](https://github.com/guillermomolina/protos/issues/1033), which separately owns eligible Protos-source migration.

The maintained [DOC016 work record](../../work/DOC016/DOC016_CONTEXTUAL_FOR_LEARNING_MATERIAL_COMPLETION_2026_11_01.md) and [DOC016 Issue](https://github.com/guillermomolina/protos/issues/1032) complete cross-reference navigation. The Issue closure comment records the exact project-docs publication SHA after GitHub commit readback.
