# TEST009-Q — post-revert diagnostic residual manifest

Date: 2026-10-06

Owning work item: `TEST009 / guillermomolina/protos#795`

Related implementation:

~~~text
PROTOS_REVISION=206bbde5593e4eea414b695047d07b894e201fbc
COMMIT_SUBJECT=TEST009-Q: revert global native-body PE specialization
PUBLISHED_VERSION=0.3.224-SNAPSHOT
~~~

This record is a compact durable manifest derived from the maintainer-supplied
post-Q `make diagnose-truffle-compilation` output. It supplements
`TEST009_Q_GLOBAL_NATIVE_BODY_PE_REVERT.md` so the next read-only investigation
can work from repository evidence rather than requiring the maintainer's local
diagnostic transcript.

## Acquisition provenance

The supplied command selected exactly one logical Case:

~~~text
CORPUS=protos/corpus/conformance
FILE=call/closure-call-and-return.protos
CASE=plain closure call
SHARD_WORKERS=1
~~~

The acquisition built the post-Q implementation code before final publication
metadata was written. Its Maven banner therefore reports
`0.3.222-SNAPSHOT`; Q's final publication metadata is
`0.3.224-SNAPSHOT`.

No claim is made that this diagnostic executed literal final commit
`206bbde5593e4eea414b695047d07b894e201fbc` after the metadata-only publication
edits.

## Aggregate result

~~~text
DIAGNOSTIC_DISCOVERY=PASS
CASES=1
DIAGNOSTIC_SHARD_WORKERS=1

SEMANTIC_CORPUS=PASS
CORPUS_PASSED=1
CORPUS_FAILED=0

COMPILATIONS_DONE=217
COMPILATION_FAILURES=23
SHUTDOWN_CASCADE_FAILURES=0
PERFORMANCE_WARNINGS=5775
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=23

CODE_INSTALLATION_TOO_LARGE=22
TOO_DEEP_INLINING=1
OTHER_PERMANENT_FAILURE_CLASS=0

DIAGNOSTIC_WALL_SECONDS=927.9
TRUFFLE_COMPILATION_DIAGNOSE=FAIL
~~~

## Permanent-failure manifest

The transcript contains the 23 permanent failures twice: once in the compact
diagnostic summary and once in the retained raw diagnostic section. The table
below deduplicates by compilation id/root identity.

