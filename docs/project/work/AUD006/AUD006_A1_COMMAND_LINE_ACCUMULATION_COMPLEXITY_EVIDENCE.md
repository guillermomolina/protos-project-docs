# AUD006-A1 — CommandLine accumulation complexity and repair boundary

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
SLICE=AUD006-A1
SLICE_TYPE=INVESTIGATION
STATUS=COMPLETE
PROTOS_HEAD_AT_PUBLICATION=7f398f623a210b2bc3837d37dd3e5980dd726e75
PROTOS_HEAD_SUBJECT=I070-A: eliminate handwritten main compilation warnings
ANALYSIS_BASE=53c54bb952354cae61e72db68c4ff8f509c827ed
ANALYSIS_BASE_SUBJECT=TEST009-O: bound shared host leaves exposed by M5 native-body PIC
HEAD_RECONCILIATION=PASS
```

This is a durable non-normative audit/evidence record. It does not define Protos
semantics and does not authorize an implementation or a new runtime primitive.

AUD006-A1 was performed against the published repository state without executing
builds, tests, programs, benchmarks, validators, or state-changing local Git
operations. The project owner subsequently reported the local validation state:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The later product commit from the analysis base to the publication HEAD was
reconciled before publication. The only intervening commit is I070-A. It does
not modify `protos/lib/cli/CommandLine.protos`,
`ProtosArrayValue`, or the `PreparedArgumentVector` implementation that
establishes the cost proof below, so the A1 conclusions remain valid at the
publication HEAD.

## Governing requirement

D115 / #441 ratified Candidate C-prime and retained the parser complexity target:

```text
time:   O(T + Svisited)
memory: O(result + Svisited)
```

AUD006 / #453 keeps LIB011 / #428 closed and treats this as a scalability /
conformance correction problem, not permission to redesign public
`std:cli/CommandLine` semantics.

## Current builder

At the publication HEAD, `CommandLine.parse` still uses the same invocation-local
balanced chunk builder:

```text
newBuilder
  chunks: Map()
  maxLevel: -1

appendBuilt(value)
  -> carryChunk(Array(value), level=0)

carry collision at level L
  -> Array(...left, ...chunk)
  -> carry to level L+1

finishBuilder
  -> collectChunks(maxLevel ... 0, Array())
  -> Array(...result, ...chunk) for every occupied level
```

The same builder is used for three result-producing surfaces in each selected
command scope:

1. option occurrences;
2. raw positional records;
3. final positional occurrences.

Large positional input therefore pays the accumulation mechanism twice: once
while collecting raw positional records and once while constructing the final
positional occurrence Array.

## Array construction cost

Current standard Array construction is materializing.

The relevant implementation path is:

```text
call spread
  -> PreparedArgumentVector(ArrayList)
  -> appendSpread(): append each source element reference
  -> snapshot(): List.copyOf(values)
  -> standard Array.call
  -> new ProtosArrayValue(prototype, supplied)
  -> new ArrayList(size) and one add per supplied reference
```

`ProtosArrayValue.indexedSnapshot()` can publish a read-only view of its current
generation in O(1), but a new standard Array does not structurally share that
backing. Its constructor allocates owned indexed storage and copies every
supplied element reference.

The current public Array surface also does not provide a general linear builder:

```text
Array.add=ABSENT
Array.atPut=REPLACEMENT_OF_EXISTING_INDEX_ONLY
Array.size=O(1)
Array.resize/grow=ABSENT
Array(length)=ABSENT
Array(3)=ONE-ELEMENT_ARRAY_CONTAINING_3
Array.freeze=O(1)_STATE_TRANSITION
```

Therefore knowing the final element count is not sufficient, by itself, to build
an exact-size Array from guest code using only the existing public mechanisms.

## Complexity proof

Let N be the number of values supplied to one balanced builder through
`appendBuilt`.

Each append creates a singleton Array. Whenever a binary carry collides at level
j, two chunks of size `2^(j-1)` are merged into a fresh Array of size `2^j`.
Counting only element-reference writes into the newly constructed standard
Arrays gives the conservative carry cost:

```text
Ccarry(N) =
  sum[j=1..floor(log2 N)] 2^j * floor(N / 2^j)
```

For `N = 2^k`:

```text
Ccarry(N) = N * k = N log2 N
```

The singleton creations add N more reference writes. For powers of two,
`finishBuilder` materializes one final N-element Array, so even this favorable
case is:

```text
append/carry = N + N log2 N
finish       = N
total        = N(log2 N + 2)
```

For `N = 2^k - 1`, every chunk level is occupied. The successive
`collectChunks` results have cumulative sizes whose sum is
`Theta(N log N)`, so finish itself has an `O(N log N)` worst case.

The exact number of standard Array materializations remains linear:

```text
singleton Arrays    = N
carry merge Arrays  = N - popcount(N)
finish empty Array  = 1
finish merge Arrays = popcount(N)

