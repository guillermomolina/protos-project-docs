# TEST009-C stale self-test reconciliation

Date: 2026-10-04

## Publication

```text
REVISION=9e8e7bf2b89050f1a6749960b606c4bf3be03845
COMMIT=TEST009-C: fix stale Bytecode API guard self-test
PARENT_ISSUE=TEST009/#795
SEMANTIC_CHANGE=NO
```

## Cause

The final TEST009-C publication `bb3a15b2c529368879eeed527356bac0bb9e7d6e` correctly expanded the Bytecode API PE guard so that:

```text
BytecodeNode.getLocalValue(int, Frame, int)
```

is a guarded sink whose bytecode index and local offset must be PE constants.

An older self-test, `test_18_overload_and_receiver_type_resolution`, still contained:

```java
bytecode.getLocalValue(tag.getEnterBytecodeIndex(), null, 0);
```

while asserting that the snippet produced no sinks. That expectation had become stale: the call is now intentionally a real sink.

The guard itself, its production baseline, and its imports were unchanged.

## Repair

The stale `getLocalValue(...)` line was removed from the overload/receiver-type test.

No guard logic was weakened and no production Java changed. Dedicated TEST009-C tests continue to own positive/risk coverage for `getLocalValue`.

Published diff:

```text
tools/test_java_bytecode_api_pe_guard.py
  additions=0
  deletions=1
```

## Validation

The maintainer reports:

```text
BYTECODE_API_SELF_TESTS=PASS
BYTECODE_API_WHOLE_SOURCE_GUARD=PASS
MAKE_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

This is maintainer-reported validation; the assistant did not execute the local suite.

## Result

```text
TEST009_C=COMPLETE
TEST009_C_GUARD_LOGIC=UNCHANGED
TEST009_C_PRODUCTION_REPAIR=UNCHANGED
STALE_SELF_TEST=FIXED
SEMANTIC_CHANGE=NO
```

The successful aggregate `make check` also removes the unrelated validation blockage that had prevented final publication evidence for TEST009-D.
