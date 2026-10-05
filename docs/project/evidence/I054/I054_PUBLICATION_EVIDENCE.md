# I054 closure evidence

I054 / `guillermomolina/protos#654` implements the bounded removal approved by
AUD009-F2 / #653: retire the retained I026-G generic GraalVM dynamic-LSP test
obligation while preserving the selected static Protos language server,
debugger/DAP, Truffle instrumentation, LM009 behavior, and the LM010 growth path.

## Exact product publication

I054 closed through a forward-only two-commit publication because the original
substantive commit omitted the mandatory implementation-version/changelog
metadata.

```text
I054_IMPLEMENTATION_REVISION=9bed87430f99f3df8c2c3112f5a51b429a6e3176
I054_IMPLEMENTATION_SUBJECT=I054: retire generic GraalVM dynamic-LSP test evidence (#654)

I054_METADATA_REVISION=00526bdeffaec4360f2ed2deed3d59e16ae3a431
I054_METADATA_SUBJECT=I054: reconcile publication metadata
FINAL_IMPLEMENTATION_VERSION=0.3.219-SNAPSHOT

ISSUE=#654
NATIVE_PARENT=#653
```

The substantive revision removes:

- the test-scope `org.graalvm.polyglot:lsp` POM dependency;
- `ProtosI026GLspTransportTest.java` (I026-G1);
- `ProtosI026GLspCapabilityTest.java` (I026-G2); and
- their now-dead serialized Java-test wiring in the `Makefile`.

It preserves:

- the `org.graalvm.polyglot:dap` runtime dependency and DAP/debugger surfaces;
- `protos debug` and `PROTOS_DEBUG_READY`;
- the dedicated static `ProtosLanguageServer` / LSP4J implementation;
- `ProtosStaticAnalysisCore`, `ProtosStaticAnalysisSession`,
  ProjectBinding and `protos.project`;
- Truffle Source/SourceSection and StatementTag/CallTag instrumentation;
- LM009 product behavior; and
- the LM010 hover/completion/signature-help growth path.

Historical I026-G evidence remains valid in Git/GitHub history.

The pre-implementation reconciliation established that
`ProtosI026GLspCapabilityTest` belonged to the exact I026-G generic dynamic-LSP
obligation classified `REMOVE_NOW_RECONSIDER_LATER` by AUD009-F2. Its removal
therefore does not constitute a new design decision.

## Forward-only metadata reconciliation

The original I054 implementation revision retained `0.3.218-SNAPSHOT` and did
not add a root `CHANGELOG.md` entry. Current implementation policy requires
both for an implementation publication.

Published history was not rewritten. Revision
`00526bdeffaec4360f2ed2deed3d59e16ae3a431` completes the missing metadata
forward-only and changes only:

```text
pom.xml
CHANGELOG.md
```

It bumps the implementation version:

```text
0.3.218-SNAPSHOT -> 0.3.219-SNAPSHOT
```

and adds an I054 changelog entry explicitly identifying
`9bed87430f99f3df8c2c3112f5a51b429a6e3176` as the already-published
substantive implementation.

The exact remote patch contains no added-line trailing-whitespace or
space-before-tab violations. No executable/product source or test file is
changed by the reconciliation commit.

## Validation and closure gates

The maintainer reported the following for the substantive I054 implementation:

```text
GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
PRODUCT_PUSH=PASS
```

The metadata reconciliation is non-executable and preserves those already-green
behavioral candidate bytes. Its exact published delta was independently checked
through GitHub and contains only the required `pom.xml` and `CHANGELOG.md`
changes.

GitHub currently exposes no exact-SHA status checks or workflow runs for either
I054 publication revision, so this record does not invent CI evidence.

The GitHub native-parent endpoint for #654 resolves to AUD009-F2 / #653.

Final closure classification:

```text
DYNAMIC_LSP_TEST_DEPENDENCY=REMOVED
I026_G1_TRANSPORT_TEST=REMOVED
I026_G2_CAPABILITY_TEST=REMOVED
STATIC_PROTOS_LSP=PRESERVED
DAP_DEBUGGER=PRESERVED
TRUFFLE_INSTRUMENTATION=PRESERVED
LM009=PRESERVED
LM010_GROWTH_PATH=PRESERVED

VERSION_INCREMENT=PASS
CHANGELOG_ENTRY=PASS
METADATA_RECONCILIATION=PASS
NATIVE_PARENT=PASS
SPECIFICATION_CHANGE=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
NEXT_TECHNICAL_SLICE=NONE
CLOSURE_STATUS=COMPLETE
```

This record is non-normative closure evidence. It does not reopen AUD009-F2,
I026-G, PLAT024, LM009, or LM010.