TOTAL_ARRAY_MATERIALIZATIONS = 2N + 1
```

The defect is not the number of Array objects. It is repeated copying of element
references across increasingly large materializations.

Final static classification:

```text
APPEND_CARRY_COST=Theta(N log N)
FINISH_COST=O(N log N)
FINISH_WORST_CASE=Theta(N log N)
TOTAL_ONE_BUILDER_COST=Theta(N log N)
POSITIONAL_DOUBLE_ACCUMULATION=Theta(N log N)
F1_STILL_PRESENT=YES
D115_TIME_BOUND_STATUS=VIOLATED
```

## Meaning of the D115 bound

For this audit, the non-circular interpretation is:

```text
T =
  input argument material actually consumed or inspected, including token
  contents such as clustered short-option members

Svisited =
  canonical CommandSpec material actually examined in selected scopes:
  command nodes, option specs, positional specs, and child specs/index entries

result =
  retained argv snapshot plus CommandResult nodes, option occurrences,
  positional occurrences, and their result Arrays
```

Repeated copies of an already-accumulated occurrence are not `Svisited`.
Neither are the internal copies performed by call-spread staging,
`List.copyOf`, or `ProtosArrayValue` construction. Counting those copies as
`visited` would make the target circular.

A single selected command scope is enough to produce the
`Theta(T log T + Svisited)` family; nested command traversal is not required to
establish F1.

## Existing retained behavior evidence

The current CLI corpus protects the observable semantics that any repair must
preserve:

- option and attached-value provenance;
- clustered short-option provenance;
- structural versus literal `--`;
- D111 positional allocation;
- D115 C-prime child traversal and parent-minimum reservation;
- irreversible child-scope ownership;
- fresh/frozen result identity and result shape;
- concurrent invocation-local parsing without a global parser registry; and
- a retained 128-option / 256-token large-spec case.

That large-spec case is functional evidence at one moderate size. It is not
retained asymptotic evidence.

No equivalent retained scaling series for large unbounded positional input was
found during A1.

## Candidate repair families

### Current balanced chunks

```text
CANDIDATE=CURRENT_BALANCED_CHUNKS
USES_ONLY_EXISTING_MECHANISMS=YES
TIME_TARGET_MET=NO
MEMORY_TARGET_MET=UNPROVEN
SEMANTICS_PRESERVED=YES
NEW_DECISION_REQUIRED=NO
```

It is statically falsified by standard Array materialization.

### Two passes with only current public Array mechanisms

```text
CANDIDATE=TWO_PASS_WITH_CURRENT_ARRAY
USES_ONLY_EXISTING_MECHANISMS=YES
TIME_TARGET_MET=NO
MEMORY_TARGET_MET=UNPROVEN
SEMANTICS_PRESERVED=YES
NEW_DECISION_REQUIRED=NO
```

A first pass could determine counts and boundaries without changing D111/D115,
but the second pass still lacks a public way to allocate N Array slots and fill
each once. Merely knowing N does not remove the construction problem.

### Private exact-size or append/finalize Array construction

A private internal linear builder can satisfy the asymptotic target if it:

- remains invocation-local;
- creates the same ordinary fresh standard Array result;
- preserves element order and identity;
- preserves freeze behavior;
- introduces no global parser state;
- changes no public Array API or semantics; and
- does not create a Java-only CommandLine parser or alternate result model.

The runtime already has internal examples of linear temporary accumulation, such
as `PreparedArgumentVector`, but no existing mechanism is currently a
CommandLine-accessible private Array builder. Reusing or introducing such a
boundary is therefore a durable runtime/Core architecture choice rather than a
purely mechanical Standard Library edit.

## Governance verdict

```text
AUD006_A_REPAIR_CLASS=NEW_PLATFORM_OR_RUNTIME_DECISION_REQUIRED
NEW_LANGUAGE_DECISION_REQUIRED=NO
PUBLIC_COMMANDLINE_SEMANTICS_CHANGE_REQUIRED=NO
PUBLIC_ARRAY_API_CHANGE_REQUIRED=NO
MINIMAL_REPAIR_FAMILY=PRIVATE_INTERNAL_LINEAR_ARRAY_CONSTRUCTION
```

A later repair must not silently expose or couple Standard Library code to an
execution-engine implementation detail. The exact private construction boundary
must cross the normal platform/runtime decision gate before product
implementation.

## Dynamic evidence still required

AUD006 requires retained scaling evidence before repair closure. The next slice
should add a deterministic or retained measurement harness covering at least:

```text
A: one repeatable value-taking option repeated N times
B: one unbounded positional receiving N values
```

Recommended scale family:

```text
N = 256, 512, 1024, 2048, 4096, 8192
```

Preferred evidence is deterministic element-reference
writes/copies/materializations, separated by accumulation surface, rather than
wall-clock timing alone. When timing is also retained, compare `time/N`,
`time/(N log2 N)`, and `time/N^2`.

The positional case should distinguish the raw-positional accumulation from the
final positional-occurrence accumulation.

Nested selected command paths are not required to prove F1 unless later
instrumentation reveals an F1-specific interaction.

## Next step

```text
NEXT_SLICE=AUD006-A2 — retained F1 scaling/copy evidence
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
A2_MAY_CHANGE_PRODUCT_SEMANTICS=NO
A2_MAY_IMPLEMENT_LINEAR_ARRAY_REPAIR=NO
A2_MAY_INTRODUCE_NEW_RUNTIME_PRIMITIVE=NO
```

A2 owns retained evidence only. Product remediation remains blocked until the
platform/runtime construction boundary has crossed its explicit decision gate.

AUD006 remains open. LIB011 / #428 remains closed.
