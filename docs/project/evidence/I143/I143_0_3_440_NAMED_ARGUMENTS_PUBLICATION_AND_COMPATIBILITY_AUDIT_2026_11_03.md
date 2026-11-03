# I143 — Named arguments implementation, publication and compatibility audit (0.3.440)

**Date:** 2026-11-03. **Formal implementation owner:** [I143 #1039](https://github.com/guillermomolina/protos/issues/1039). **Design authority:** ratified [D237 #1038](https://github.com/guillermomolina/protos/issues/1038) contextual syntax, MIX-C and R3, in the [immutable D237 decision record](../../decisions/language/D237_NAMED_ARGUMENTS_CONTEXTUAL_MIX_C_R3_RATIFIED.md) published at `guillermomolina/protos-project-docs@4adb907c067edf3fc7f5579b514740f49119b20b`. This record is **non-normative, revision-bound evidence**; the normative grammar and semantics are in the published Protos `spec/` files.

## Verified product publication

- **PROTOS_REVISION:** [`c2a2b8ccbf51874b89d3c5a3b714d104c08ee414`](https://github.com/guillermomolina/protos/commit/c2a2b8ccbf51874b89d3c5a3b714d104c08ee414)
- **PARENT_REVISION:** `2e85e8dc0b85548add6962ef6556f24513bc8d05`
- **COMMIT_TITLE:** `I143: implement named arguments and slot-creation disambiguation (0.3.440)`
- **IMPLEMENTATION_VERSION:** `0.3.440-SNAPSHOT`, with the committed `pom.xml` and `CHANGELOG.md` updated together
- **SPECIFICATION_VERSION:** `0.1.492`, as recorded in the committed implementation changelog and normative specification changelog
- **PUBLISHED_DELTA:** 40 files; 4,463 additions and 123 deletions, reported by GitHub's published commit API. Groups: 16 `src/main/**` files, 13 `src/test/**` files, six `spec/**` files, two `protos/**` files, one `tools/**` file and both version/changelog files.
- **TEST_ARTIFACTS:** 13 added `ProtosI143*` / parser/canonicalizer Java tests; `protos/tests/conformance/surface-sugar/named-arguments.protos` added and registered in `protos/tests/conformance/manifest.tsv`.

## Published implementation scope

The product changelog and revision-bound diff document structural parsing of `identifier: expression` as a named argument in ordinary call-item position. Explicit grouping, e.g. `f((a: 1))`, preserves the previous positional slot-creation expression. The approved Array-literal and contextual-control precedences remain governed by the D237 normative specification.

The published source adds positional/name-bearing argument representation and association after argument evaluation, positional ordinal stability, selected Closure signature matching, unknown-name and positional/name collision errors, default/presence/rest behavior, and forwarding through the actual standard `Object.call` path to Closure activation or selected constructor `init`. It also updates parser, lowering, runtime, specification, Java regressions, suite-native conformance and the generated-bytecode BCI topology baseline. The changelog records static-tooling/formatter/LSP coverage; no separate tooling repository publication is claimed here.

## Source-compatibility audit: earlier slot creation as an argument

The owner requested a pre-commit check for old programs that rely on `f(a: 1)` to **create a slot and pass its result** instead of binding a named formal. The migration spelling, if needed, is `f((a: 1))`; analogous Array-literal cases use `[(a: 1)]`. The audit was not a literal search for `a: 1`: the supplied read-only Python scanner reads identifiers generically (`label.isidentifier()`), tracks nested `(`, `[`, and `{` boundaries, checks comma-delimited arguments across newlines, masks comments and strings, distinguishes grouped expressions, and excludes the newly added named-argument conformance file. It is a lexical approximation, **not** a parser proof of semantic compatibility.

The maintainer executed `python3 /tmp/I143_audit_slot_creation_arguments.py` in the Protos working tree and reported:

```text
AUDITED_PROTOS_FILES=909
POTENTIALLY_CHANGED_ARGUMENT_ITEMS=0
SCAN_ERRORS=0
```

This is evidence that the scanner found **no migration candidates** in the 909 audited `.protos` files, not proof that no other compatibility concern exists. The auditor itself was supplied in the implementation discussion as a temporary script, not as a file in the I143 product commit. No previously existing source call was migrated solely on the basis of this audit.

## Validation and provenance

**Maintainer-reported local results, after the product push:**
- `git diff --check`: **CLEAN**, reported.
- `make test`: **PASS**, reported.
- `make check`: **PASS**, reported.
- Product commit/push: **COMPLETE**, verified independently against GitHub at the exact revision above.

**Independently checked from GitHub:** published commit SHA and parent, exact title, 40 changed paths and file groups, the committed implementation changelog, and the existence and registration of the named-arguments conformance artifact.

**Not independently executed or established in this coordination pass:** local full-suite reproduction, performance/benchmark comparison, Native Image-specific execution, whole-tree parser-level compatibility proof, or separate extension/Marketplace publication. Those outcomes must not be inferred from passing integration checks.

## Completion/coordination boundary

This I143 record is the required version-pinned implementation publication evidence and does not supersede D237's ratified decision. Live acceptance, closure and Project status remain in [I143 #1039](https://github.com/guillermomolina/protos/issues/1039); D237's design lifecycle remains owned by [D237 #1038](https://github.com/guillermomolina/protos/issues/1038). Any unverified Project routing or post-publication coordination state must be reported rather than silently assumed.

```text
WORK_ITEM=I143
PROTOS_REVISION=c2a2b8ccbf51874b89d3c5a3b714d104c08ee414
PARENT_REVISION=2e85e8dc0b85548add6962ef6556f24513bc8d05
PRODUCT_VERSION=0.3.440-SNAPSHOT
SPEC_REVISION=0.1.492
PRODUCT_PUBLICATION=VERIFIED
PRODUCT_CHANGED_FILES=40
AUDITED_PROTOS_FILES=909
POTENTIALLY_CHANGED_ARGUMENT_ITEMS=0
SCAN_ERRORS=0
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
PERFORMANCE_BENCHMARK=NOT_ATTESTED
NATIVE_IMAGE_TEST=NOT_ATTESTED
PROJECT_ROUTING=NOT_INDEPENDENTLY_VERIFIED
```
