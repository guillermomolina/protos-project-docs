# DOC017 — D213 Array Standard Library tutorials and cookbook completion

**Owning work:** [DOC017 #1036](https://github.com/guillermomolina/protos/issues/1036) in `guillermomolina/protos`.
**Product publication:** [`08a3f4538b538f00649829b3200f19cba808ebfc`](https://github.com/guillermomolina/protos/commit/08a3f4538b538f00649829b3200f19cba808ebfc) in `guillermomolina/protos`, verified through the GitHub commit and all four newly published Protos source files.
**Date:** 2026-11-02.
**Classification:** completed independently tracked `DOCxxx` learning material, following [LIB023 #953](https://github.com/guillermomolina/protos/issues/953) (completed) and [D213 #949](https://github.com/guillermomolina/protos/issues/949) (ratified); not an additional Array API, implementation slice, design approval or specification revision.
**Implementation and normative baseline:** the parent Array API was already published at [`91c40a94f4fc12844c0757f131abe436d56314a5`](https://github.com/guillermomolina/protos/commit/91c40a94f4fc12844c0757f131abe436d56314a5), Protos `0.3.438-SNAPSHOT`, normative revision `0.1.491`. DOC017 itself changes no executable library/runtime source, Maven version, root implementation changelog or normative specification.

## Exact published learning-material delta

The product commit contains **exactly seven paths**:

| Role | Product repository path |
| --- | --- |
| Progressive tutorial — copies, queries and default sort | [`protos/tutorials/05-collections/03-array-copies-and-queries.protos`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/tutorials/05-collections/03-array-copies-and-queries.protos) |
| Progressive tutorial — bulk transformations | [`protos/tutorials/05-collections/04-array-bulk-transformations.protos`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/tutorials/05-collections/04-array-bulk-transformations.protos) |
| Cookbook — incident severity, teams and stable groups | [`protos/examples/collections/array-incident-summary.protos`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/examples/collections/array-incident-summary.protos) |
| Cookbook — stock pairing and deduplication | [`protos/examples/collections/array-pairing-and-deduplication.protos`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/examples/collections/array-pairing-and-deduplication.protos) |
| Progressive tutorial navigation | [`protos/tutorials/README.md`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/tutorials/README.md) |
| Cookbook navigation | [`protos/examples/README.md`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/protos/examples/README.md) |
| Existing programmer's guide cross-links | [`docs/guide/05-values-identity-equality-and-collections.md`](https://github.com/guillermomolina/protos/blob/08a3f4538b538f00649829b3200f19cba808ebfc/docs/guide/05-values-identity-equality-and-collections.md) |

All four new `.protos` files carry the existing full APL license header. They are ordinary, self-contained, standalone-CLI programs requiring only the published `std:collections/Array` Standard Library import, with no ambient network, resolver or other special host authority.

### Progressive lessons

- `03-array-copies-and-queries.protos` demonstrates unchanged original input length and contents, fresh `append`/`concat`/`slice`/`insertAt`/`removeAt`/`reverse` results, default-comparator `sort`, `find`, ordered `contains`, `any`, `all`, `findIndex` for the matching-`null` ambiguity, and saturating `take`/`drop`. It contains **18 Boolean-print checks**.
- `04-array-bulk-transformations.protos` composes `flatMap`, `zip`, `partition`, and `distinct`, explicitly checks new zip pair identities, the shortest-input boundary, stable grouping, source preservation, and callback snapshot behavior when a callback mutates the original input. It contains **9 Boolean-print checks**.

### Task-oriented cookbook

- `array-incident-summary.protos`: five incident records with ordinary newline-separated Protos object slots; stable urgent/routine partition, distinct responsible teams, prefix selection, matching incident lookup, and strict predicate summaries. **8 Boolean-print checks**.
- `array-pairing-and-deduplication.protos`: pairs item names with available counts, deliberately truncates an unmatched tail, partitions stock thresholds, flattens pairs, deduplicates names, and checks membership. **9 Boolean-print checks**.

The existing guide's already-published LIB023 normative explanation remains intact: this change adds educational navigation instead of duplicating semantic authority. Learning material uses real Protos syntax and regular array APIs; it does not imply mutable Array growth, in-place Array copy operations, new Core selectors, new Java/Truffle mechanisms, or an alternate public Array module.

## Validation provenance

**The maintainer, not this publishing assistant, ran the validations in the local product checkout.** The reported observations:

1. Initial tutorial-focused run produced **27 printed `true` values** (18 from `03-array-copies-and-queries` and 9 from `04-array-bulk-transformations`).
2. Initial incident cookbook lint found an invalid comma within object slot definitions. A bounded correction rewrote the five event objects to the Protos newline-separated slot syntax; the exact corrected source was subsequently verified in the published commit.
3. Re-run of `bin/protos lint` and `bin/protos` for both cookbook programs printed **8 + 9 = 17 `true` values** with no observed errors, bringing the four new programs to **44/44 successful assertions**.
4. The maintainer reported **`git diff --check` clean** after the focused corrections, and subsequently **all tests PASS** following the requested `make test && make check` integrated validation.
5. The subsequent Git push updated `main` from `91c40a94` to `08a3f453`, and GitHub commit read-back confirmed that exactly the seven intended learning-material files were published.

These are explicitly **human-reported execution and validation results**. The publishing assistant separately verified immutable GitHub commit identity, changed path names and the four published program contents; it did not independently run Protos, Maven, `make test`, `make check`, Native Image or full stdout snapshot comparison.

## Completion and authority boundaries

- DOC017's tutorials, cookbook examples and navigation are published as a single coherent seven-path learning-material change.
- The existing [LIB023-B/C completion record](../../evidence/LIB023/LIB023_BC_438_ARRAY_COLLECTION_COMPLETION_2026_11_02.md) records the prior Array API implementation; DOC017 consumes it and makes the approved operations easier to learn rather than extending implementation scope.
- A separate documentation-only publication here does not advance `pom.xml`, `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md` or create a tagged release. The normative API owner remains `guillermomolina/protos/spec/semantics/VALUES_AND_COLLECTIONS.md`.
- Live status and closure belong to [DOC017 #1036](https://github.com/guillermomolina/protos/issues/1036); this record is durable, revision-coupled completion evidence, not a replacement for that Issue.

**PROTOS_REVISION=`08a3f4538b538f00649829b3200f19cba808ebfc`**

