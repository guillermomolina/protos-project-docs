# BUG040 — Maintainer-reported implementation and validation checkpoint

Date: 2026-11-25

- Owning issue: [BUG040 #1174](https://github.com/guillermomolina/protos/issues/1174).
- Dependent work: [I176 #1173](https://github.com/guillermomolina/protos/issues/1173).
- Reported product commit subject: `BUG040: Preserve formatter comments and add Protos source-preservation comparison`.
- Exact product revision: [`efb2dd5dcc56b0d9b3752dde1d739ae672897946`](https://github.com/guillermomolina/protos/commit/efb2dd5dcc56b0d9b3752dde1d739ae672897946). Verified as the remote main head after the initial visibility delay. The published change contains the three formatter Protos files, three Java test files, pom.xml and CHANGELOG.md; no production Java changes.

## Reported implementation

Map and Object entry rendering shares the existing sequence-boundary renderer with Program and Closure sequences. Comment consumption is checked per attachment, with rollback for discarded provisional renderings. Unconsumed admitted comments fail closed.

The new Protos-side Preservation.protos comparator, exposed through the formatter module as preservesSource, compares ordered source-layout facts, including comments, structural paths, separator kinds, significant blank-line facts, structural and Closure forms, trailing Closures, contextual else and token spellings. Production Java and language semantics were not changed in the reported implementation.

## Validation provenance

The maintainer explicitly reports a clean git diff --check, all local tests passing, and publication. Earlier focal evidence reported 38 source-layout tests, 42 formatter bootstrap tests and 16 LSP formatting tests passing; the subsequent comparator slice reported 51 formatter bootstrap tests passing (including nine new guest-policy tests). These counts refer to successive validation checkpoints, not additive totals. The final full-suite test count and exact run identity were not supplied.

## Remaining acceptance work

The last implementation report explicitly states that whole-document acceptance is not connected to preservesSource. Independent candidate reparsing, span-independent canonical equivalence and invocation of the production Protos comparator before success remain unverified/unimplemented on the available evidence. Direct negative production candidate-acceptance tests and the I176 contextual match regressions remain outstanding. Passing guest-policy tests do not establish the production acceptance gate.

At this checkpoint the implementation report still described BUG040 and I176 as blocked. On 2026-11-25 the maintainer explicitly approved closing BUG040 for the corrected defect and transferring the additional general verification gate to independent [I177 #1177](https://github.com/guillermomolina/protos/issues/1177). I176 may resume and must retain and rerun its contextual match formatter/LSP regressions; it does not depend on I177. PERF044 remains independent.

This revision-bound record supports BUG040 closure under the explicitly narrowed scope. The general production acceptance gate and its negative integration tests are not claimed complete; they are owned by I177. Maintainer-reported clean diff and all local tests passing remain the validation provenance.
