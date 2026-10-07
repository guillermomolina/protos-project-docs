# LIB014-3 — implementation evidence

Status: **COMPLETED AND PUBLISHED**

Owning work item: `guillermomolina/protos#431` — LIB014

Decision authority: `guillermomolina/protos#809` — D187

Platform closure gate: `guillermomolina/protos#813` — PLAT051

Implementation slice: **LIB014-3 — replacement, split, public integration and baseline completion**

Publication date: **2026-10-07**

## Published revision

```text
PUBLISHED_SHA=c3b581ac424db1283ac27d3af1d6bbd3a0c92a00
COMMIT_MESSAGE=LIB014-3: add Regex replacement and split
IMPLEMENTATION_VERSION=0.3.254-SNAPSHOT
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

## Public baseline completed by LIB014-3

The published `std:regex/Regex` Pattern surface now includes:

```text
Pattern.replaceFirst(text, replacement)
Pattern.replaceAll(text, replacement)
Pattern.split(text)
```

Together with LIB014-1 and LIB014-2, this completes the owner-ratified D187
baseline public API. LIB014-3 does not add Regex convenience methods to String,
a mutable Matcher cursor, callbacks, streaming, rich/backtracking constructs,
or Core `pattern.match(subject)` integration.

## Replacement grammar and validation

Replacement Strings accept only the D187 grammar:

```text
$$       literal $
${0}     whole match
${N}     numbered capture
${name}  named capture
```

The implementation parses the replacement template before matching begins and
validates every referenced capture against Pattern metadata before any output is
produced. Therefore malformed `$` syntax or a nonexistent numbered/named group
signals ordinary Error even when the subject has no match.

A valid capture that did not participate expands to the empty String.

`Regex.escapeReplacement(text)` remains the existing escaping authority: each
literal `$` is doubled so the escaped String can be inserted literally by the
replacement operations.

## Replacement execution

`replaceFirst` replaces exactly the match selected by `search` and returns text
with unchanged content when no match exists.

`replaceAll` consumes the same non-overlapping traversal authority as
`eachMatch` and `findAll`.

The implementation keeps the search progression independent from the input copy
boundary. After a zero-width match at scalar position `k`, the next search may
advance beyond `k`, while the original scalar skipped only for search progress
remains part of the copied output.

The retained contract example is therefore:

```text
Regex.compile("").replaceAll("ab", "-") == "-a-b-"
```

Output is accumulated in an invocation-local UTF-8 byte buffer and subject
slices are appended using Unicode scalar offsets without normalization.

## Split semantics

`Pattern.split(text)` uses the same non-overlapping traversal authority.

Published behavior:

```text
DELIMITER_MATCHES_IN_OUTPUT=NO
DELIMITER_CAPTURES_IN_OUTPUT=NO
LEADING_EMPTY_FIELDS=PRESERVED
INTERIOR_EMPTY_FIELDS=PRESERVED
TRAILING_EMPTY_FIELDS=PRESERVED
ZERO_WIDTH_COMMON_PROGRESS_RULE=YES
```

The field copy boundary remains separate from the search position, so zero-width
progress does not discard subject scalars.

Retained contract examples include:

```text
Regex.compile(",").split("a,,b,") == ["a", "", "b", ""]
Regex.compile("").split("ab") == ["", "a", "b", ""]
```

Delimiter captures are intentionally not injected into the result Array.

## Shared traversal authority

LIB014-3 refactors the private traversal helper so `eachMatch`, `findAll`,
`replaceAll`, and `split` consume the same search-progression semantics rather
than maintaining independent zero-width algorithms.

The D187 rule remains:

1. emit the selected match;
2. after a consuming match, continue from its end;
3. after zero-width `[k,k)`, emit it once and continue from the next Unicode
   scalar boundary;
4. emit an end-of-input zero-width match once and then stop.

## Unicode and output construction

The existing per-operation decoded subject representation now exposes private
span append support. Public offsets remain Unicode-scalar indexes; output
construction uses the exact scalar spans selected by the matcher and performs no
implicit normalization.

No JVM UTF-16 indexing contract is exposed.

## Public/private surface

The Pattern public API is extended only with:

```text
replaceFirst
replaceAll
split
```

Private parser/program/thread/traversal/template machinery remains hidden.

No public host matcher, program nodes, engine cursor, `lastIndex`, replacement
AST, or traversal state is exposed.

## Tests and corpus

LIB014-3 adds dedicated retained corpus files:

```text
protos/tests/library/regex/replacement.protos
protos/tests/library/regex/split.protos
```

The repository corpus plan and structural membership checks are updated to
include them.

The published tests cover replacement grammar and escaping, numbered/named/group
0 expansion, nonparticipating captures, malformed/missing references, no-match
prevalidation, zero-width replacement progression, split empty-field retention,
zero-width split, delimiter capture omission, invalid inputs, Unicode/supplementary
scalar behavior, and public Pattern privacy.

All existing LIB014-1/LIB014-2 corpus remains part of the maintainer-reported
passing local suite.

## Architecture and safety boundary

LIB014-3 continues to use the Protos-owned prioritized Pike matcher from
LIB014-2.

It introduces no `java.util.regex`, PCRE, ICU-regex, RE2-native, JNI, native
regex dependency, recursive backtracking engine, public step counter, wall-clock
timeout, or correctness-critical global mutable cache.

D187 semantics are unchanged and no specification change is introduced.

## Actor/Process portability remains a platform gate

The public functional Regex baseline is complete, but final LIB014 closure
cannot yet claim D187 Actor/Process portability complete.

That pre-existing runtime limitation is owned by:

```text
PLAT051=#813
TITLE=Standard Library semantic-value Actor transfer and rematerialization boundary
STATUS=READY_FOR_INVESTIGATION
```

Current source-backed Pattern/Match values contain Closure slots and the existing
Actor transfer machinery rejects ordinary Closure transfer. LIB014-3 does not
special-case Regex, does not make arbitrary Closures transferable, and does not
freeze the current non-transferability as desired semantics.

Therefore:

```text
LIB014_3=COMPLETE
REGEX_BASELINE_FUNCTIONAL_API=COMPLETE
LIB014_FINAL_CLOSURE=BLOCKED_BY_PLAT051
```

## Optional LIB014-4 acceleration

D187 allows an optional measured acceleration slice only when evidence justifies
it.

LIB014-3 provides no new benchmark or production evidence requiring such a
slice. No performance defect is established by this publication.

Therefore:

```text
LIB014_4_STATUS=NOT_OPENED
LIB014_4_REQUIRED=NO
LIB014_4_TRIGGER=FUTURE_MEASURED_EVIDENCE_ONLY
```

## Result

```text
LIB014_3_IMPLEMENTATION=PUBLISHED
PUBLISHED_SHA=c3b581ac424db1283ac27d3af1d6bbd3a0c92a00
IMPLEMENTATION_VERSION=0.3.254-SNAPSHOT

GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
ALL_LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

D187_FUNCTIONAL_BASELINE=IMPLEMENTED
REPLACEMENT=PASS
SPLIT=PASS
ZERO_WIDTH_COMMON_PROGRESS=PASS
HOST_REGEX_ENGINE=ABSENT
CORE_PATTERN_MATCH_INTEGRATION=ABSENT

LIB014_NEXT_FUNCTIONAL_SLICE=NONE
LIB014_4=OPTIONAL_NOT_JUSTIFIED
LIB014_FINAL_CLOSURE_BLOCKED_BY=PLAT051/#813
NEXT_ACTION=PLAT051_INVESTIGATION
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
using the published LIB014-3 revision, D187, the existing PLAT051 gate, and the
human executor's explicit validation report. No independent execution of the
reported local tests is claimed by this record.
