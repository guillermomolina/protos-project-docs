# I097 — Published receiver-preserving Boolean `if` / `else` (0.3.345)

**Evidence date:** 2026-10-13  
**Owning implementation issue:** [I097 / #885](https://github.com/guillermomolina/protos/issues/885).  
**Ratified language decision:** [D200 / #880](https://github.com/guillermomolina/protos/issues/880); [durable D200 record](../../decisions/language/D200_BOOLEAN_ADDITIVE_CHAINED_MESSAGES.md).  
**PROTOS_REVISION:** [`6dbbbe46612e5821c3c5191cd9d5723a3987d78f`](https://github.com/guillermomolina/protos/commit/6dbbbe46612e5821c3c5191cd9d5723a3987d78f).  
**Protos previous main revision:** [`8f7e5ea5b7ad75f8052adf7ce51945539348b4a1`](https://github.com/guillermomolina/protos/commit/8f7e5ea5b7ad75f8052adf7ce51945539348b4a1) — PERF043, `0.3.344-SNAPSHOT`.  
**Published product version:** `0.3.345-SNAPSHOT` (verified published `pom.xml` / `CHANGELOG.md`).  
**Published specification revision:** `0.1.458` (verified `spec/PROTOS_SPEC_CHANGELOG.md`).  
**Evidence boundary:** maintainer-reported local tests and `git diff --check`, plus independently read product `main` commit, content/diff and version/spec metadata. The recording agent did **not** run builds, tests, benchmarks, Native Image, or validations.

## Published product facts

The maintainer reported:

> `I097: implement receiver-preserving Boolean if/else (0.3.345) pushed, el git diff check esta limpio. Todos los tests han pasado en local.`

The linked product commit is the verified `main` HEAD at the publication checkpoint. Its title is exactly `I097: implement receiver-preserving Boolean if/else (0.3.345)` and its diff changes **15 files**:

| Domain | Files / verified change |
| --- | --- |
| Boolean native/structured selector | `src/main/java/com/guillermomolina/protos/execution/ProtosStandardBooleanProtocol.java` — stable standard `if`/`else` Closure and native-body identities, two classified kinds, exact root slot installation |
| Structured result protocol | `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java` — `PreparedBooleanCall` selection, callback index 0, immediate result and selected normal result handling |
| Root surface | `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java` — two new required root slots |
| Lowerer note only | `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java` — documents the two new local Boolean kinds; no new Bytecode operations or lowerer branches in this commit |
| Java guards | `src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java`, `ProtosStandardBooleanProtocolTest.java`, `ProtosI072PhaseEStructuredSendConvergenceTest.java` |
| Protos conformance | `protos/tests/conformance/boolean/receiver-preserving-if-else.protos` (23 `Test` cases) and registration in `protos/tests/conformance/manifest.tsv` |
| Normative specification | `spec/semantics/VALUES_AND_COLLECTIONS.md` §16, `spec/PROTOS_GRAMMAR.md` §36, `spec/PROTOS_SPEC_CHANGELOG.md` |
| User documentation | `docs/guide/04-control-flow-through-protocols.md` |
| Product publication metadata | `pom.xml`, `CHANGELOG.md` |

File list above is based on the published commit patch, **not** on an unobserved worktree.

### Exact semantic result

The addition supplies **ordinary** standard `Object` selectors of arity 1, with canonical Boolean receiver identity:

| Receiver and selector | Selected callback | Exact normal result |
| --- | --- | --- |
| `true.if(block)` | invoke once, zero args | original canonical `true` |
| `false.if(block)` | skipped, no callability validation | original canonical `false` |
| `true.else(block)` | skipped, no callability validation | original canonical `true` |
| `false.else(block)` | invoke once, zero args | original canonical `false` |

In the published source, `ProtosBytecodeRootNode.PreparedBooleanCall`:

- `hasCallback()` selects `IF` on `TRUE`, `ELSE` on `FALSE`;
- `selectedIndex()` selects argument position 0 for either;
- `immediateResult()` returns the original receiver if callback unselected;
- `finishCallback(result)` discards the selected callback's normal result and returns the original receiver without Boolean-result validation.

The native bodies implement the same return policy for permissible synchronous non-Task execution. The existing `ProtosClosureInvoker` guards forbid synchronous method invocation inside a Task. Extending the **already-existing structured Boolean C-prime path** therefore preserves Task callbacks, Future suspension, continuation and ordinary exception handling; it is the correctness reason for selecting this path over a native-only implementation. This extension adds two selector classifications and switch arms, **no new Bytecode DSL operation, physical interpreter node, or independent control-flow lowering branch**. Canonical selection is guarded by stable standard Closure identity plus method home; nearer overrides and noncanonical receivers preserve ordinary dispatch/error behavior.

Unchanged, and protected by tests/spec: result-returning `ifTrue`, `ifFalse`, `ifTrueIfFalse`; `not`, `and`, `or`; mandatory `!`, `&&`, `||` ordinary-protocol lowering. The `ifTrue` direct lowerer specialization still matches only the literal name `"ifTrue"`; it is not silently borrowed for `if`. There is **no** `elseIf`, `elif`, new grammar/keyword, truthiness, hidden Task or call carrier. Many-way effect-only alternatives compose with existing lazy `or`/`||`. D201 loop-return decisions and PERF044 matcher inlining remain independent and unchanged.

### Regression surface

The newly published 23 Protos-native conformance cases include:

- all four Boolean/selector outcomes, exact identity and selected-vs-skipped invocation;
- arbitrary selected callback returns, Future non-adoption, noninvokable selected callback versus skipped no-validation, arity/defaults/rest and polymorphic `call`;
- eager argument-producing expression effects, both chaining orders, ordinary trailing Closure, repeat `else`, later-send evaluation order, `or`/`||` short circuit and absence of `elseIf`/`elif`;
- Error, non-local return, cancellation observation, Future wait/suspend within a Task with no callback replay;
- noncanonical receiver rejection, copied standard behavior, custom same-name selector shadowing, and old protocol regression.

The Java tests extend published/frozen Object-slot expectations, the audited exact native-selector surface and both canonical Boolean guarded-structured-send identities. The existing provider-count rule remains unchanged because new standard native bodies share the original `standardNative(...)` provider.

## Human validation and publication evidence

| Gate | Evidence / status |
| --- | --- |
| `git diff --check` | **PASS**, reported explicitly by the human executor (clean diff) |
| Local tests | **PASS**, human explicitly reported “Todos los tests han pasado en local”; no individual transcript/test count claimed |
| `make check`, Native Image, separate benchmarking | **NOT REPORTED**, not silently treated as PASS |
| Product commit/push | **PUBLISHED**, exact SHA verified through GitHub `main` and commit patch |
| Product version | **VERIFIED** at published revision: `0.3.345-SNAPSHOT` |
| Normative revision | **VERIFIED** at published revision: `0.1.458` |
| Additional performance gain | **NOT CLAIMED**; this is a correctness/public-protocol release, not a PERF result |
| Post-metadata tests | **NOT REQUESTED**, honoring the human-executor versioning gate |

The explicit human report plus the actual published revision satisfy the implementation work's validation and publication requirements. Do not assert tests were performed by this recording agent or that untouched old test fixtures alone prove the new contract.

## Ownership, reconciliation and closure

- **D200 / #880:** owner-ratified additive receiver-preserving `if`/`else` design. Implementation now published under I097; D200 remains a completed **decision**, not reopened for implementation.
- **I097 / #885:** bounded implementation published with spec, guide, conformance and product metadata; can close as completed after re-reading this exact project evidence revision and GitHub issue state.
- **D201 / #882:** independent loop-result decision **ratified and closed** under its own authority, with separate implementation [I098/#886](https://github.com/guillermomolina/protos/issues/886) still required. I097 neither implements nor alters the D201 contract.
- **PERF044 / #883:** separate, unresolved aggressive matching/`caseOf` physical inlining research; no outcome inferred.
- **I095 / #878 and PLAT057 / #881:** existing Boolean `ifTrue` fastpath and general Closure inlining not altered/ratified here.

```text
IDENTIFIER=I097
PROTOS_REVISION=6dbbbe46612e5821c3c5191cd9d5723a3987d78f
PRODUCT_VERSION=0.3.345-SNAPSHOT
SPEC_REVISION=0.1.458
PRODUCT_PUBLICATION=VERIFIED_MAIN
TESTS_EXECUTOR=HUMAN
HUMAN_REPORTED_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=PASS
MAKE_CHECK=NOT_REPORTED
NATIVE_IMAGE=NOT_REPORTED
TESTS_RUN_BY_RECORDING_AGENT=NO
D200_CONTRACT=RATIFIED_ADDITIVE_IF_ELSE_NO_ELSEIF
D201_AND_PERF044=UNTOUCHED
```

**Cross-references:** Product commit links the authoritative `I097` work item by ID and includes D200 normative specification; this evidence links the exact product SHA and owning Issues. The I097 issue closure must link the **exact commit of `guillermomolina/protos-project-docs` that publishes this file**, not a moving `main`.
