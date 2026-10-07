# TEST009-K — compiler expansion debt batch

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger/consumer: `PERF030 / guillermomolina/protos#784`

## Exact publication

~~~text
PROTOS_REVISION=3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9
PARENT_REVISION=04189acc0021ba3514e9937113efdb98b176b93e
COMMIT_SUBJECT=TEST009-K: cut remaining compiler expansion debt
IMPLEMENTATION_VERSION=0.3.213-SNAPSHOT
~~~

Maintainer-reported publication state:

~~~text
GIT_DIFF_CHECK=PASS
LOCAL_TESTS=PASS
PRODUCT_PUSH=COMPLETE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Batch scope

TEST009-K intentionally accumulated four independently motivated compilerability
repairs before paying for one expensive Truffle diagnosis.

### K1 — TextReader queue handoff

`ProtosTextReader` now advances queued requests through a non-reentrant iterative
`pump()` loop. `finishQueueRequest` only releases the active queue position
instead of recursively re-entering `pump`.

The previous causal graph:

~~~text
advanceUntilInputOrTerminal
  -> failAndFinish
  -> finishQueueRequest
  -> pump
  -> advanceUntilInputOrTerminal
~~~

was the source of the prior `TooDeepInlining` expansion in which the four
TextReader methods repeated approximately 243 times.

A regression test covers ordered handoff of many queued reads across a permanent
`LineTooLong` failure and verifies that the Java stack depth does not grow
across the queued handoff.

### K2 — residual lexical write fallback

The residual String-keyed bare-write destination walk is now a narrow host
boundary, matching the already bounded residual read fallback. Statically
proven frame-backed lexical paths remain inline.

### K3 — Polyglot entered-Context probe

`ProtosLanguageContext.currentIfEnteredForRuntime()` is now the explicit host
boundary for callers without a Truffle Node. Node-owned hot paths continue to
use `ContextReference` directly.

### K4 — Context-local Bytecode plan cache hit

The host `ConcurrentHashMap` lookup used to reuse an existing Context-local
Bytecode execution plan is isolated behind a narrow boundary. Projection
selection and semantic ownership remain outside the host-cache helper.

## Final diagnostic evidence

The final manual diagnosis used the selected
`closure-call-and-return.protos` corpus. BUG016-B had changed the diagnostic
surface to run one fresh JVM per logical Case. With three selected Cases and
`--shard-workers 1`, the complete run was therefore serial across three JVMs:

~~~text
DIAGNOSTIC_DISCOVERY=PASS
CASES=3
DIAGNOSTIC_SHARD_WORKERS=1

DIAGNOSTIC_ACQUISITION=COMPLETE
DIAGNOSTIC_SHARDS_PASS=0
DIAGNOSTIC_SHARDS_FAIL=3
DIAGNOSTIC_SHARDS_ERROR=0
DIAGNOSTIC_SHARDS_TIMEOUT=0

DIAGNOSTIC_COMPILATIONS_DONE=642
DIAGNOSTIC_COMPILATION_FAILURES=75
DIAGNOSTIC_PERFORMANCE_WARNINGS=59964
DIAGNOSTIC_PE_CONSTANT_FAILURES=0
DIAGNOSTIC_OTHER_PERMANENT_FAILURES=75
DIAGNOSTIC_WALL_SECONDS=2732.1
TRUFFLE_COMPILATION_DIAGNOSE=FAIL
~~~

Each Case produced essentially the same compilerability result:

~~~text
SEMANTIC_CORPUS=PASS
CORPUS_PASSED=1
CORPUS_FAILED=0
COMPILATIONS_DONE=214
COMPILATION_FAILURES=25
PERFORMANCE_WARNINGS=19988
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=25
~~~

The aggregate 75 failures are therefore not 75 independent defect families;
they are approximately the same 25 findings repeated across three Cases.

## K-owned causal result

The post-K evidence removes all four targeted expansion families:

~~~text
OLD_TEXT_READER_RECURSIVE_CHAIN=ABSENT
ProtosTextReader.failAndFinish=0

writableContextByName=0
selectByNameOrNull=0

AbstractPolyglotImpl.getCurrentContext=0

Context-plan ConcurrentHashMap$Node.find=0
~~~

The new TextReader path appears once through the compiled stack rather than
recursively repeating the Protos queue methods. TEST009-K therefore closes its
owned causal repairs even though the global compilerability gate remains red.

~~~text
K1_TEXT_READER_ORIGINAL_RECURSION=FIXED
K2_RESIDUAL_LEXICAL_WRITE_FALLBACK=FIXED
K3_POLYGLOT_CONTEXT_EXPANSION=FIXED
K4_BYTECODE_PLAN_CACHE_EXPANSION=FIXED
TEST009_K_CAUSAL_RESULT=PASS
GLOBAL_COMPILERABILITY=RED_ON_OTHER_DEBT
~~~

## Newly exposed debt

The deeper post-K compilation reaches three later failure families per Case:

~~~text
CODE_INSTALLATION_TOO_LARGE~=23
COMPILER_OUT_OF_MEMORY_ERROR=1
NEW_TOO_DEEP_INLINING=1
~~~

The compiler OOM was:

~~~text
ProtosSemanticBytecodeRootNodeGen id=1983
java.lang.OutOfMemoryError:
Required array length 1342177280 + 1342177280 is too large
~~~

The new `TooDeepInlining` is not K1 recursion. The Protos TextReader frames
appear once, after which a JDK bounds/error-message path recursively expands:

~~~text
AwaitTextReaderSourceFuture
-> observeLowerForCPrimeRuntime
-> consumeLowerForCPrimeRuntime
-> pump
-> advanceUntilInputOrTerminal
-> ProtosTextReader.scanLine
-> StringBuilder / String bounds path
-> Preconditions
-> Formatter
-> Locale
-> BaseLocale / ReferencedKeyMap
-> ConcurrentHashMap
-> Class.getGenericInterfaces
-> SignatureParser
-> String.substring
-> String bounds path
~~~

The repeating JDK cycle is about 33 deep while the involved Protos TextReader
methods occur once. It is separate follow-up debt.

## Workflow conclusion

The 2732.1-second diagnostic produced three nearly equivalent copies of the same
compilerability findings. TEST009 follow-up must therefore amortize dynamic
diagnostics:

~~~text
accumulate multiple high-confidence causal repairs
-> run cheap focused/functional validation during implementation
-> run exactly one expensive diagnostic at the end of the batch
~~~

No intermediate diagnostic is justified for one micro-repair. The next batch
must consume the already retained K evidence first. If a dynamic diagnostic is
still needed after that batch, it should be a single deliberately selected
diagnostic execution, not one diagnostic per repair iteration.

This evidence does not redefine BUG016; it records the observed cost and the
TEST009 work-policy consequence for the remaining compilerability cleanup.

AI assistance: this durable record was drafted with ChatGPT from the exact
published product revision, maintainer-reported validation, and retained final
diagnostic output.
