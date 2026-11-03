# DOC018 — D237 named-argument guide, tutorials, and cookbook completion

**Owner:** [DOC018 #1040](https://github.com/guillermomolina/protos/issues/1040). **Date:** 2026-11-03. **Classification:** independently tracked, non-normative documentation and executable learning materials. **Product publication:** [`guillermomolina/protos@2c89adf6062f212c4d9e20f68b919e843a05b523`](https://github.com/guillermomolina/protos/commit/2c89adf6062f212c4d9e20f68b919e843a05b523), parent `c2a2b8ccbf51874b89d3c5a3b714d104c08ee414`.

## Authority and boundary

DOC018 documents the already-ratified [D237 contextual named-argument design](../../decisions/language/D237_NAMED_ARGUMENTS_CONTEXTUAL_MIX_C_R3_RATIFIED.md) and already-shipped [I143 #1039](https://github.com/guillermomolina/protos/issues/1039), rather than changing the language. The published I143 prerequisite supplies the name-aware call implementation and the normative specification `0.1.492`; the product implementation version remains `0.3.440-SNAPSHOT`. DOC018 introduces no runtime/Java/Standard Library changes, no new normative semantics, no specification changes, and no `pom.xml` or `CHANGELOG.md` edits.

**Canonical normative owners:** `spec/PROTOS_GRAMMAR.md` (contextual argument syntax and Array-literal equivalence), `spec/semantics/CALLABLES.md` (MIX-C evaluation, association and standard `Object.call` forwarding), `spec/semantics/ERRORS.md`, and `spec/semantics/VALUES_AND_COLLECTIONS.md` in the published product. The durable D237 design decision remains non-normative; the published product specification is authoritative.

## Exactly six published product paths

| Role | Immutable product source |
| --- | --- |
| New progressive tutorial | [`protos/tutorials/07-call-arguments/02-named-arguments.protos`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/tutorials/07-call-arguments/02-named-arguments.protos) |
| New cookbook program | [`protos/examples/closures/named-arguments.protos`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/examples/closures/named-arguments.protos) |
| Closure guide, D237 section | [`docs/guide/03-closures-methods-and-receivers.md`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/docs/guide/03-closures-methods-and-receivers.md) |
| Guide navigation | [`docs/guide/README.md`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/docs/guide/README.md) |
| Tutorial navigation | [`protos/tutorials/README.md`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/tutorials/README.md) |
| Cookbook navigation | [`protos/examples/README.md`](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/examples/README.md) |

GitHub's exact product commit reports six files, 189 insertions, zero deletions. Both new `.protos` programs carry the full existing APL notice. Neither requires extra host authority.

The tutorial follows the earlier D203 positional/default/rest lesson and provides eleven printed Boolean checks: name reordering, mixed named and positional values, positional spread, once-only left-to-right source evaluation, omitted-versus-`null` defaults, positional-only rest, `f((a: value))` slot creation, and `[(a: value)]` Array-literal slot creation. The cookbook builds and triages incident objects through the standard constructor `Object.call` / `init` path, mixing named optional fields and defaults, with six printed Boolean checks.

The guide explains the explicit D237 compatibility change from slot creation to named call items; fixed positional ordinals and dynamic selected-call association; duplicate-label parsing errors versus dynamic unknown-name/collision errors; ordinary `Object.call` forwarding and its custom/native/interop limits; and contextual-control precedence. The three README changes make the runnable learning material discoverable without duplicating normative specification text.

## Validation and publication provenance

**Maintainer-executed and reported:**

- Batch 1 program runs: eleven `true` outputs from the tutorial and six `true` outputs from the example; 17/17 instructional Boolean checks reported successful.
- Batch 2 Python/ZIP Markdown preflight and application: four files patched, +74 guide lines, +10 guide-index lines, +9 tutorial-index lines, and +1 example-index line.
- `git diff --check`: clean, with the precise four-file Markdown diff supplied for review.
- Integrated final `make test`: **PASS**, owner-reported after the six-file substantive candidate was ready.
- Integrated final `make check`: **PASS**, owner-reported.
- Final `git commit` and `git push origin main`: completed by the maintainer. The supplied terminal output shows the exact SHA and a clean `git status --short` after publication.

**Coordinator-independent GitHub read-back:** product commit hash, parent, subject and six-path delta verified against the published `main`. The coordinator did not independently run the local Protos CLI, full suite, Native Image, or an additional link-check executable. The earlier proposed static link/check script has no separately supplied execution output, so its independent PASS is not claimed; the guide and navigation links were inspected as part of the bounded Markdown review.

## Durable evidence and live closure

See [revision-coupled DOC018 evidence](../../evidence/DOC018/DOC018_D237_NAMED_ARGUMENTS_PUBLICATION_AND_VALIDATION_2026_11_03.md). This maintained work record does not mirror live Issue scheduling, assignee, priority or Project fields. The final [DOC018 #1040](https://github.com/guillermomolina/protos/issues/1040) comment owns closure and identifies the exact project-docs SHA once the durable publication has been completed and reread.

```text
WORK_ITEM=DOC018
PROTOS_REVISION=2c89adf6062f212c4d9e20f68b919e843a05b523
PRODUCT_PARENT=c2a2b8ccbf51874b89d3c5a3b714d104c08ee414
CHANGED_PATHS=6
ADDITIONS=189
DELETIONS=0
NEW_STANDALONE_PROTOS_PROGRAMS=2
HUMAN_CLI_CHECKS=17_TRUE_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
PRODUCT_PUBLICATION=VERIFIED
SPEC_REVISION=0.1.492_UNCHANGED
PRODUCT_VERSION=0.3.440-SNAPSHOT_UNCHANGED
```
