# AUD006-B1 — CommandLine depth/recursion remediation analysis

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
SLICE=AUD006-B1
SLICE_TYPE=INVESTIGATION
STATUS=COMPLETE

ANALYSIS_BASE=845a1103b031abcf95d8ba852e0d780ba6b6591a
ANALYSIS_BASE_SUBJECT=AUD006-A4: make CommandLine accumulation linear

RECONCILED_PRODUCT_HEAD=62f3f5710210f247aad8574d3d0d56d3254bfd70
RECONCILED_PRODUCT_HEAD_SUBJECT=LIB010-E2-A: make TOML temporal fraction encoding linear
HEAD_RECONCILIATION=PASS

F2_STILL_PRESENT=YES
CANONICALIZATION_STACK_DEPTH_RISK=YES_STATIC
PARSE_STACK_DEPTH_RISK=YES_STATIC
OTHER_COMMANDLINE_DEPTH_RISK=NONE_FOUND

CANONICALIZATION_MECHANICAL_ITERATIVE_REPAIR=YES
PARSE_MECHANICAL_ITERATIVE_REPAIR=YES

NEW_SEMANTIC_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
PUBLIC_DEPTH_LIMIT_REQUIRED=NO
RETAINED_DYNAMIC_EVIDENCE_REQUIRED=YES

RECOMMENDED_NEXT_SLICE=AUD006-B2
RECOMMENDED_NEXT_SLICE_TYPE=IMPLEMENTATION
RECOMMENDED_IMPLEMENTATION_REPOSITORY=guillermomolina/protos
OWNER_APPROVAL_REQUIRED=NO

AUD006_B_STATUS=IN_PROGRESS
LIB011_ISSUE_428=KEEP_CLOSED
```

This is durable non-normative investigation evidence for AUD006-B1. It records
the static depth/recursion analysis and the implementation boundary for F2. It
does not define Protos language or Standard Library semantics.

## Moving-HEAD reconciliation

B1 was investigated at:

```text
845a1103b031abcf95d8ba852e0d780ba6b6591a
AUD006-A4: make CommandLine accumulation linear
```

Before publication, current `guillermomolina/protos:main` was:

```text
62f3f5710210f247aad8574d3d0d56d3254bfd70
LIB010-E2-A: make TOML temporal fraction encoding linear
```

The two intervening commits are:

1. `b6f62a1f52a4d26cd026beb69d3c28a6730c83b7` —
   `TEST009-T: add causal compilerability acquisition`. It adds diagnostic-only,
   property-gated compilerability observation. The property is absent from
   ordinary execution and the strict compilation gate. It does not modify the
   CommandLine source or tests.
2. `62f3f5710210f247aad8574d3d0d56d3254bfd70` —
   `LIB010-E2-A: make TOML temporal fraction encoding linear`. It changes TOML
   encoding/tests and release bookkeeping only.

The directly relevant CommandLine blobs are byte-identical between the analysis
base and reconciled HEAD:

```text
protos/lib/cli/CommandLine.protos
  ea1a1acf5e80cdcaad0931311a782146fb8e9686

src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineSpecModuleTest.java
  7f1a9401b51c010daa059465fcd3d0d7e9361088

src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineResultModelTest.java
  db1ff8ad46359ac0972cb808cf0bc648fa1ab72d

protos/tests/library/cli/specification-and-result.protos
  c0e99bd689ff2e6880e463a857adf74024774228

protos/tests/library/cli/subcommands.protos
  40ac0805364ff5dbe3f14f0fd69e3780bc607357