| Compilation id | Root | Class | Total ms | PE ms | Compiler ms |
| ---: | --- | --- | ---: | ---: | ---: |
| 169 | `ProtosSemanticBytecodeRootNodeGen@36319294` | CodeTooLarge | 18385 | 4818 | 13567 |
| 179 | `ProtosSemanticBytecodeRootNodeGen@3ab40a91` | CodeTooLarge | 16560 | 4181 | 12379 |
| 233 | `ProtosSemanticBytecodeRootNodeGen@1944607f` | CodeTooLarge | 10177 | 1848 | 8330 |
| 248 | `ProtosSemanticBytecodeRootNodeGen@ffa1c1a` | CodeTooLarge | 8953 | 1939 | 7013 |
| 273 | `ProtosSemanticBytecodeRootNodeGen@5434ec40` | CodeTooLarge | 12912 | 4338 | 8575 |
| 321 | `ProtosSemanticBytecodeRootNodeGen@51db161c` | CodeTooLarge | 11200 | 2647 | 8553 |
| 332 | `ProtosSemanticBytecodeRootNodeGen@1332d806` | CodeTooLarge | 10870 | 1939 | 8930 |
| 333 | `ProtosSemanticBytecodeRootNodeGen@20f2a273` | CodeTooLarge | 37501 | 6992 | 30509 |
| 341 | `ProtosSemanticBytecodeRootNodeGen@d71232a` | CodeTooLarge | 24301 | 11822 | 12479 |
| 356 | `ProtosSemanticBytecodeRootNodeGen@4c0e3d18` | CodeTooLarge | 14509 | 7860 | 6649 |
| 1568 | `ProtosSemanticBytecodeRootNodeGen@53109b5a` | CodeTooLarge | 12470 | 3855 | 8616 |
| 1580 | `ProtosSemanticBytecodeRootNodeGen@468d9bed` | CodeTooLarge | 11277 | 2813 | 8465 |
| 1714 | `ProtosSemanticBytecodeRootNodeGen@6f6208b1` | CodeTooLarge | 9674 | 1578 | 8097 |
| 1795 | `ProtosSemanticBytecodeRootNodeGen@1883cc46` | CodeTooLarge | 11585 | 3057 | 8528 |
| 1852 | `ProtosBytecodeRootNodeGen@354213a1` | CodeTooLarge | 8324 | 1598 | 6726 |
| 1858 | `ProtosSemanticBytecodeRootNodeGen@f841df4` | CodeTooLarge | 14686 | 5187 | 9499 |
| 1962 | `ProtosSemanticBytecodeRootNodeGen@5ada6547` | CodeTooLarge | 13190 | 4062 | 9128 |
| 1979 | `ProtosSemanticBytecodeRootNodeGen@788e00ec` | CodeTooLarge | 14729 | 6123 | 8607 |
| 1988 | `ProtosSemanticBytecodeRootNodeGen@152fa05d` | CodeTooLarge | 11572 | 3011 | 8561 |
| 2232 | `ProtosSemanticBytecodeRootNodeGen@be58819` | CodeTooLarge | 14285 | 3544 | 10741 |
| 2252 | `ProtosSemanticBytecodeRootNodeGen@609e1341` | CodeTooLarge | 12593 | 2040 | 10552 |
| 2874 | `ProtosBytecodeRootNodeGen@77f9e5bd` | TooDeep | 563 | 563 | 0 |
| 2885 | `ProtosSemanticBytecodeRootNodeGen@3ab40a91(resume_bci=9106)` | CodeTooLarge | 8550 | 2470 | 6080 |

Root-family distribution:

~~~text
PERMANENT_FAILURES_SEMANTIC_BYTECODE=21
  CODE_TOO_LARGE=21
  TOO_DEEP=0

PERMANENT_FAILURES_BYTECODE=2
  CODE_TOO_LARGE=1
  TOO_DEEP=1
~~~

The single TooDeep record is therefore distinct from the dominant residual
code-size family and must not be silently folded into it.

## Retained TooDeep inlining trace

The raw post-Q transcript contains additional causal evidence for the single
TooDeep record, compilation id `2874`,
`ProtosBytecodeRootNodeGen@77f9e5bd`.

Immediately after the bailout, Graal prints its inlined-method frequency list.
The dominant repeated host chain appears 33 times:

~~~text
SignatureParser.parseZeroOrMoreFormalTypeParameters
 -> SignatureParser.parseClassSignature
 -> ClassRepository.parse
 -> AbstractRepository.<init>
 -> Class.getGenericInfo
 -> Class.getGenericInterfaces
 -> ConcurrentHashMap.comparableClassFor
 -> ConcurrentHashMap$TreeNode.findTreeNode
 -> ConcurrentHashMap.replaceNode
 -> ConcurrentHashMap.remove
 -> ReferencedKeyMap.removeStaleReferences
 -> ReferencedKeyMap.existingKey
 -> BaseLocale.getInstance
 -> Locale.getInstance
 -> Locale.initDefault
 -> Locale.getFormatLocale
 -> Locale.getDefault
 -> Formatter.<init>
 -> Preconditions.outOfBoundsMessage
 -> Preconditions...checkFromToIndex
 -> String.checkBoundsBeginEnd
 -> String.substring
 -> SignatureParser.remainder/error/parseFormalTypeParameters
~~~

The Protos/runtime tail printed once is:

~~~text
ProtosTextReader.scanText(ProtosEncodingValue$DecodePreview)
 -> ProtosTextReader.advanceUntilInputOrTerminal(ProtosTextReader$Request)
 -> ProtosTextReader.pump()
 -> ProtosTextReader.consumeLowerForCPrimeRuntime(ProtosIoOperation, ProtosFutureValue)
 -> ProtosTextReader.observeLowerForCPrimeRuntime(ProtosIoOperation, ProtosFutureValue)
 -> ProtosTextReaderCPrimeExecution$CallState.awaitSourceFuture(...)
 -> ProtosBytecodeRootNode$AwaitTextReaderSourceFuture.perform(...)
 -> ProtosBytecodeRootNodeGen$CachedBytecodeNode.handleAwaitTextReaderSourceFuture_(...)
 -> ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt(...)
 -> ProtosBytecodeRootNodeGen.continueAt(...)
 -> ProtosBytecodeRootNodeGen.execute(...)
 -> OptimizedCallTarget.executeRootNode(...)
 -> OptimizedCallTarget.profiledPERoot(...)
~~~

This is materially stronger localization than the aggregate TooDeep count, but
it still does not prescribe a source edit. TEST009-R must determine why this JDK
reflection/locale/formatter chain is reachable from the TextReader C-prime path
and classify the correct remedy class under the systematic procedure.

It must not assume that the presence of a host chain means
`@TruffleBoundary` is automatically correct, and it must verify whether the
relevant current source path is already covered by an independently justified
boundary or whether the expansion enters through a different responsibility.

## Performance-warning population

The transcript contains exactly 5,775 textual performance-warning records. All
are unresolved-call warnings of the form:

~~~text
Partial evaluation could not inline the virtual runtime call ...
~~~

Root-family distribution:

~~~text
SEMANTIC_BYTECODE_WARNING_RECORDS=5054
BYTECODE_WARNING_RECORDS=721
TOTAL=5775
~~~

A mechanical target-signature grouping of those 5,775 records gives:

~~~text
JAVA_COLLECTION_DISPATCH=3966
BYTECODE_ROOT_HELPER_OR_LAMBDA=931
GENERIC_NATIVE_BODY_EXECUTE=692
LEXICAL_BINDING_AUTHORITY=131
STANDARD_PROTOCOL_LAMBDA=54
IO_RELEASE_CPRIME=1
TOTAL=5775
~~~

The groups above are transcript organization only, not causal remedy
classification.

### Exact warning-target histogram

| Count | Target |
| ---: | --- |
| 1173 | `List.size()` |
| 740 | `List.get(int)` |
| 692 | `ProtosNativeClosureBody.execute(ProtosActivation, List)` |
| 480 | `ProtosBytecodeRootNode$ReadFrameLocal$$Lambda.get()` |
| 465 | `ImmutableCollections$List12.size()` |
| 463 | `ImmutableCollections$List12.get(int)` |
| 442 | `Collection.isEmpty()` |
| 442 | `Collection.toArray()` |
| 191 | `ProtosBytecodeRootNode$$Lambda.get()` |
| 129 | `ProtosBytecodeRootNode$Lookup$$Lambda.get()` |
| 117 | `ProtosLexicalBindingAuthority.prepareForContextObservation()` |
| 62 | `ProtosBytecodeRootNode$ResolveCapturedMaterializedWritableLexicalTarget$$Lambda.get()` |
| 52 | `ImmutableCollections$ListN.size()` |
| 52 | `ImmutableCollections$ListN.get(int)` |
| 43 | `Map.get(Object)` |
| 34 | `BytecodeRootNode.getBytecodeNode()` |
| 26 | `List.add(Object)` |
| 22 | `ProtosStandardNumberOrderingProtocol$$Lambda.execute(ProtosActivation, List)` |
| 19 | `ProtosStandardArrayProtocol$$Lambda.get()` |
| 14 | `ProtosLexicalBindingAuthority.isEmpty()` |
| 13 | `Map.computeIfAbsent(Object, Function)` |
| 13 | `ProtosBytecodeRootNode$ReadMember$$Lambda.get()` |
| 8 | `List.indexOf(Object)` |
| 8 | `List.contains(Object)` |
| 8 | `List.remove(int)` |
| 8 | `List.remove(Object)` |
| 8 | `List.isEmpty()` |
| 8 | `Map.remove(Object)` |
| 8 | `ProtosStandardNumberEqualityProtocol$$Lambda.execute(ProtosActivation, List)` |
| 5 | `ProtosStandardBytesProtocol$$Lambda.get()` |
| 4 | `ProtosBytecodeRootNode$PreparedLocalEachCall.hasNext()` |
| 4 | `ProtosBytecodeRootNode$PreparedLocalEachCall.prepareInlineCurrent()` |
| 4 | `ProtosBytecodeRootNode$PreparedLocalEachCall.finish()` |
| 4 | `ProtosBytecodeRootNode$PreparedLocalEachCall.admitsInlineLiteralChild(...)` |
| 4 | `ProtosBytecodeRootNode$PreparedLocalEachCall.advance()` |
| 2 | `List.subList(int, int)` |
| 2 | `ProtosBytecodeRootNode$PreparedClosureCall.taskForRuntime()` |
| 2 | `ImmutableCollections$ListN.isEmpty()` |
| 2 | `ImmutableCollections$ListN.toArray()` |
| 1 | `Set.contains(Object)` |
| 1 | `ProtosIoReleaseCPrimeExecution$Sequence$$Lambda.get()` |

