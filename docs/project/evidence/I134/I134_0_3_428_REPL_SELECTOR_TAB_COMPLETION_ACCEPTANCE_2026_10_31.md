# I134 — REPL receiver-selector TAB completion, 0.3.428

**Date:** 2026-10-31. **Role:** immutable implementation and maintainer-acceptance evidence for [I134 #1002](https://github.com/guillermomolina/protos/issues/1002). Non-normative; live Issue state belongs to GitHub and language semantics remain under `protos/spec/`.

- **Exact published product revision:** [`guillermomolina/protos@02e32411ac614ae69b974757430d8a29d291619d`](https://github.com/guillermomolina/protos/commit/02e32411ac614ae69b974757430d8a29d291619d).
- **Commit subject:** `I134: add REPL selector Tab completion (0.3.428)`.
- **Published Maven version:** `0.3.428-SNAPSHOT` (release-series shorthand `0.3.428`).
- **Approved design:** [D228-A1](../../decisions/tooling/D228_REPL_RECEIVER_SELECTOR_TAB_COMPLETION_AUTHORITY_AND_DELEGATION.md) / [D228 #1001](https://github.com/guillermomolina/protos/issues/1001), ratified in this documentation repository at [`2327f7196d19125d38e031d080ab020831d5e81c`](https://github.com/guillermomolina/protos-project-docs/commit/2327f7196d19125d38e031d080ab020831d5e81c). Preserves the prior D227-A1 / CLI011 bare-lexical-name completion contract.

## Exact product publication

The published I134 product commit was retrieved from GitHub and contains **24 changed paths**: `CHANGELOG.md`, `pom.xml`, `docs/guide/tools/cli.md`, the static member-site proof and AST traversal, CLI/JLine candidate composition, names-only lexical binding authority helpers, admitted runtime selector observation, RootActor scheduler observation permit, Process direct-entry observation barrier, related JUnit tests and two PE-guard baselines. No `spec/` or Protos Standard Library source is changed by this commit.

The feature proves `SurfaceMember` sites through the actual lexer and parser, accepting existing lexical receiver names or `this` with grouping parentheses. It enumerates **own and inherited ordinary slot names** in delegation order, with nearest-name shadowing and no distinction between method-valued and data-valued slots. It filters against text on both sides of the cursor and projects candidates into JLine without guest evaluation. Computed/chained receivers, imports, foreign inspection and mutation targets remain excluded.

The admitted observation reads at most one existing PRESENT lexical receiver binding, never invokes a getter, Closure, selector or import, and does not retain guest object references in candidates. It fails closed for unavailable Process/Actor admission, unsupported representations, unsafe parser positions and exceeded budgets. The published resource limits are **64 delegation nodes, 1,024 examined slot names and 256 presented candidates**. Runtime admission combines the persistent RootActor gate, its active-direct-entry observation barrier and an Actor-local scheduler permit. The JLine adapter preserves D227 lexical completion and noninteractive behavior.

## Validation and acceptance provenance

These results are **reported by the maintainer**, not execution performed by the documenting agent:

| Gate | Result | Provenance |
| --- | --- | --- |
| `git diff --check` | PASS / clean | Maintainer report after product push |
| Focal I134 JUnit suites | PASS | Maintainer report before full-suite run |
| `make test` | PASS | Maintainer report on local product checkout |
| `make check` | PASS | Maintainer report on local product checkout |
| Interactive JVM REPL selector TAB | PASS | Maintainer confirmed functional behavior, including delegated completion |
| Interactive Native Image REPL selector TAB | PASS | Maintainer confirmed the same behavior with Native |

The document records human-reported validation, not independently reproducible logs, CI run identifiers or a newly executed Native build by the documentation agent. The exact product SHA, changed paths, version and changelog are independently verified through GitHub. In particular, a previous GitHub read returned the pre-publication `d5554589` snapshot; a subsequent live ref read confirmed `02e32411ac614ae69b974757430d8a29d291619d` on `main`.

## Safety scope and deferred boundaries

The implementation's admission proof is limited to the actual hosted Process/RuntimeHost path used by the interactive REPL. An independently constructed scheduler cannot certify ownership or exclude turns running on a different scheduler merely by issuing a provisional observation permit for the same Actor. This cross-scheduler hypothetical was identified in review; **no generic multi-scheduler exclusivity guarantee is asserted by I134 evidence**. The ordinary REPL path obtains its scheduler through its owning RuntimeHost. This record does not authorize a global Actor registry or a change to Actor scheduling semantics.

Completion intentionally does not support `context`, `super`, computed/chained/literal/indexed receiver values, foreign/interop introspection, module imports during TAB, method callability inference, remote Actors or a general LSP completion provider. No normative Protos specification revision is claimed.

## Closure identity

```text
WORK_ITEM=I134
DECISION=D228-A1
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=02e32411ac614ae69b974757430d8a29d291619d
PRODUCT_VERSION=0.3.428-SNAPSHOT
PUBLISHED_CHANGED_PATHS=24
PRODUCT_SPEC_CHANGED=NO
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
FOCAL_TESTS=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
JVM_INTERACTIVE_TAB=PASS_MAINTAINER_REPORTED
NATIVE_INTERACTIVE_TAB=PASS_MAINTAINER_REPORTED
DOCS_PUBLICATION=THIS_REVISION_BOUND_RECORD
```

This revision-coupled evidence supports formal closure of [I134 #1002](https://github.com/guillermomolina/protos/issues/1002) once the exact documentation publication revision is verified and cited by the final Issue closure comment.
