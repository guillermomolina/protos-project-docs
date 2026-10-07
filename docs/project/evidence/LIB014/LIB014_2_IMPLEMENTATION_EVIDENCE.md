# LIB014-2 — implementation evidence

Status: **COMPLETED AND PUBLISHED**

Owning work item: `guillermomolina/protos#431` — LIB014

Decision authority: `guillermomolina/protos#809` — D187

Platform follow-up: `guillermomolina/protos#813` — PLAT051

Implementation slice: **LIB014-2 — safe matcher, captures and traversal**

Publication date: **2026-10-07**

## Published revision

```text
PUBLISHED_SHA=7675b726cd95e6491bd83f0f33babb647b2ccc39
COMMIT_MESSAGE=LIB014-2: add std:regex/Regex matching
IMPLEMENTATION_VERSION=0.3.252-SNAPSHOT
```

At evidence publication time this revision is the live `main` HEAD of
`guillermomolina/protos`.

## Human-executor validation

After publication, the project owner explicitly reported:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
```

No additional execution result is invented by this record.

## Published public surface

LIB014-2 adds these Pattern operations on top of the LIB014-1 compilation and
escaping surface:

```text
Pattern.fullMatch(text)
Pattern.search(text)
Pattern.searchFrom(text, scalarOffset)
Pattern.eachMatch(text, block)
Pattern.findAll(text)
```

No-match answers `null`. Public offsets are Protos Unicode-scalar indexes.
Invalid text/offset inputs use ordinary Error signaling.

`replaceFirst`, `replaceAll`, and `split` remain intentionally absent for
LIB014-3.

## Matcher architecture

The implementation is a Protos-owned **prioritized Pike VM** operating directly
on the private postfix compiled node table introduced by LIB014-1.

It does not:

- recursively backtrack;
- delegate matching to `java.util.regex` or another host regex engine;
- generate executable regex code;
- expand counted repetitions into repeated instruction copies.

Each thread state tracks a private node phase. Parent/child navigation metadata
is derived once per Pattern and frozen. Mutable matching state remains local to
one invocation.

The epsilon closure is iterative, using an explicit priority-preserving DFS
stack rather than host recursion. At one input position, the first thread
reaching an equivalent future execution state wins; lower-priority duplicates
are discarded while retaining the winner's capture state.

Search start candidates are seeded behind already-live earlier candidates, so a
single `search` does one prioritized scan instead of restarting an O(m*n)
matcher independently at every possible start.

## Priority semantics

Published behavior follows D187:

```text
MATCH_SELECTION=LEFTMOST_FIRST
ALTERNATIVES=ORDERED
GREEDY_QUANTIFIERS=YES
LAZY_QUANTIFIERS=YES
```

Earliest start wins first; ordered alternative and greedy/lazy priority then
select among paths from that start.

The final nullable-repeat correction is included in the published revision:
a greedy repetition may select a first zero-width iteration and preserve its
capture effects, while equivalent subsequent zero-width iteration at the same
position is prevented from looping indefinitely. Lazy priority may exit before
that participation when its exit path has priority.

Retained capture tests distinguish greedy/lazy nullable forms including
`(a?)*`, `(a?)+`, `(a?)*?`, `(a?)+?`, bounded optional repetition and behavior
after consuming input.

## Repetition and termination

Counted repetitions retain private counters instead of IR expansion.
Zero-width progress markers prevent epsilon/repetition loops from re-entering an
equivalent iteration indefinitely at the same input position.

A repeat whose body cannot consume may fast-forward required empty iterations,
so constructs such as:

```text
(?:){1000000000}
```

do not execute one billion explicit iterations.

A known non-blocking performance residual remains for huge minimum counts whose
body is nullable but can consume, for example:

```text
(?:a?){1000000000}
```

The current representation may still perform work proportional to that effective
repeat count. This is not catastrophic backtracking and does not invalidate the
D187 asymptotic contract because the effective pattern size includes the repeat
work. Any later optimization belongs to measured optional acceleration work and
must preserve capture/priority semantics.

## Captures and Match values

The published matcher implements:

```text
GROUP_0=WHOLE_MATCH
NUMBERED_CAPTURES=OPENING_PARENTHESIS_ORDER
NAMED_CAPTURES=ALSO_NUMBERED
NONPARTICIPATING_CAPTURE=NULL
EMPTY_PARTICIPATING_CAPTURE=EMPTY_STRING_WITH_EQUAL_OFFSETS
REPEATED_CAPTURE=LAST_SELECTED_PARTICIPATION
```

Capture state is copied only at capture transitions rather than on every scalar
transition. Match snapshots remain frozen; `groups()` and `namedGroups()` return
fresh snapshots.

Captured text is reconstructed from Unicode scalars without normalization.
Public start/end positions are half-open scalar offsets.

## Assertions and Unicode behavior

The matcher implements the D187 zero-width assertions:

```text
^  $
\A \z
\b \B
```

Multiline `^`/`$` use LF, CR, NEL, U+2028 and U+2029, with CRLF treated as one
line-boundary sequence and no artificial boundary between CR and LF.

Word boundaries use the D187 Unicode 17 word set prepared by LIB014-1.

## Traversal and zero-width progression

`eachMatch` and `findAll` use the ratified non-overlapping progression:

1. emit the selected match;
2. consuming match `[s,e)` resumes from `e`;
3. zero-width `[k,k)` is emitted once;
4. the next search begins at the next Unicode scalar boundary;
5. a zero-width match at end-of-input is emitted once and terminates.

All mutable thread lists, counters, captures and temporary buffers are local to
the operation. No public mutable matcher cursor, `lastIndex`, global match state,
or correctness-critical global lock/cache is introduced.

Task-sharing evidence verifies that the same frozen Pattern can be used by
concurrent Tasks without cross-contaminating matching state.

## Complexity / ReDoS evidence

The implementation preserves the selected safe-baseline architecture. Retained
adversarial tests cover classic recursive-backtracker explosions such as:

```text
(a+)+b
(a|aa)*b
(a?)*b
(?:a*)*b
```

on long no-match input, together with ambiguous counted repetition, hundreds of
alternatives, large counted repetition, dense Unicode input and many captures.

No public engine-step counter or wall-clock timeout was introduced.

The single-search implementation remains one prioritized scan consistent with
the D187 O(m*n) search contract; repeated traversal may use the separately
ratified polynomial/O(m*n^2) envelope.

## Actor/Process portability gate

LIB014-2 exposed one existing runtime limitation that was deliberately **not**
misclassified as a matching failure and was not silently patched with a Regex
special case.

Current Pattern and Match values contain Closure slots. Current
`ProtosActorValueTransfer` explicitly rejects `ProtosClosureValue`, and ordinary
snapshot formation traverses object/delegation graphs. Therefore the current
source-backed Standard Library representation cannot yet satisfy D187's desired
portable Pattern/Match rematerialization across Actor/Process boundaries.

This is now tracked by:

```text
PLAT051=#813
TITLE=Standard Library semantic-value Actor transfer and rematerialization boundary
TYPE=INVESTIGATION
LIB014_3_BLOCKED=NO
LIB014_FINAL_PORTABILITY_CLOSURE_BLOCKED=YES
```

PLAT051 owns a generic runtime/Standard-Library reconstruction boundary. It must
not make arbitrary Closures transferable and must not introduce a Regex-specific
exception merely for convenience.

No test was added that would freeze current Regex non-transferability as desired
semantics.

## Published test corpus

LIB014-2 adds/updates retained regex coverage for:

```text
matching.protos
captures.protos
assertions.protos
repetition.protos
traversal.protos
complexity.protos
sharing.protos
pattern-model.protos
```

The repository Test Tool corpus plan and its Java structure test are updated to
include the expanded regex corpus.

## Scope boundary

LIB014-2 intentionally does not implement:

```text
Pattern.replaceFirst
Pattern.replaceAll
Pattern.split
Core pattern.match(subject) integration
rich/backtracking regex extensions
DFA/JIT acceleration
Actor-transfer runtime reconstruction
```

`Regex.protos` is now materially larger than the repository's preferred source
size guideline. Splitting/refactoring it is retained as non-blocking maintenance
debt and is not mixed into the semantic matcher publication.

## Result

```text
LIB014_2_IMPLEMENTATION=PUBLISHED
PUBLISHED_SHA=7675b726cd95e6491bd83f0f33babb647b2ccc39
IMPLEMENTATION_VERSION=0.3.252-SNAPSHOT

GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
ALL_LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

D187_MATCHING_CONTRACT=PASS
TASK_SHARING=PASS
ACTOR_PROCESS_PORTABILITY=BLOCKED_BY_PLAT051

NEXT_LIB014_SLICE=LIB014-3
NEXT_LIB014_SLICE_TYPE=IMPLEMENTATION
NEXT_LIB014_SLICE_REPOSITORY=guillermomolina/protos

LIB014_4=OPTIONAL_ONLY_IF_MEASURED_ACCELERATION_IS_LATER_JUSTIFIED
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the published LIB014-2 revision, the ratified D187 contract, the current
runtime transfer implementation, and the human executor's explicit validation
report. No independent execution of the reported local tests is claimed by this
record.
