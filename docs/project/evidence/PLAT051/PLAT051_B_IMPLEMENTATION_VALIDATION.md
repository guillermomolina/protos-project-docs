# PLAT051-B — Regex semantic-transfer implementation validation

Status: **COMPLETED AND PUBLISHED**

Evidence date: **2026-10-07**

Decision Issue: `guillermomolina/protos#813` — PLAT051

Parent workstream: `guillermomolina/protos#431` — LIB014

Governing semantic authority: `guillermomolina/protos#809` — D187.

## Published product revision

~~~text
PUBLISHED_SHA=b3d85c1deee91455c2777d0025a19ec3970530ac
COMMIT_MESSAGE=PLAT051-B: opt std:regex/Regex Pattern and Match into semantic transfer
IMPLEMENTATION_VERSION=0.3.264-SNAPSHOT
~~~

At evidence capture time this revision is the live `main` HEAD of `guillermomolina/protos`.

## Human-executor validation

After publication, the project owner explicitly reported:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
~~~

No unreported build, benchmark or standalone Native Image package command is claimed by this record.

## One exact Regex semantic-transfer family

The published implementation introduces one production descriptor:

~~~text
ProtosRegexSemanticTransferFamily
OWNER_MODULE=std:regex/Regex
~~~

Core bootstrap registers the family once. Pattern and Match share this one descriptor; their payload discriminator is data rather than authority.

The family is still governed by the PLAT051 bootstrap identity rule. An ordinary look-alike object or a second descriptor with the same module key does not gain authority.

~~~text
REGEX_FAMILY_REGISTERED_EXACTLY_ONCE=PASS
STANDARD_LIBRARY_MEMBERSHIP_ALONE_IMPLIES_PORTABILITY=NO
FORGERY_NEGATIVE_GATE=PASS
~~~

## Trusted private minting

`Regex.protos` now mints its Pattern and Match objects through the private bootstrap facility owned by the exact Regex family.

The facility is captured during exact `std:regex/Regex` module initialization and removed from the published module surface.

Pattern/Match remain observably ordinary frozen Protos objects. Their public member sets and behavior remain under `Regex.protos`; the host facility does not implement the regex language or matcher.

~~~text
PATTERN_TRUSTED_MINTING=PASS
MATCH_TRUSTED_MINTING=PASS
PUBLIC_REGEX_SURFACE_DELTA=NONE
JAVA_PARALLEL_REGEX_LIBRARY=NO
~~~

## Pattern portable representation

Pattern source-stage payload is exactly:

~~~text
["Pattern", source, canonicalFlags]
~~~

The destination materializer invokes the destination Actor/P domain's own Regex module and recompiles from the semantic source and canonical flags.

The payload excludes:

- the Pike program;
- compiled node tables;
- matcher execution state;
- source Closures;
- source lexical/module execution contexts.

Therefore:

~~~text
PATTERN_PAYLOAD=SOURCE_PLUS_CANONICAL_FLAGS_ONLY
PATTERN_DESTINATION_RECOMPILE=PASS
PATTERN_SOURCE_PIKE_TRANSFER=NO
PATTERN_SOURCE_CLOSURE_TRANSFER=NO
~~~

## Match portable representation

Match source-stage payload is a result snapshot containing:

~~~text
kind = "Match"
captureCount
groups:
    [false]
    or [true, capturedText, scalarStart, scalarEnd]
capture-name -> group-number pairs
~~~

Group 0 is required to participate. Nonparticipating captures remain distinct from participating empty captures.

The destination does not run the matcher again. It reconstructs the Match from the result metadata through the destination Regex module's private local factory.

The payload excludes:

- the whole subject merely for convenience;
- the Pattern/Pike implementation;
- matcher execution state;
- source Match Closures.

~~~text
MATCH_PAYLOAD=CAPTURES_BOUNDS_COUNT_NAMES
MATCH_DESTINATION_CONSTRUCTION=PASS
MATCH_REMATCH=NO
MATCH_WHOLE_SUBJECT_TRANSFER=NO
MATCH_SOURCE_PIKE_TRANSFER=NO
MATCH_SOURCE_CLOSURE_TRANSFER=NO
~~~

## Destination-local guest factory

PLAT051-B adds the minimal generic destination-module seam required by the amended A2 architecture.

During exact Regex module initialization, the private transfer facility installs the module's guest factory in that Actor-local module record. `ProtosSemanticTransferDestination` exposes only the owning family's module-local factory to materialization.

The factory is destination-local guest code, not a source Closure and not public module surface.

~~~text
DESTINATION_LOCAL_GUEST_FACTORY=PASS
GLOBAL_MUTABLE_FACTORY_REGISTRY=NO
PUBLIC_DESERIALIZATION_HOOK=NO
~~~

## Actor and P portability

The retained `ProtosRegexSemanticTransferTest` exercises real production Pattern/Match values through the hosted execution boundary.

Coverage includes:

- Actor spawn;
- Actor request arguments;
- Actor request replies back into the requester domain;
- Pattern and Match as isolated-P inputs;
- Pattern and Match returned from isolated P and rematerialized back in the caller domain;
- fresh destination identities;
- repeated aliases remaining aliases;
- separate equal-content Pattern/Match identities remaining distinct.

~~~text
ACTOR_PATTERN_TRANSFER=PASS
ACTOR_MATCH_TRANSFER=PASS
P_PATTERN_TRANSFER=PASS
P_MATCH_TRANSFER=PASS
REQUEST_REPLY_MATERIALIZATION=PASS
P_RESULT_MATERIALIZATION=PASS
ALIAS_PRESERVATION=PASS
DISTINCT_IDENTITY_PRESERVATION=PASS
SOURCE_DESTINATION_IDENTITY_DIFFERENT=PASS
~~~

