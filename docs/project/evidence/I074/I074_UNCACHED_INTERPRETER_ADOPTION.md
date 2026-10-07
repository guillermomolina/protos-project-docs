# I074 — Bytecode DSL uncached-interpreter adoption

Status: **COMPLETE**

This durable, non-normative record retains the exact publication and validation
evidence for I074 / guillermomolina/protos#745.

## Identity

```text
DATE=2026-09-30
WORK_ITEM=I074
PARENT=PERF011 / guillermomolina/protos#693

PROTOS_REVISION=a89a8897ea20b10344785ee9573ed329d188ef28
PARENT_REVISION=41a7a6e0bc08ef3d29f8f591c48af5e45821900e
IMPLEMENTATION_VERSION=0.3.124-SNAPSHOT
```

## Published implementation

The substantive runtime change is exactly one line in
`ProtosBytecodeRootNode`:

```text
+ enableUncachedInterpreter = true
```

The same publication correctly includes the required implementation metadata:

```text
pom.xml      0.3.123-SNAPSHOT -> 0.3.124-SNAPSHOT
CHANGELOG.md I074 entry present
```

The annotation processor accepted every existing Bytecode DSL operation
unchanged.

```text
UNCACHED_INTERPRETER=ENABLED
UNCACHED_COMPATIBILITY_CHANGES=NONE
FORCE_CACHED_USAGE=NONE
SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

## Validation

The project owner reported:

```text
1264 passed, 0 failed
```

for the I074 candidate.

Reported full-suite wall observations:

```text
BASE    80 s
I073    88 s
I074    73 s
```

These are single observations from sequential repository states, not a controlled
performance experiment. They are retained only as lightweight directional
sanity evidence.

No performance claim is made from these numbers. In particular, the spread
between 80 s, 88 s and 73 s demonstrates that one-run suite timing is too noisy
and confounded to attribute the I074 change causally.

## Closure

```text
UNCACHED_INTERPRETER=ENABLED
COMMON_COLD_EXECUTION=ACCEPTED_BY_GENERATOR
FORCE_CACHED_USAGE=NONE
OBSERVABLE_PROTOS_SEMANTICS=UNCHANGED
REQUIRED_VALIDATION=PASS
CLEAR_MATERIAL_REGRESSION=NOT_ESTABLISHED
PUBLICATION=PASS
I074_CLOSURE=PASS
```

## Routing

```text
I073/#744=CLOSED
I074/#745=CLOSED
I075/#746=READY

NEXT_SLICE=I075-A
NEXT_SLICE_TYPE=INVESTIGATION_ONLY
NEXT_REPOSITORY=NONE
```

I075-A must determine whether Protos has a clean, semantics-invisible primitive
carrier path that makes Bytecode DSL boxing elimination useful before any
implementation is attempted.
