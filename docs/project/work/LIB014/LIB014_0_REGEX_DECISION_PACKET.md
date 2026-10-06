# LIB014-0 — Regular-expression comparative design packet

Status: **RESEARCH COMPLETE — C5 + C1 APPROVED; REMAINING CONTRACT DECISIONS OPEN**

Owning work item: `guillermomolina/protos#431` — `LIB014 — Regular expressions and pattern-matching library`

Nature: durable Standard Library design/work record; **non-normative**

Research revision:

```text
PROTOS_REVISION=e1904bb01b61c6ee0da5fd279da0f7feb143229e
```

Approval evidence:

```text
APPROVAL_RECORD=docs/project/evidence/LIB014/LIB014_0_C5_C1_OWNER_APPROVAL.md
APPROVAL_RECORD_REVISION=ed4fa6cd06ef1d7d794868884a1c39a219dd9f29
```

## Executive conclusion

The recommended architecture is a deliberate combination of:

- **Candidate 5** as product/evolution strategy: keep a portable baseline and reserve any future incompatible richer feature set for a separately named explicit extension/profile;
- **Candidate 1** as the baseline language family: a restricted portable regular-expression language selected to preserve predictable/linear-time matching behavior rather than unrestricted Perl/PCRE-style backtracking semantics; and
- a possible constrained Candidate 3 mechanism only as internal implementation freedom when exact Protos semantic equivalence can be demonstrated.

On 2026-10-06 the project owner explicitly approved **C5 + C1**. Candidate 3 and the remaining exact public-contract choices are not yet approved.

## Current Protos constraints found in HEAD

### String / Unicode

`spec/semantics/VALUES_AND_COLLECTIONS.md` establishes that a Protos `String` is an immutable ordered sequence of Unicode scalar values. Core `String.size` and indexed access operate on scalar values, not bytes, UTF-16 code units, or grapheme clusters. String identity/equality performs no implicit Unicode normalization.

Consequences for LIB014:

- regex matching should use Unicode scalar values as its baseline text unit;
- public match offsets should use Protos scalar indexes, not JVM UTF-16 indexes or UTF-8 byte offsets;
- grapheme-cluster matching must not silently redefine String semantics;
- no implicit Unicode normalization should be introduced by regex.

### General matching protocol

`spec/semantics/MATCHING.md` owns the general Protos pattern-matching protocol:

```text
pattern.match(subject)
```

Its normal outcome carrier is exactly canonical `false`, canonical `true`, or a non-empty standard Array of positional captures.

A regex `Match` result needs richer information such as text spans, capture participation, named groups, and offsets. LIB014 therefore must not accidentally redefine the existing Core matching protocol or return a regex Match object from `Pattern.match(subject)` without a separate explicit design decision.

### Error model

`spec/semantics/ERRORS.md` owns ordinary Error signaling. Regex compile/match domain failures should use existing Protos Error machinery rather than introduce a parallel exception universe.

### Concurrency and ownership

`spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`, and `spec/semantics/MODULES.md` require Actor isolation, allow semantically invisible immutable physical sharing, and forbid mutable host/library globals from becoming cross-Actor shared mutable application state.

Consequences:

- a compiled Pattern should be semantically immutable;
- per-match execution state must remain local;
- no public mutable Matcher cursor is required;
- no required global mutable pattern cache;
- internal immutable compiled representations may be shared when unobservable.

### Cancellation

`spec/concurrency/FUTURES_AND_TASKS.md` owns cooperative cancellation boundaries. LIB014 should not silently invent a new cancellation checkpoint inside synchronous regex execution. If interruptible CPU-bound regex execution is desired as a portable guarantee, that requires a separate substantive concurrency decision.

### Standard Library conventions

Current `std:*` modules such as `protos/lib/semver/SemVer.protos` expose ordinary Protos APIs and values rather than leaking host implementation objects. `protos/lib/collections/Range.protos` provides callback-oriented traversal and does not establish a generic mutable Iterator/cursor institution.

## External ecosystem evidence

The research compared the required systems using official documentation and upstream sources.

### RE2

RE2 deliberately rejects constructs such as backreferences and look-around that are incompatible with its automata model, and is designed to avoid catastrophic backtracking with linear-time behavior in input length for a fixed expression.

