# DOC015 — Contextual non-local return: one-slice documentation, tutorials and examples

**Issue:** [DOC015 #1025](https://github.com/guillermomolina/protos/issues/1025) (`family:DOC`, `status:ready` at creation, unassigned, priority unset). **Created:** 2026-11-01. **Authority:** Already shipped [D234-B2-S1 / I138](https://github.com/guillermomolina/protos/issues/1021), exact source `guillermomolina/protos@ab637aa264ddbb2b99fdb03cb19207cc19651f45` (product 0.3.432-SNAPSHOT, normative spec 0.1.487).

**Classification:** documentation-only **IMPLEMENTATION/authoring** in local `guillermomolina/protos`, not D research, and not a new runtime feature. **Publication:** after human validation, one product commit/push; the GitHub Issue owns live status, and product source remains the normative authority.

## Scope and files (one slice)

The implementation I138 only added a short prose introduction in the closure guide, tests and normative grammar. This separate DOC item develops runnable teaching, discoverable in the existing progression:

- New `protos/tutorials/03-closures/03-return-home.protos`: small runnable CLI lesson showing that `return value` and `^value` have the same non-local return home, and distinguishing implicit final-expression result from early return in a nested callback.
- New `protos/examples/closures/non-local-return.protos`: a task-oriented first-match search with callback, early `return`, equivalent `^` variant, and a stable fallthrough case.
- Existing `protos/tutorials/README.md` and `protos/examples/README.md`: link/index the new runnable sources.
- Existing `docs/guide/03-closures-methods-and-receivers.md`: extend existing I138 section minimally with a mental model, grouped ordinary identifier escapes, return-home lifetime and links; no duplicated normative table.
- Existing `docs/guide/04-control-flow-through-protocols.md`: show non-local return with existing contextual `if` and ordinary Boolean/Closure callbacks; teach effects of `ensure` without implying JavaScript-style callback-local return.
- Existing `docs/guide/README.md`: add discoverable route/link preserving prior navigation.

**Total:** two new executable Protos files plus five existing indices/guide files. No `spec/`, `src/`, `pom.xml`, `CHANGELOG.md`, runtime, test harness or language semantics; no version bump for doc-only material. All development text is English with existing APL Part 5 notices on new Protos sources. Keep existing DOC014 bitwise teaching separate.

## Semantic authority and examples

For the **already published** specification, source read anchors are:

- [Grammar section 36](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/PROTOS_GRAMMAR.md) — contextual ordinary `return`, exact recognized and protected cases, same-line operand requirement, grouped escapes, no ASI or empty return statement.
- [Callable return semantics §13–14](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/semantics/CALLABLES.md) — captured return home, `InvalidReturn`, escaped or foreign-Task return, nested closures and `ensure`.
- [I138 conformance](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/protos/tests/conformance/surface-sugar/contextual-return.protos) — module-level helper pattern avoids Test Tool early-return vacuity.

Mandatory teaching distinction: `return item` in an ordinary nested callback returns from the enclosing home invocation, not from only the callback. `return` remains an identifier outside the approved contextual syntax; `return()`, `return(x,y)`, `object.return(x)`, `(return)(x)`, `return: x` etc. stay ordinary; `return x` is non-local. No `return;`, no bare-return implicit null and no operand continuation over a logical NEWLINE. Use `return (%{...})` for a Map, not `return %{...}`.

## Human-executor validation / closure

Read local `AGENTS.md`, `AGENTS.work/DOCUMENTATION.md`, `protos/AGENTS.md`, `docs/guide/SOURCE_STYLE.md` and current normative grammar. An editing agent may perform bounded text/source edits without executing tests, build, scripts or Git mutation. The human runs executable lesson + example via the actual CLI, verifies exact output, source style/formatter/README links, `git diff --check`, and required `make test` / `make check` per local impact policy. The two print-dependent sources must be automatically exercisable by existing CLI regression discovery, with no accidentally bypassed result assertions or host authority.

After a successful **one-commit** product publication, add a revision-bound record under `docs/project/evidence/DOC015/`, update the Issue and close only after recording the exact product/docs SHAs and human PASS claims. No DOC015 implementation or test is claimed by this planning record.
