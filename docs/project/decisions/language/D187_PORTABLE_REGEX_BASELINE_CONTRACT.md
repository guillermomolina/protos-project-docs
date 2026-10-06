# D187 — Portable regular-expression baseline contract

Status: **RATIFIED — OWNER APPROVED; DEPENDENT IMPLEMENTATION RELEASE WAITS FOR LIVE HIERARCHY RECONCILIATION**

Owning decision: `guillermomolina/protos#809` — `D187 — Portable regular-expression baseline contract`

Parent workstream: `guillermomolina/protos#431` — `LIB014 — Regular expressions and pattern-matching library`

Nature: durable implementation-independent language/Standard-Library decision record; **non-normative**

Approval date: **2026-10-06**

Audited Protos revision:

```text
PROTOS_REVISION=e1904bb01b61c6ee0da5fd279da0f7feb143229e
```

## Approval provenance

LIB014-0 completed the comparative regex investigation and first obtained explicit owner approval for C5+C1.

The remaining exact-contract bundle was then presented as one approval unit. In the active interaction the project owner answered:

> aprobado.

This record therefore ratifies the exact bundle below.

```text
DECISION_APPROVAL_PROVENANCE=PASS
C5=APPROVED
C1=APPROVED
EXACT_BASELINE_CONTRACT=APPROVED
```

## Invariant consistency

Previously approved LIB014 invariants:

```text
C5_PORTABLE_BASELINE_PLUS_SEPARATELY_NAMED_RICH_EXTENSION
C1_RESTRICTED_PORTABLE_PREDICTABLE_LINEAR_TIME_BASELINE
UNRESTRICTED_PCRE_PERL_BASELINE=REJECTED
```

