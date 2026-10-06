# LIB014-1 — implementation evidence

Status: **COMPLETED AND PUBLISHED**

Owning work item: `guillermomolina/protos#431` — LIB014

Decision authority: `guillermomolina/protos#809` — D187

Implementation slice: **LIB014-1 — parser, exact profile and immutable Pattern/Match model**

Publication date: **2026-10-06**

## Published revision

```text
PUBLISHED_SHA=0abd607a3a354ecd2b6551e344661d4b727d7910
COMMIT_MESSAGE=LIB014-1: add std:regex/Regex compilation and escaping
IMPLEMENTATION_VERSION=0.3.245-SNAPSHOT
```

At evidence publication time this revision was also the live `main` HEAD of `guillermomolina/protos`.

## User-provided validation evidence

After publication, the project owner explicitly reported:

```text
git diff --check = clean
all local tests = passed
```

This record preserves that execution evidence as supplied by the human executor. It does not invent an additional CI result.

## Published implementation

The slice adds the initial `std:regex/Regex` compilation surface under the D187 contract.

Public operations now provided:

```text
Regex.compile(source)
Regex.compileWithFlags(source, flags)
Regex.escape(text)
Regex.escapeReplacement(text)
```

The Pattern surface exposes:

```text
source
flags
captureCount
captureNames
```

Independent compilations remain distinct frozen Pattern objects.

Matching execution operations are deliberately not yet published.

## Parser / compiled representation

`protos/lib/regex/Regex.protos` owns the parser and accepted dialect.

Its private compiled representation is a **postfix frozen node table** whose nodes reference earlier child indexes. The node families are private implementation machinery and include:

```text
set
empty
assert
concat
alternate
capture
repeat
```

Counted repetition is retained structurally rather than expanded. The parser uses explicit frame/state structures rather than host recursion so deeply nested source does not consume the host call stack.

The private compiled program retains the information required by LIB014-2, including capture metadata, matching flags and assertion information, while remaining inaccessible through public Pattern slots.

## Unicode 17 integration

The implementation adds the private runtime facility:

`src/main/java/com/guillermomolina/protos/execution/ProtosRegexUnicodeFacility.java`

and provisions it privately to `std:regex/Regex`.

The module resolves the selected Unicode 17.0.0 categories/properties/scripts and simple case-closure behavior without delegating regex syntax or matching authority to `java.util.regex`.

The accepted property model covers the D187 baseline, including General_Category, Script, Script_Extensions, selected required binary properties, Any, ASCII and Assigned.

```text
JAVA_UTIL_REGEX_LANGUAGE_AUTHORITY=ABSENT
UNICODE_DATA_BASELINE=17.0.0
```

## Captures and Match preparation

Compilation numbers capturing groups in opening-parenthesis order and validates unique ASCII capture names.

The private Match constructor prepared for LIB014-2 stores:

- group 0 plus explicit groups;
- half-open scalar start/end bounds;
- null for nonparticipating groups;
- captured text snapshots;
- named-group lookup metadata.

Match snapshots are frozen and collection-returning accessors produce fresh snapshots so callers cannot mutate Match state.

## Resource behavior

LIB014-1 represents counted repetition without source-size expansion and records an unbounded Integer weight estimate for compiled nodes.

Regression coverage includes extremely large repetition counts and deep nesting, proving the parser/representation does not naively expand repetitions or depend on host recursion for nested structure.

No wall-clock timeout, public engine-step counter or global mutable regex cache was introduced.

## Test corpus

The published slice adds the suite-native corpus:

```text
protos/tests/library/regex/compile.protos
protos/tests/library/regex/escape.protos
protos/tests/library/regex/pattern-model.protos
protos/tests/library/regex/rejection.protos
```

and registers it with the repository Test Tool corpus/suite infrastructure.

Coverage includes accepted syntax, excluded syntax, flags, Unicode properties, capture metadata, Pattern immutability, non-expanded counted repetition, deep nesting and escaping.

Native Java tests cover the Unicode facility and the new runtime/native-boundary registration.

## Files published

Material LIB014-1 files include:

```text
protos/lib/regex/Regex.protos
protos/tests/library/regex/compile.protos
protos/tests/library/regex/escape.protos
protos/tests/library/regex/pattern-model.protos
protos/tests/library/regex/rejection.protos
src/main/java/com/guillermomolina/protos/execution/ProtosRegexUnicodeFacility.java
```

The commit also updates the required runtime bootstrap/CLI/Test Tool corpus wiring, architecture guards, `pom.xml`, and `CHANGELOG.md`.

## Scope boundary preserved

LIB014-1 intentionally does **not** implement:

```text
Pattern.fullMatch
Pattern.search
Pattern.searchFrom
Pattern.eachMatch
Pattern.findAll
Pattern.replaceFirst
Pattern.replaceAll
Pattern.split
```

No Core `pattern.match(subject)` integration is added.

No rich/backtracking regex extension is introduced.

## Slice result

```text
LIB014_1_IMPLEMENTATION=PUBLISHED
PUBLISHED_SHA=0abd607a3a354ecd2b6551e344661d4b727d7910
GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

D187_CONTRACT_PRESERVED=YES
NEXT_SLICE=LIB014-2
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT from the published LIB014-1 revision, the ratified D187 contract, and the human executor's explicit local validation report. No independent execution of the reported local tests is claimed by this record.