```

Therefore all B1 conclusions remain valid at the reconciled product HEAD.

## Governing boundary

AUD006-A is complete and F1 is resolved by A4. B1 does not reopen D111, D115,
D118, D119, AUD006-A or LIB011/#428.

The current F2 owner remains:

```text
command-tree depth remains host/guest-stack proportional
```

A semantics-preserving mechanical conversion may proceed. A public maximum
depth or another observable policy remains design-gated.

## Exact proportional-depth recursion inventory

B1 finds exactly two product recursion sites relevant to CommandLine tree depth.

### Canonicalization

`CommandLine.command(...)` defines `canonicalizeCommand(candidate)` and calls
it recursively for child descriptors.

The recursive call is not in tail position. After a child returns, the parent
still performs duplicate child-name detection, name registration, child result
storage, sibling advancement, subcommand Array freeze, active-path removal,
parent result construction and parent result freeze.

Therefore command canonicalization consumes execution-stack depth proportional
to nested command depth under the current implementation.

### Selected-child parse traversal

`CommandLine.parse(...)` defines `parseScope(scopeSpec, startIndex,
commandTokenIndex)` and recursively enters the selected child.

This recursive call is also non-tail: after the child returns, the parent still
constructs and freezes its `commandResult` with the returned child result in
the `subcommand` field.

Therefore selected parse traversal consumes execution-stack depth proportional
to the selected child path.

### No third relevant recursion found

`renderHelp` walks `commandPath` iteratively and selects children through
iteration. Ordinary shallow `freeze()` does not recursively walk the command
graph. Options, positionals and current parse allocation helpers do not add
another CommandLine tree-depth recursion.

```text
OTHER_COMMANDLINE_DEPTH_RISK=NONE_FOUND
```

## Canonicalization invariants

The current `visiting: IdentityMap()` is an active-path identity set, not a
global visited set.

The semantic distinction is:

```text
ancestor identity reached while still active
    -> reject cycle

same descriptor identity reached after a completed branch
    -> permitted and canonicalized again
```

Consequences:

- direct self-cycles fail;
- indirect ancestor cycles fail;
- descriptor reuse in different completed branches is valid;
- no identity memoization occurs; and
- every valid occurrence receives a fresh canonical result.

A repeated sibling descriptor is canonicalized again before the parent performs
the existing duplicate-child-name failure check. An iterative repair must not
move that failure earlier.

The per-command ordering to preserve is:

```text
active-path cycle check
add candidate to visiting

validate name/help/options/positionals/subcommands

canonicalize options in declaration order
  duplicate key / long / short checks
freeze canonical options

canonicalize positionals in declaration order
  duplicate key check
  unlimited positional only last
freeze canonical positionals

for each child in declaration order:
  canonicalize child completely
  duplicate child-name check
  record child name
  store canonical child

freeze canonical subcommands
remove candidate from visiting
construct canonical command
freeze canonical command
```

## Parse invariants

The current D115 traversal rules remain authoritative and unchanged:

```text
earliest feasible exact-child boundary
parent-minimum reservation
literal option-value ownership
current-scope -- escape
irreversible child scope transfer
```

Before descending into a selected child, the parent scope has already completed
its local token scan, minima validation, positional allocation, option
materialization and occurrence freezing.

The state that must survive descent is only the state necessary to reconstruct
the parent result after the child completes:

```text
name
commandTokenIndex
endOfOptionsIndex
optionOccurrences
positionalOccurrences
parent link
```

The recursive result shape does not require a recursive execution stack. The
deepest result can be built first and parent results reconstructed bottom-up.

## Runtime / Truffle classification

Both sites are ordinary nested Protos Closure invocations. Neither recursive
call is structurally tail-recursive.

B1 found no normative/runtime guarantee that arbitrary Closure recursion is
eliminated or transformed into stack-constant execution. Truffle compilation
and inlining are implementation optimizations and cannot be used as a
CommandLine depth guarantee.

Therefore B1 distinguishes:

```text
STATIC_FACT:
  execution-stack consumption grows with command depth

DYNAMIC_FACT:
  exact failure depth N on environment X
  NOT ESTABLISHED BY B1