The exact contract preserves all of them.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_PRIOR_INVARIANTS=NONE
```

## Ratified contract

### Matching

```text
MATCH_SELECTION=LEFTMOST_FIRST
ALTERNATIVES=ORDERED
GREEDY_QUANTIFIERS=YES
LAZY_QUANTIFIERS=YES
FULL_MATCH=YES
SEARCH=YES
SEARCH_FROM_SCALAR_OFFSET=YES
```

### Unicode / text model

```text
MATCHING_UNIT=UNICODE_SCALAR
OFFSET_UNIT=PROTOS_STRING_SCALAR_INDEX
SPAN_FORM=HALF_OPEN_START_END
UNICODE_DATA_BASELINE=17.0.0
UTS18_LEVEL1_TARGET=YES
IMPLICIT_NORMALIZATION=NO
GRAPHEME_MATCHING_BASELINE=NO
CASE_FOLDING=SIMPLE_UNICODE_LOCALE_INDEPENDENT
```

- `\d` follows Unicode Decimal_Number.
- `\s` follows Unicode White_Space.
- `\w` and `\b` follow the selected UTS #18 Level 1 Unicode word-character/boundary model rather than ASCII-only rules.
- Multiline line boundaries recognize CRLF as one line sequence, plus LF, CR, NEL U+0085, U+2028, and U+2029.
- Dot excludes the selected line terminators unless dot-all is enabled.
- No implicit Unicode normalization is performed.
- Baseline regex does not match by extended grapheme cluster.

### Baseline syntax

Included:

- literals and escapes;
- Unicode scalar escapes;
- dot;
- character classes, negation, ranges, intersection and subtraction;
- Unicode properties/categories/scripts;
- concatenation and ordered alternation;
- numbered captures;
- non-capturing groups;
- unique named captures;
- `?`, `*`, `+`, `{n}`, `{n,}`, `{n,m}`;
- greedy and lazy forms;
- `^`, `$`, `\A`, `\z`, `\b`, `\B`;
- compile flags `i`, `m`, `s`, `x`.

Named capture names use:

```text
[A-Za-z_][A-Za-z0-9_]*
```

Compile flags are supplied as a String containing zero or more unique letters from `i`, `m`, `s`, `x`; order is semantically irrelevant. Unknown or duplicate flags are invalid.

Explicitly deferred/excluded:

- backreferences;
- look-ahead and look-behind;
- recursive patterns and subroutine calls;
- conditionals and branch-reset groups;
- duplicate capture names and capture history;
- balancing groups;
- callouts and embedded code;
- atomic groups and possessive quantifiers;
- `\G`, `\K`, `\X`;
- canonical-equivalence matching;
- locale-sensitive matching;
- byte/code-unit matching;
- mutable global/sticky cursor state;
- inline embedded flag-modifier syntax;
- streaming/incremental matching.

Atomic and possessive constructs are deferred for simplicity, not because they are classified as intrinsically unsafe.

### Capture contract

```text
GROUP_0=WHOLE_MATCH
EXPLICIT_GROUP_NUMBERING=OPEN_PAREN_SOURCE_ORDER
NAMED_GROUPS=YES
NAMED_GROUPS_ALSO_NUMBERED=YES
DUPLICATE_NAMES=NO
NONPARTICIPATING_CAPTURE=NULL
EMPTY_PARTICIPATING_CAPTURE=EMPTY_STRING_WITH_EQUAL_OFFSETS
REPEATED_CAPTURE=LAST_SELECTED_PARTICIPATION
CAPTURE_OFFSETS=UNICODE_SCALAR_HALF_OPEN
```

### Pattern / Match model

```text
PATTERN=ORDINARY_SEMANTICALLY_IMMUTABLE_VALUE
PATTERN_NORMAL_IDENTITY=YES
COMPILED_REPRESENTATION=PRIVATE
MUTABLE_PUBLIC_MATCHER_CURSOR=NO
MATCH=IMMUTABLE_ORDINARY_RESULT
EXECUTION_STATE=PER_OPERATION_LOCAL
TASK_SAFE_SHARED_PATTERN=YES
REQUIRED_GLOBAL_MUTABLE_MATCH_STATE=NO
```

Pattern/Match values may cross existing isolation boundaries only through ordinary portable value semantics. A live host matcher is never transferred. Private compiled artifacts may be reconstructed or immutably shared when existing Protos isolation rules permit and the distinction is unobservable.

Collections exposed from Match are snapshots or otherwise must not allow caller mutation to change the Match.

### Public module/API

Initial module:

```text
std:regex/Regex
```

Initial public operations:

```text
Regex.compile(source)
Regex.compileWithFlags(source, flags)
Regex.escape(text)
Regex.escapeReplacement(text)

Pattern.fullMatch(text) -> Match | null
Pattern.search(text) -> Match | null
Pattern.searchFrom(text, scalarOffset) -> Match | null

Pattern.eachMatch(text, block) -> null
Pattern.findAll(text) -> Array<Match>