Primary source: <https://github.com/google/re2>

### Rust `regex`

Rust's `regex` crate supports a Unicode-oriented regular language while excluding look-around and backreferences. It documents `O(m * n)` worst-case search behavior for pattern size `m` and haystack size `n`, while also documenting that repeated iteration can in some cases reach `O(m * n^2)` without being exponential backtracking.

Primary sources:

- <https://docs.rs/regex/latest/regex/>
- <https://docs.rs/crate/regex/latest/source/src/lib.rs>

### Go `regexp`

Go's `regexp` package uses RE2-style syntax/semantics and guarantees linear-time execution. It also provides leftmost-first matching semantics, demonstrating that user-familiar ordered matching does not require recursive backtracking.

Primary source: <https://pkg.go.dev/regexp>

### PCRE2

PCRE2 provides a rich Perl-compatible feature set including backreferences, look-around, atomic groups, possessive quantifiers, recursion/subroutines, and other advanced constructs. It provides match/depth/heap limits to bound problematic execution.

Primary sources:

- <https://pcre2project.github.io/pcre2/doc/pcre2pattern/>
- <https://pcre2project.github.io/pcre2/doc/pcre2api/>

### Perl

Perl remains important prior art for rich regex semantics, ordered alternatives, captures, zero-width behavior, and global matching rules.

Primary source: <https://perldoc.perl.org/perlre>

### Java `java.util.regex`

Java exposes an immutable/thread-safe `Pattern` and a mutable/non-thread-safe `Matcher`. The dialect includes look-around, backreferences, atomic groups and possessive quantifiers and is implemented as an ordered NFA/backtracking family.

Primary source: <https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/regex/Pattern.html>

The useful precedent is Pattern/Matcher state separation. The Java regex dialect and UTF-16-host representation must not define the Protos contract accidentally.

### .NET

.NET provides a traditional backtracking engine with timeout support and also a `NonBacktracking` mode that excludes constructs such as look-around/backreferences/atomic groups in exchange for stronger execution guarantees.

Primary source: <https://learn.microsoft.com/dotnet/standard/base-types/backtracking-in-regular-expressions>

This is strong evidence that a safe baseline and a rich compatibility mode are meaningfully different contracts.

### Python `re`

Python provides mature `Match` objects, numbered/named captures, nonparticipating groups, iteration and replacement semantics. It is useful API prior art but does not supply the baseline complexity guarantee recommended for Protos.

Primary source: <https://docs.python.org/3/library/re.html>

### ECMAScript RegExp

ECMAScript exposes mutable `lastIndex` state on regex values for global/sticky matching and must express Unicode behavior within its UTF-16 String representation.

Primary source: <https://tc39.es/ecma262/multipage/text-processing.html>

This is negative evidence for introducing a public mutable cursor into Protos.

### Oniguruma / Onigmo and Ruby

Oniguruma/Onigmo expose a large syntax space including named/duplicate groups, look-around, subexpression calls and other rich constructs. Ruby has introduced memoization to make many regex cases linear but still retains advanced cases requiring fallback protections/timeouts.

Primary sources:

- <https://github.com/kkos/oniguruma/blob/master/doc/SYNTAX.md>
- <https://www.ruby-lang.org/en/news/2022/12/25/ruby-3-2-0-released/>

### ICU

ICU provides strong Unicode regex support and an explicit Pattern/Matcher API, while also documenting pathological exponential cases and offering time/stack limits.

Primary source: <https://unicode-org.github.io/icu/userguide/strings/regexp.html>

### Smalltalk / Pharo

Pharo provides regex as a library facility rather than making regex a fundamental new language pattern universe.

Relevant source: <https://books.pharo.org/deep-into-pharo/>

## Candidate families

### C1 — restricted portable linear-time language — APPROVED BASELINE FAMILY

A deliberately restricted language whose semantics admit predictable automata-based matching and exclude constructs that destroy the baseline guarantee.

Strengths:

- strongest ReDoS posture;
- portable contract independent of JVM;
- easy immutable Pattern model;
- scalable across Tasks/Actors;
- preserves alternate engine freedom.

Cost:

- rejects some patterns familiar to PCRE/Java/Python users.

### C2 — rich PCRE/Perl compatibility

