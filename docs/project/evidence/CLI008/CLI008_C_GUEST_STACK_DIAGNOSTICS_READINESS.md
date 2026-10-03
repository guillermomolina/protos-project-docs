# CLI008-C guest stack diagnostics readiness checkpoint

**Work item:** CLI008-C / guillermomolina/protos#416  
**Parent:** CLI008 / guillermomolina/protos#312  
**Governing decision:** D063 / guillermomolina/protos#314 — Candidate B + S3  
**Protos revision inspected:** `12a42ff718144162ee72bb321b3eca7d70ce3cc9`  
**Checkpoint date:** 2026-10-03

## Status

CLI008-C is active and remains open. This record is a readiness/re-audit
checkpoint, not implementation or closure evidence.

The historical PERF006-B4/#397 and PERF006-B5/#398 blockers recorded by #416
are closed, so the next required work is the implementation re-audit mandated
by CLI008-C itself.

## Exact-revision evidence

At the inspected Protos revision:

- `src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java`
  carries the exact Error object and selected-handler control state, but no
  guest stack/source diagnostic payload.
- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` catches uncaught
  `ProtosSignalException` occurrences in REPL, `-e`, and direct-file paths and
  still emits only `Error: <diagnostic inspection>`; no guest-frame sequence is
  presented.
- No current product commit claims CLI008-C completion.
- D063 remains the public presentation authority: guest-only structured frames
  belong to the Error transfer/failure occurrence rather than guest-visible
  Error mutation; host/runtime/scheduler frames are excluded; async-origin
  segments are allowed only from trustworthy retained provenance; successful
  ordinary execution must not pay an always-on stack-capture tax.

These facts mean CLI008-C cannot be closed from existing publication evidence.

## Next bounded slice

The next slice is **CLI008-C1 — current-HEAD guest-stack implementation
re-audit**.

It is an investigation-only slice. It must determine whether the completed
logical execution, Error/unwind, source, instrumentation, and debugger
machinery already exposes enough truthful and bounded information to implement
D063 S3 mechanically.

The investigation must end in exactly one routing result:

1. `MECHANICAL_IMPLEMENTATION_READY` — existing authority is sufficient and a
   bounded implementation slice can be specified without a new architecture
   decision; or
2. `PLAT_DECISION_REQUIRED` — faithful capture still requires a durable
   Truffle/JVM/continuation/diagnostic-provenance choice, so CLI008-C
   implementation stops and the missing choice is routed through a PLATxxx
   decision.

The investigation must not choose a new durable platform architecture locally.

## Validation

No Protos product files were changed by this checkpoint and no product tests
were run. The evidence is derived from exact-revision repository inspection and
live GitHub coordination state.