Runtime-generated lambda suffixes/addresses are intentionally omitted from the
durable target names above. They are process-local identities and are not stable
regression identifiers.

## Comparison authority

Retained TEST009-O versus post-Q:

| Metric | TEST009-O | Post-Q |
| --- | ---: | ---: |
| Compilations done | 211 | 217 |
| Compilation failures | 40 | 23 |
| Performance warnings | 5758 | 5775 |
| PE-constant failures | 0 | 0 |
| Code installation too large | 30 | 22 |
| TooDeep inlining | 8 | 1 |
| >100-second permanent failures | 2 | 0 |
| Diagnostic wall seconds | 1500.6 | 927.9 |

This evidence is compatible with TEST009-P's conclusion that removing M5
materially retracts hard compiler-expansion debt. It does not identify the
correct remedy for any residual family.

## Current-head reconciliation

At the time this residual manifest was prepared, current
`guillermomolina/protos` had advanced one commit beyond Q:

~~~text
CURRENT_PROTOS_HEAD=bb98d68b043b0af81385fbfbaa44d682e29244d6
CURRENT_PROTOS_VERSION=0.3.225-SNAPSHOT
CONCURRENT_COMMIT=TEST002-A3: migrate TOML semantic tests to Protos
~~~

The exact Q-to-current-HEAD comparison contains only:

- `CHANGELOG.md` and `pom.xml` metadata;
- TOML conformance-test additions / manifest entries;
- removal or reduction of corresponding Java TOML tests.

There are no `src/main/java` changes in that delta. Therefore this concurrent
advance does not itself change the product runtime architecture whose residual
compilerability is classified by TEST009-R. R must nevertheless re-read live
HEAD before drawing any current-state conclusion.

## Investigation boundary for TEST009-R

This manifest intentionally does **not** select a source repair.

The established systematic rule remains:

~~~text
DIAGNOSTIC_FAILURE != AUTOMATIC_BOUNDARY_PRESCRIPTION
BOUNDARY_CHASING_ALLOWED=NO
CODE_TOO_LARGE_MICRO_REPAIR_BATCH_ALLOWED=NO
~~~

TEST009-R must determine, from repository/web evidence and without running a new
diagnostic, whether the residual families are:

- one or more identifiable structural expansion mechanisms;
- ordinary generic/native dispatch warnings that are expected consequences of
  the restored generic frontier;
- valid-but-too-large PE graphs requiring structural/inlining design;
- an independently classifiable recursive-inlining path;
- or insufficiently localized by the retained textual evidence.

Where the evidence is insufficient, R must say so rather than infer a source
edit from warning frequency.

~~~text
TEST009_Q_RESIDUAL_MANIFEST=COMPLETE
TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-R
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SCOPE=POST_Q_RESIDUAL_COMPILERABILITY_CLASSIFICATION
IMPLEMENTATION_AUTHORIZED=NO
EXPENSIVE_DIAGNOSTIC_RERUN_REQUIRED=NO
~~~

AI assistance: this record was drafted with ChatGPT by mechanically classifying
the maintainer-supplied TEST009-Q diagnostic transcript and reconciling it with
the exact Q publication and current GitHub HEAD.