Rich compatibility with look-around, backreferences and related constructs plus explicit resource controls.

Rejected as baseline because timeouts/step limits are mitigations, not the same public guarantee as a regular-language baseline.

### C3 — portable Protos contract with host-backed engines

Potentially useful as implementation freedom only.

A host-backed engine can be used only when exact equivalence is demonstrated for syntax, matching selection, Unicode, captures, offsets, zero-width behavior and resource/complexity guarantees. Directly wrapping `java.util.regex` is not acceptable as the language contract.

**Owner approval status: NOT APPROVED.**

### C4 — pure-Protos engine

Viable as a fallback/reference implementation but should not be part of the public contract. Requiring a pure-Protos engine permanently would constrain later implementations unnecessarily.

### C5 — portable baseline + separately named richer extension — APPROVED PRODUCT STRATEGY

The baseline keeps its guarantees permanently. If future demand justifies constructs incompatible with those guarantees, they belong to a clearly separate profile/module/API with its own resource and portability contract.

## Candidate scorecard

Scores 1–5; higher is better. Runtime/resource cost scores favor lower/more predictable cost.

| Candidate | Correctness | Protos alignment | Future resilience | Scalability | Simplicity | Portability / freedom | Runtime / resources | Operability | Reversibility | Evidence maturity |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **C1 restricted portable** | 5 | 5 | 4 | 5 | 4 | 5 | 5 | 5 | 4 | 5 |
| C2 rich PCRE/Perl | 3 | 2 | 3 | 2 | 2 | 2 | 2 | 2 | 2 | 5 |
| C3 portable + host engines | 3 | 4 | 4 | 3 | 4 | 4 | 4 | 3 | 5 | 4 |
| C4 pure-Protos engine | 3 | 5 | 4 | 3 | 2 | 5 | 2 | 3 | 4 | 2 |
| **C5 staged baseline/extensions** | 5 | 5 | 5 | 5 | 4 | 5 | 5 | 5 | 5 | 5 |

Focused owner axes:

| Candidate | Aguante de futuro | Escalabilidad | Filosofía Protos |
| --- | ---: | ---: | ---: |
| **C1** | **4** | **5** | **5** |
| C2 | 3 | 2 | 2 |
| C3 | 4 | 3 | 4 |
| C4 | 4 | 3 | 5 |
| **C5** | **5** | **5** | **5** |

## Recommended exact baseline contract — PENDING OWNER APPROVAL

The completed research recommended the following contract details. They remain proposals unless separately marked approved below.

### Matching selection

Proposed:

- leftmost-first search;
- ordered alternatives;
- greedy and lazy quantifiers.

Go/RE2 demonstrate that leftmost-first semantics do not require recursive backtracking.

### Unicode

Proposed:

- Unicode scalar values are the matching/indexing unit;
- match offsets are half-open scalar indexes `[start,end)`;
- no implicit normalization;
- no grapheme-cluster matching in the baseline;
- Unicode properties/categories/scripts available;
- Unicode-aware `\d`, `\s`, `\w`, boundaries and simple case folding;
- Unicode line-boundary semantics for multiline mode;
- UTS #18 Level 1 as the intended conformance baseline.

Because current Protos String work already references Unicode 17.0.0, the packet proposed aligning LIB014 with that project baseline instead of silently upgrading String/text semantics merely because a newer Unicode release exists.

Primary Unicode source: <https://unicode.org/reports/tr18/>

### Proposed baseline syntax

Include:

- literals and escapes;
- Unicode scalar escapes;
- `.`;
- character classes, negation, ranges, intersection/subtraction;
- Unicode property classes;
- concatenation;
- ordered alternation;
- numbered capturing groups;
- non-capturing groups;
- unique named captures;
- `?`, `*`, `+`, `{n}`, `{n,}`, `{n,m}`;
- greedy/lazy forms;
- `^`, `$`, `\A`, `\z`;
- `\b`, `\B`;
- flags equivalent to case-insensitive, multiline, dot-all and extended/free-spacing.

Explicitly defer/exclude from baseline:

- backreferences;
- look-ahead;
- look-behind;
- recursive patterns;
- subroutine calls;
- conditional constructs;
- branch-reset groups;
- duplicate capture names;
- capture history;
- balancing groups;
- callouts/embedded code;
- atomic groups;
- possessive quantifiers;
- `\G`;
- `\K`;
- `\X`;
- canonical-equivalence matching;
- locale-sensitive matching;
- mutable global/sticky cursor state.

Atomic groups and possessive quantifiers are deferred for simplicity, not classified as intrinsically unsafe.

### Captures

Proposed:

- group 0 = whole match;
- explicit groups numbered in source order;
- named groups also have numbers;
- names unique;
- nonparticipating capture -> `null`;
- empty participating capture -> empty String with equal start/end;
- repeated capture -> last selected participation;
- public offsets use scalar indexes.

### Pattern / Match model

Proposed:

- `Pattern` is an ordinary semantically immutable Protos value;
- compiled engine representation is private;
- match execution state is local to each operation;
- `Match` is an immutable ordinary result, not a live mutable engine cursor;
- APIs expose whole match, captures, named captures and start/end offsets.

### Public API shape

Proposed conceptual module:

```text
std:regex/Regex
```

Proposed operations:

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

The packet recommends not adding String convenience methods initially.

### General matching separation

The packet recommends that LIB014 baseline **not** redefine Core `pattern.match(subject)`.

If regex Patterns later participate directly in language-level matching, that adapter must obey `spec/semantics/MATCHING.md` and should be handled by a separate explicit design decision.

### Complexity / ReDoS

Proposed public contract:

- no catastrophic exponential backtracking in the baseline;
- single `fullMatch` / `search`: worst-case `O(m * n)` for effective compiled pattern size `m` and input scalar length `n`;
- repeated search/iteration may be polynomial and can reach `O(m * n^2)` under leftmost-first semantics, without becoming exponential backtracking;
- compilation must have finite resource budgets and fail cleanly for pathological pattern size/state expansion;
- no wall-clock timeout is required for ordinary safe-baseline matching;
- no engine-step counter should become a public semantic unit.

### Zero-width progress

Proposed deterministic progress rule:

1. emit the selected match;
2. after a consuming match, resume at its end;
3. after a zero-width match `[k,k)`, emit it once and resume from the next Unicode scalar boundary;
4. a zero-width match at end-of-input is emitted once, then iteration terminates.

### Replacement

Proposed minimal replacement grammar:

```text
$$       literal $
${0}     whole match
${1}     numbered capture
${name}  named capture
```

Invalid group references fail. A valid but nonparticipating group expands to the empty String.

Callable/effectful replacements are deferred.

### Split

Proposed:

- delimiters are removed;
- delimiter captures are not inserted into output;
- empty leading/interior/trailing fields are preserved;
- zero-width delimiters use the same deterministic progress rule.

### Engine architecture

Proposed reference/fallback architecture:

```text
regex source
    -> Protos-owned parser
    -> validated AST / private compiled IR
    -> prioritized Thompson NFA / Pike VM fallback
    -> optional bounded DFA/lazy-DFA optimizations
    -> optional exactly-equivalent host accelerator
```

The parser, public language contract, Unicode behavior and result semantics remain Protos-owned.

### Caching / concurrency

Proposed:

- Pattern may share immutable compiled representation;
- each operation has local mutable execution state;
- hidden bounded caches may exist only when semantically invisible;
- no required global mutable cache;
- no global serialization lock across independent matches.

### Native Image / alternate runtimes

The recommended contract avoids mandatory JNI/native dependencies and does not make Java regex semantics public. Host/runtime acceleration remains optional machinery.

## Owner approval recorded on 2026-10-06

Exact approved portion:

```text
APPROVED_CANDIDATES=C5+C1
C3_APPROVED=NO
```

Durable evidence is in:

`docs/project/evidence/LIB014/LIB014_0_C5_C1_OWNER_APPROVAL.md`

Approved invariants:

1. baseline belongs to the restricted portable predictable/linear-time C1 family;
2. future incompatible richer constructs belong behind a distinct explicit C5 extension/profile rather than silently weakening baseline guarantees;
3. unrestricted PCRE/Perl-style compatibility is not the approved baseline.

All exact contract details in the preceding sections remain proposals until explicitly selected.

## Remaining owner decisions

Before implementation, owner approval is still required for one exact bundle covering:

1. matching selection: leftmost-first / ordered / greedy-lazy;
2. Unicode/indexing/property/case-folding/newline contract;
3. exact baseline syntax and explicit exclusions;
4. capture numbering/naming/nonparticipation/repetition semantics;
5. Pattern/Match immutability, ownership and public operations;
6. exact public complexity wording for single and repeated matching;
7. engine/fallback/host-delegation freedom;
8. zero-width iteration/replacement/split semantics;
9. baseline separation from Core `pattern.match(subject)`;
10. exact initial module/API spelling.

The packet recommends resolving these together, not as one-file micro-slices.

## Recommended implementation decomposition after full ratification

After the remaining contract is explicitly approved and formally ratified, the recommended implementation sequence is:

### LIB014-1 — parser, contract and immutable value model

One substantial slice covering:

- approved syntax/profile;
- Unicode contract;
- parser and validation;
- capture numbering/names;
- immutable Pattern/Match model;
- compile/resource Errors;
- private IR;
- focused conformance tests.

### LIB014-2 — safe matcher, captures and traversal

One substantial slice covering:

- Thompson/Pike execution;
- full-match/search/search-from;
- leftmost-first + greedy/lazy;
- captures and scalar offsets;
- zero-width progress;
- eachMatch/findAll;
- adversarial complexity tests;
- concurrency/ownership evidence.

### LIB014-3 — replacement, split and integration closure

One substantial slice covering:

- replacement grammar;
- escaping;
- replace-first/all;
- split;
- edge cases;
- docs/conformance;
- public std: integration;
- Native Image correctness;
- integration consumers where already justified.

### LIB014-4 — optional measured acceleration

Only if benchmarks justify it:

- bounded lazy-DFA/cache;
- prefix/literal optimizations;
- alternate exactly-equivalent engine adapters;
- Native Image tuning.

No semantic changes are permitted in this optional performance slice.

## Strongest argument against the recommendation

Users coming from PCRE/Python/Java/JavaScript often expect look-around and backreferences. Rejecting those constructs may require code rewrites and may make Protos regex feel less compatible with existing corpora.

The countervailing argument is that Protos currently has no compatibility debt forcing it to inherit catastrophic-backtracking/resource semantics. C5 preserves an escape path: later demand can justify a separate explicit rich profile without weakening the safe baseline.

## Regret scenario and escape path

Regret becomes plausible if real Protos workloads repeatedly import PCRE/Java/Python expressions requiring look-around/backreferences and the rewrite cost is significant.

Escape path:

- keep `std:regex` baseline guarantees unchanged;
- add a separately named rich profile/module only after its semantics/resource model are explicitly designed;
- continue allowing the baseline engine implementation to evolve internally because compiled representation is not public.

## Files materially inspected in `guillermomolina/protos`

- `AGENTS.md`
- `AGENTS.work/LIBRARY.md`
- `AGENTS.work/DESIGN.md`
- `AGENTS.work/REFERENCE.md`
- `spec/semantics/MATCHING.md`
- `spec/semantics/VALUES_AND_COLLECTIONS.md`
- `spec/semantics/ERRORS.md`
- `spec/semantics/MODULES.md`
- `spec/concurrency/ACTORS.md`
- `spec/concurrency/PARALLEL_EXECUTION.md`
- `spec/concurrency/FUTURES_AND_TASKS.md`
- `protos/AGENTS.md`
- `protos/lib/collections/Range.protos`
- `protos/lib/semver/SemVer.protos`
- live `guillermomolina/protos#431`

## Current gate

```text
LIB014_0_RESEARCH=COMPLETE
LIB014_C5_C1_SELECTION=APPROVED
LIB014_REMAINING_CONTRACT=NEEDS_OWNER_DECISION
FORMAL_DXXX_RATIFICATION=PENDING
LIB014_IMPLEMENTATION_AUTHORIZED=NO
NEXT_STEP=ONE_BOUNDED_REMAINING_CONTRACT_DECISION_PASS
```

## AI-assistance disclosure

This durable decision packet was materially prepared with AI assistance from ChatGPT from the LIB014-0 exhaustive comparative research, current Protos repository authority, official upstream documentation, and the project owner's explicit C5 + C1 approval. No independent human review is claimed by this record.
