# I054 publication evidence

I054 / `guillermomolina/protos#654` implements the bounded removal approved by
AUD009-F2 / #653: retire the retained I026-G generic GraalVM dynamic-LSP test
obligation while preserving the selected static Protos language server,
debugger/DAP, Truffle instrumentation, LM009 behavior, and the LM010 growth path.

## Exact product publication

```text
PROTOS_REVISION=9bed87430f99f3df8c2c3112f5a51b429a6e3176
PROTOS_VERSION=0.3.218-SNAPSHOT
COMMIT=I054: retire generic GraalVM dynamic-LSP test evidence (#654)
ISSUE=#654
PARENT_DECLARATION=#653
```

At that exact Protos revision:

- the test-scope `org.graalvm.polyglot:lsp` POM dependency is removed;
- `ProtosI026GLspTransportTest.java` (I026-G1) is removed;
- `ProtosI026GLspCapabilityTest.java` (I026-G2) is removed;
- both GraalVM LSP tests are removed from the serialized Java-test lane in the
  `Makefile`;
- the DAP runtime dependency and DAP test surfaces are untouched;
- the dedicated static `ProtosLanguageServer` / LSP4J surface is untouched;
- Truffle Source/SourceSection and StatementTag/CallTag instrumentation are not
  removed; and
- historical I026-G evidence remains in Git/GitHub history.

The pre-implementation reconciliation established that the I026-G2 capability
test belonged to the exact I026-G generic dynamic-LSP obligation classified
`REMOVE_NOW_RECONSIDER_LATER` by AUD009-F2. Its removal therefore does not
constitute a new design decision.

## Validation reported by the maintainer

For the published product candidate the maintainer reports:

```text
GIT_DIFF_CHECK=PASS
LOCAL_REQUIRED_TESTS=PASS
PRODUCT_PUSH=PASS
```

GitHub currently exposes no exact-SHA status checks or workflow runs for
`9bed87430f99f3df8c2c3112f5a51b429a6e3176`; this record therefore does not
invent CI evidence.

## Publication-policy reconciliation

The exact published commit changes:

```text
Makefile
pom.xml
src/test/java/com/guillermomolina/protos/execution/ProtosI026GLspCapabilityTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosI026GLspTransportTest.java
```

It does **not** change the root `CHANGELOG.md`, and the implementation version
remains `0.3.218-SNAPSHOT`, which was already the version published by the
immediately preceding I056 change.

Current `AGENTS.work/IMPLEMENTATION.md` requires the implementation version bump
and matching changelog entry for an implementation publication, and states that
a version bump must not be omitted merely because a change is incremental or
internal.

Therefore the implementation result itself is published and validated, but the
I054 closure transaction is not yet complete:

```text
DYNAMIC_LSP_TEST_DEPENDENCY=REMOVED
I026_G1_TRANSPORT_TEST=REMOVED
I026_G2_CAPABILITY_TEST=REMOVED
STATIC_PROTOS_LSP=PRESERVED
DAP_DEBUGGER=PRESERVED
TRUFFLE_INSTRUMENTATION=PRESERVED
LM009=PRESERVED
LM010_GROWTH_PATH=PRESERVED
SPECIFICATION_CHANGE=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO

VERSION_INCREMENT=ABSENT
CHANGELOG_ENTRY=ABSENT
CLOSURE_STATUS=BLOCKED_ON_PUBLICATION_METADATA_RECONCILIATION
```

Because published history must not be rewritten, the repair must be a bounded
follow-up metadata reconciliation on current `guillermomolina/protos` HEAD,
deriving the then-current next implementation version and adding the matching
I054 changelog entry without altering the already-published substantive removal.

This record is non-normative evidence. It does not reopen AUD009-F2, I026-G,
PLAT024, LM009, or LM010.
