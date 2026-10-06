# AUD006-A2 — Full-parse scaling instrument infeasibility evidence

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
SLICE=AUD006-A2
SLICE_TYPE=IMPLEMENTATION_EVIDENCE_EXPERIMENT
STATUS=BLOCKED_BY_PLATFORM_RUNTIME_DECISION
PROTOS_HEAD_RECONCILED=206bbde5593e4eea414b695047d07b894e201fbc
PROTOS_HEAD_SUBJECT=TEST009-Q: revert global native-body PE specialization
PRODUCT_CHANGE_PUBLISHED=NO
RETAINED_TEST_PUBLISHED=NO
RETRY_SAME_FULL_PARSE_APPROACH=NO
```

This is a durable non-normative evidence record. It records why the attempted
black-box scaling instrument cannot provide the retained evidence originally
requested by AUD006-A2. It does not weaken the D115 complexity requirement and
does not select a runtime repair.

## A1 authority remains valid

AUD006-A1 established from current Array construction semantics that the
invocation-local balanced builder inside `std:cli/CommandLine.parse` has:

```text
APPEND_CARRY_COST=Theta(N log N)
FINISH_COST=O(N log N)
FINISH_WORST_CASE=Theta(N log N)
TOTAL_ONE_BUILDER_COST=Theta(N log N)
POSITIONAL_DOUBLE_ACCUMULATION=Theta(N log N)
D115_TIME_BOUND_STATUS=VIOLATED
```

The product repository advanced after A1. The current reconciled HEAD for this
record is:

```text
206bbde5593e4eea414b695047d07b894e201fbc
TEST009-Q: revert global native-body PE specialization
```

The relevant F1 mechanisms remain materially unchanged at that HEAD:

- `CommandLine.parse` still defines `newBuilder`, `carryChunk`,
  `appendBuilt`, `collectChunks`, and `finishBuilder` as local closures;
- carry still merges with `Array(...left, ...chunk)`;
- finish still repeatedly materializes `Array(...result, ...chunk)`;
- `ProtosArrayValue` still copies supplied elements into fresh owned
  `ArrayList` storage in its constructor; and
- `PreparedArgumentVector.appendSpread` still copies each spread element into
  an intermediate vector and `snapshot()` still uses `List.copyOf(values)`.

Therefore the static A1 proof remains authoritative.

## A2 experiment

A2 attempted to observe F1 through the complete public
`CommandLine.parse` path, using allocation and timing scaling over increasing
input sizes.

The experiment covered the two required families:

1. large unbounded positional input;
2. repeated value-taking `--tag` occurrences.

The maintainer reported that the synthetic classifier itself could distinguish
the intended mathematical classes, but the real full-parse measurement could
not isolate the builder term.

The key reported observations were:

| Workload | allocated bytes per N | observed variation in g(N)/N |
| --- | ---: | ---: |
| positional | approximately 61-65 KB | approximately +15,177 to -7,664 |
| repeated `--tag` | approximately 90-95 KB | approximately +7,557 to -16,861 |

At `N = 8192`, one measured parse cost was approximately:

```text
wall time ~= 27 s
allocated bytes ~= 0.7 GB
```

These values are exploratory maintainer-reported measurements, not a retained
performance contract.

## Why the instrument is non-discriminating

The F1 signal being sought is the extra balanced-builder term above the dominant
linear parse cost.

For the builder, the incremental distinction between linear and
`N log N` is on the order of a small number of copied references per element
per level. In the complete parse, however, the interpreter/runtime allocates on
the order of tens of kilobytes per input occurrence.

The reported full-parse baseline was therefore approximately:

```text
positional: 61-65 KB / N
--tag:      90-95 KB / N
```

while the expected incremental F1 signal was only on the order of tens of bytes
per element.

The reported per-element variation from Truffle compilation, caches, warmup,
GC, and runtime allocation behavior was thousands to tens of thousands of bytes,
roughly three orders of magnitude larger than the signal to classify.

Consequently:

```text
FULL_PARSE_ALLOCATION_SCALING=NON_DISCRIMINATING
FULL_PARSE_TIMING_SCALING=NON_DISCRIMINATING
SIGNAL_TO_NOISE=INSUFFICIENT_BY_ORDERS_OF_MAGNITUDE
INCREASING_REASONABLE_N=NOT_A_CREDIBLE_FIX
THRESHOLD_TUNING=NOT_A_CREDIBLE_FIX
REPEATING_SAME_TEST=NOT_JUSTIFIED
```

Timing does not rescue the approach: `time/N` was reported as flat or
decreasing because warmup effects dominate the small superlinear builder term.

## Test-policy consequence

The experiment is also unsuitable as an ordinary retained test.

At the largest attempted size it consumed tens of seconds and hundreds of
megabytes of allocation per parse. This conflicts with the repository's
test-discipline requirement that ordinary tests not become benchmark/stress
loads and with the general requirement to avoid disproportionately expensive
validation.

The experimental test therefore must not be retained merely as disabled or
flaky evidence.

```text
RETAIN_EXPENSIVE_TEST=NO
RUN_IT_AGAIN=NO
PROMOTE_TO_ORDINARY_TEST=NO
PROMOTE_TO_GENERIC_BENCH_INFRASTRUCTURE=NO
```

At the time the maintainer reported the result, the experimental file remained
uncommitted in the local working tree and had been marked intent-to-add by
`git add -N`. No product publication is associated with this experiment.
Cleanup belongs to the local maintainer checkout and does not itself require
another test execution.

## Options considered after the failed instrument

### A — isolate the existing builder in production structure

Extract the local builder closures into a private/internal product-level
mechanism and exercise that mechanism directly.

This would make the `N log N` term measurable with a much cleaner signal, but
it changes durable production structure at exactly the runtime/Core boundary
that A1 identified as unresolved.

It is not the linear repair itself, but it prejudges the architecture of the
future repair/evidence boundary.

```text
OPTION_A=TECHNICALLY_CREDIBLE
OPTION_A_ALLOWED_INSIDE_A2=NO
REASON=REQUIRES_PLATFORM_RUNTIME_ARCHITECTURE_APPROVAL
```

### B — external Java-agent / bytecode instrumentation

Instrument `ProtosArrayValue` construction or allocation externally without
changing production code.

This could count materializations and sizes, but it introduces substantial
special-purpose instrumentation infrastructure for one audit slice and does not
improve the product architecture.

```text
OPTION_B=TECHNICALLY_POSSIBLE
OPTION_B_RECOMMENDED=NO
REASON=DISPROPORTIONATE_ONE_OFF_INFRASTRUCTURE
```

### C — retain A1 static proof and block dynamic isolation on the decision gate

Keep the exact static proof as the current conformance evidence, retain this A2
record as evidence that complete-parse black-box scaling is not discriminating,
and defer isolated retained dynamic evidence until the platform/runtime boundary
is explicitly selected.

```text
OPTION_C=SELECTED_FOR_AUDIT_FLOW
A2_STATUS=BLOCKED_BY_PLATFORM_RUNTIME_DECISION
F1_STATIC_PROOF_REMAINS=Theta(N log N)
D115_REQUIREMENT_REMAINS=O(T + Svisited)
```

This is an audit-flow decision only. It does not select which runtime
construction mechanism will ultimately be approved.

## Required decision boundary

A1 already established that a genuine linear repair needs a private internal
linear Array construction mechanism or an equivalent architecture.

A2 adds a second requirement on that decision packet: the chosen architecture
should permit deterministic retained evidence of the accumulation cost without
forcing complete-parser benchmark/stress loads or global diagnostic authority.

That observability requirement is secondary to correctness. It must not select
the runtime mechanism by itself.

The platform/runtime investigation must compare at least:

- private append/finalize builder producing an ordinary standard Array;
- private exact-size allocation plus indexed fill;
- any existing internal construction mechanism that can be safely generalized;
- status quo / defer, explicitly acknowledging that D115 remains violated;
- other credible mechanisms found by exhaustive PLAT research.

It must also decide the ownership/lifetime/visibility boundary and verify:

```text
PUBLIC_ARRAY_API_CHANGE=NO_REQUIRED
PUBLIC_COMMANDLINE_API_CHANGE=NO_REQUIRED
COMMANDLINE_SEMANTICS_CHANGE=NO_REQUIRED
FINAL_RESULT_IS_ORDINARY_ARRAY=REQUIRED
ORDER_AND_IDENTITY_PRESERVED=REQUIRED
FREEZE_BEHAVIOR_PRESERVED=REQUIRED
NO_GLOBAL_PARSER_STATE=REQUIRED
NO_JAVA_ONLY_COMMANDLINE_PARSER=REQUIRED
NO_HIDDEN_GUEST_AUTHORITY=REQUIRED
NATIVE_IMAGE_AND_TRUFFLE_IMPACT=MUST_BE_RESEARCHED
RETAINED_COST_EVIDENCE=MUST_BE_POSSIBLE
```

## Current audit state

```text
AUD006_STATUS=IN_PROGRESS
AUD006_A1=COMPLETE
AUD006_A2=BLOCKED_BY_PLATFORM_RUNTIME_DECISION

F1_STATIC_PROOF=Theta(N log N)
F1_BLACK_BOX_FULL_PARSE_SCALING=NON_DISCRIMINATING

LINEAR_REPAIR=NOT_IMPLEMENTED
PLATFORM_RUNTIME_DECISION_REQUIRED=YES

AUD006_B=BLOCKED_BY_A
AUD006_C=READY_INDEPENDENT
AUD006_D=BLOCKED_BY_A_B_C

LIB011_ISSUE_428=KEEP_CLOSED
```

No additional execution of the failed A2 full-parse scaling instrument is
required or justified.