## Regex semantic behavior retained

The published tests verify destination-reconstructed Pattern behavior including flags, named/numbered captures, Unicode scalar behavior, search and replacement operations.

Match transfer retains:

- `captureCount`;
- group 0;
- numbered and named captures;
- `start` / `end`;
- nonparticipating capture nulls;
- participating empty captures;
- repeated capture selection;
- `groups()`;
- `namedGroups()`;
- Unicode-scalar offsets.

No Regex syntax, priority, Unicode, replacement, split or complexity contract changes are introduced.

~~~text
D187_DELTA=NONE
REGEX_SEMANTIC_DELTA=NONE
SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

## Pay-as-you-grow and generic transfer boundary

PLAT051-B adds no Regex branch to the generic Actor/P copiers.

The retained tests verify ordinary Actor/P transfer can occur without loading `std:regex/Regex`.

~~~text
REGEX_BRANCH_IN_GENERIC_COPIERS=NO
ORDINARY_TRANSFER_REGEX_COST_DELTA=NONE
ORDINARY_TRANSFER_REGEX_IMPORT=NO
ORDINARY_TRANSFER_REGEX_SOURCE_EXECUTION=NO
GLOBAL_MUTABLE_REGISTRY=NO
PAY_AS_YOU_GROW=PASS_STRUCTURAL
~~~

## Native / portability boundary

The change includes the repository's native-boundary architecture gate updates and introduces no Java serialization, reflection-driven family lookup, `Class.forName`, `ServiceLoader`, JNI requirement or dynamic class generation.

The project owner reports the complete local suite passing. A separate standalone Native Image package build is not independently claimed by this record.

## PLAT051 completion

With A, A2 and B published and validated:

~~~text
PLAT051_A=COMPLETE
PLAT051_A2=COMPLETE
PLAT051_B=COMPLETE

PLAT051_IMPLEMENTATION=COMPLETE
D187_ACTOR_P_PORTABILITY=COMPLETE
PROCESS_WIRE_FORMAT=DEFERRED_BY_DECISION
NEXT_REQUIRED_PLAT051_SLICE=NONE
~~~

No Process wire-format work is implied by closure of PLAT051.

## LIB014 release

PLAT051-B resolves the final required portability blocker identified by LIB014.

~~~text
LIB014_FUNCTIONAL_BASELINE=COMPLETE
LIB014_FINAL_PORTABILITY_BLOCKER=RESOLVED
LIB014=CLOSABLE
LIB014_4=OPTIONAL
LIB014_4_TRIGGER=MEASURED_PERFORMANCE_EVIDENCE_ONLY
~~~

No acceleration slice is justified merely because the baseline work is complete.

## Result

~~~text
PLAT051_B_IMPLEMENTATION=PUBLISHED_AND_VALIDATED
PUBLISHED_SHA=b3d85c1deee91455c2777d0025a19ec3970530ac
IMPLEMENTATION_VERSION=0.3.264-SNAPSHOT

GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
ALL_LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

REGEX_TRANSFER_FAMILY=PASS
REGEX_FAMILY_OWNER=std:regex/Regex
REGEX_FAMILY_REGISTERED_EXACTLY_ONCE=PASS
PATTERN_TRUSTED_MINTING=PASS
MATCH_TRUSTED_MINTING=PASS
PUBLIC_REGEX_SURFACE_DELTA=NONE

PATTERN_PAYLOAD=SOURCE_PLUS_CANONICAL_FLAGS_ONLY
PATTERN_DESTINATION_RECOMPILE=PASS
PATTERN_SOURCE_PIKE_TRANSFER=NO
PATTERN_SOURCE_CLOSURE_TRANSFER=NO

MATCH_PAYLOAD=CAPTURES_BOUNDS_COUNT_NAMES
MATCH_DESTINATION_CONSTRUCTION=PASS
MATCH_REMATCH=NO
MATCH_WHOLE_SUBJECT_TRANSFER=NO
MATCH_SOURCE_PIKE_TRANSFER=NO
MATCH_SOURCE_CLOSURE_TRANSFER=NO

ACTOR_PATTERN_TRANSFER=PASS
ACTOR_MATCH_TRANSFER=PASS
P_PATTERN_TRANSFER=PASS
P_MATCH_TRANSFER=PASS
REQUEST_REPLY_MATERIALIZATION=PASS
P_RESULT_MATERIALIZATION=PASS

ALIAS_PRESERVATION=PASS
DISTINCT_IDENTITY_PRESERVATION=PASS
SOURCE_DESTINATION_IDENTITY_DIFFERENT=PASS
FORGERY_NEGATIVE_GATE=PASS
ARBITRARY_CLOSURE_TRANSFER=NO
CAPABILITY_RULE_DELTA=NONE

REGEX_BRANCH_IN_GENERIC_COPIERS=NO
ORDINARY_TRANSFER_REGEX_COST_DELTA=NONE
GLOBAL_MUTABLE_REGISTRY=NO
PAY_AS_YOU_GROW=PASS_STRUCTURAL

D187_DELTA=NONE
SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

NEXT_REQUIRED_PLAT051_SLICE=NONE
LIB014_FINAL_PORTABILITY_BLOCKER=RESOLVED
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from ChatGPT using the published PLAT051-B revision, the maintained PLAT051/D187 records, current GitHub coordination state, and the project owner's explicit human-executor validation report. No unreported execution result is claimed.