Pattern.replaceFirst(text, replacement) -> String
Pattern.replaceAll(text, replacement) -> String
Pattern.split(text) -> Array<String>
```

No regex convenience methods are initially added to String.

No new public regex-specific Error hierarchy is created by D187. Invalid arguments, syntax, replacement references, offsets and exhausted compile-resource budgets use ordinary Protos Error signaling unless already-ratified narrower standard error semantics apply.

### Complexity / ReDoS

For effective compiled pattern size `m` and input length `n` in Unicode scalars:

```text
single fullMatch/search/searchFrom: O(m * n) worst case
successive eachMatch/findAll/replaceAll/split: polynomial; O(m * n^2) worst case permitted
catastrophic exponential backtracking: excluded by baseline contract
```

Compilation is resource-bounded and must fail cleanly on pathological source/state expansion rather than relying on host stack exhaustion or uncontrolled allocation.

```text
WALL_CLOCK_TIMEOUT_REQUIRED=NO
PUBLIC_ENGINE_STEP_COUNTER=NO
PUBLIC_DFA_CACHE_LIMIT=NO
```

### Host-engine and implementation freedom

Host delegation is not a public compatibility promise.

```text
HOST_ENGINE_DELEGATION=OPTIONAL_INTERNAL_OPTIMIZATION
EXACT_PROTOS_EQUIVALENCE_REQUIRED=YES
PORTABLE_FALLBACK_REQUIRED=YES
JAVA_UTIL_REGEX_IS_NOT_LANGUAGE_AUTHORITY=YES
```

Any accelerator must preserve:

- accepted and rejected syntax;
- Unicode version and property semantics;
- case folding;
- newline behavior;
- leftmost-first / ordered / greedy-lazy selection;
- capture semantics;
- scalar offsets;
- zero-width behavior;
- complexity and resource guarantees.

### Reference/fallback architecture

The public contract does not expose one engine representation.

```text
PROTOS_OWNED_REGEX_PARSER=YES
PRIVATE_COMPILED_IR=YES
PORTABLE_REFERENCE_FALLBACK=PRIORITIZED_THOMPSON_NFA_OR_PIKE_VM
BOUNDED_DFA_OR_LAZY_DFA=OPTIONAL_OPTIMIZATION
```

The exact IR, automaton layout, caches, compiled metadata, and specialization strategy remain implementation machinery.

### Zero-width progression

For successive operations:

1. emit the selected match;
2. after a consuming match `[s,e)`, continue from `e`;
3. after a zero-width match `[k,k)`, emit it once and continue from the next Unicode scalar boundary;
4. a zero-width match at end-of-input is emitted once and then iteration terminates.

### Replacement

Initial replacement grammar:

```text
$$       literal $
${0}     whole match
${1}     numbered capture
${name}  named capture
```

- reference to a nonexistent capture is Error;
- a valid nonparticipating capture expands to the empty String;
- callable/effectful replacement is deferred.

### Split

```text
DELIMITER_MATCHES_IN_OUTPUT=NO
DELIMITER_CAPTURES_IN_OUTPUT=NO
PRESERVE_EMPTY_FIELDS=YES
ZERO_WIDTH_USES_COMMON_PROGRESS_RULE=YES
```

Leading, interior and trailing empty fields are retained.

### Core matching boundary

```text
LIB014_REDEFINES_CORE_PATTERN_MATCH=NO
REGEX_MATCH_RETURNS_RICH_MATCH_OBJECT_VIA_REGEX_API=YES
DIRECT_PATTERN_MATCH_PROTOCOL_INTEGRATION=DEFERRED
```

Any future adapter from Regex Pattern to Core `pattern.match(subject)` must obey the existing matcher-outcome contract and requires its own explicit design decision.

### Streaming / caches

```text
STREAMING_INCREMENTAL_MATCHING_INITIAL=NO
REQUIRED_GLOBAL_MUTABLE_CACHE=NO
HIDDEN_BOUNDED_CACHE=MAY_BE_IMPLEMENTATION_FREEDOM
GLOBAL_SERIALIZATION_LOCK=NO
```

## Implementation decomposition

After the live D187 publication postconditions are complete, implementation may proceed as:

### LIB014-1

Parser, exact profile, immutable Pattern/Match value model, compile validation/resource limits, focused conformance.

### LIB014-2

Safe matcher, captures, full/search/searchFrom, traversal, zero-width behavior, adversarial complexity and concurrency evidence.

### LIB014-3

Replacement, split, escaping, public integration, documentation/conformance and Native Image correctness.

### LIB014-4

Optional measured acceleration only if later benchmarks justify it. No semantic change is authorized in this optional slice.

## Publication and closure state

This durable record completes the decision publication itself, but D187 must not be reported closed or dependent implementation released until the live GitHub hierarchy required by project governance is verified.

At publication time the connector used to create D187 did not expose native sub-issue mutation, so the textual `Parent: #431` declaration is bootstrap input only.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
NATIVE_PARENT=VERIFY_LIVE_BEFORE_CLOSURE
IMPLEMENTATION_RELEASE=WAIT_FOR_D187_CLOSURE_POSTCONDITIONS
```

## AI-assistance disclosure

This durable ratification record was materially prepared with AI assistance from ChatGPT from the completed LIB014-0 research packet, current repository policy, live GitHub state, and the project owner's explicit C5+C1 and exact-contract approvals. No independent human review is claimed by this record.