```

No numeric safe depth is claimed by this investigation.

## Complexity

Let `D` be canonical command depth, `Ds` selected parse depth, `S` expanded
command/specification surface, and `T` supplied argument-token count.

Current canonicalization:

```text
time:             O(S)
heap:             O(result + active per-level state)
execution stack:  O(D)
```

Expected iterative canonicalization:

```text
time:             O(S)
heap:             O(result + explicit active frames)
execution stack:  O(1) with respect to D
```

Current parse after A4:

```text
time:             O(T + Svisited)
heap:             O(result + current/visited scope state)
execution stack:  O(Ds)
```

Expected iterative parse:

```text
time:             O(T + Svisited)
heap:             O(result + explicit parent frames + current scope state)
execution stack:  O(1) with respect to Ds
```

Replacing implicit stack `O(depth)` with explicit heap state `O(depth)` is
the intended repair; the recursive result shape already requires depth-related
result memory.

## Selected implementation family

For both recursion sites, B1 selects an iterative linked-frame state machine
implemented entirely in Protos.

Canonicalization uses conceptual phases:

```text
ENTER
CHILD
AFTER_CHILD
EXIT
```

Each frame links to its parent and retains the current candidate, phase,
child index and partial canonicalization state. `AFTER_CHILD` preserves the
existing post-child duplicate-name check and fresh-per-occurrence behavior.

The parser can process one scope completely, link a small completion frame when
a child is selected, continue with the child, then reconstruct
`commandResult` values from deepest child back to root.

No new public Array stack API or Java/runtime facility is needed.

## Rejected alternatives

B1 rejects:

- a new Java/runtime stack facility, because current Protos mechanisms are
  sufficient;
- a public maximum depth, because no normative rule requires one and an
  iterative repair is available;
- retaining recursion merely because normal CLIs are usually shallow; and
- global descriptor memoization, because it would alter active-path cycle
  semantics and canonical-result freshness.

## Required retained evidence

B2 must retain a deterministic canonicalization-depth regression. It must:

- construct a one-child descriptor chain iteratively;
- invoke the real `CommandLine.command(root)`;
- walk the canonical result iteratively;
- verify exact depth/leaf behavior;
- preserve self-cycle rejection;
- add indirect ancestor-cycle coverage if missing;
- prove descriptor reuse in different completed branches remains valid; and
- prove distinct fresh canonical results for reused occurrences.

A scale such as 4096 levels is a test scale, not a public limit. If the
pre-repair implementation still passes that scale, the implementation work may
increase the scale systematically while keeping the retained test cheap. No
safe-depth number is established by B1.

B3 must separately retain a deep selected-child parse regression after B2 makes
construction of a legitimately canonical deep tree independent of the old
canonicalization recursion.

The evidence must be deterministic, non-timing-based and non-benchmark.

## Recommended decomposition

```text
AUD006-B1 = COMPLETE investigation
AUD006-B2 = iterative command canonicalization + retained depth/cycle/freshness evidence
AUD006-B3 = iterative selected-child parse traversal + retained D111/D115/depth evidence
```

Dependency order:

```text
B2 -> B3
```

B2 must precede B3 so that the parse-depth test can obtain a canonical deep
CommandSpec through the public constructor without the old canonicalization
recursion masking the parse result.

No separate B4 implementation slice is currently required. Cross-slice
reconciliation can be performed at B3 closure unless new evidence appears.

## Design-gate result

```text
NEW_SEMANTIC_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
PUBLIC_DEPTH_LIMIT_REQUIRED=NO
OWNER_APPROVAL_REQUIRED=NO
```

AUD006-B may therefore proceed as semantics-preserving mechanical
implementation.

## Next slice

```text
AUD006-B2
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
PURPOSE=eliminate canonicalization execution-stack depth with linked iterative frames
```

B2 must stop rather than decide if implementation would require a public depth
policy, new runtime/native facility, public collection API, changed cycle
semantics, changed canonical freshness, changed failure/evaluation ordering or
a change to D111/D115/D118/D119.
