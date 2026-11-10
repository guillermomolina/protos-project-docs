# D254-A — Preserve mandatory InvalidReturn for unreachable return homes

**Status:** OWNER-RATIFIED (2026-11-10). **No semantic change, no implementation authorization.**

- Authoritative decision: [D254 #1103](https://github.com/guillermomolina/protos/issues/1103).
- Owner's explicit selection (2026-11-10): “ok elijo A ... (que es sin cambios)”.
- This is a **non-normative** decision-history record. The existing product specification is unchanged and remains authoritative: [CALLABLES.md §§13–14](https://github.com/guillermomolina/protos/blob/main/spec/semantics/CALLABLES.md), [ERRORS.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/ERRORS.md), [EXECUTION_AND_CONTROL.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/EXECUTION_AND_CONTROL.md).
- [Approval and invariant evidence](../../evidence/D254/D254_A_OWNER_APPROVAL_AND_NO_CHANGE_RECONCILIATION_2026_11_10.md).

## Ratified choice

**A: Retain existing mandatory `InvalidReturn`.** A non-local `^value` (or contextual `return value`) targets the exact lexically captured return-home activation. A Closure invocation does not rebind that home. If that home is no longer reachable — including a completed home or a home owned by another Task — the operation signals the existing `InvalidReturn` error. It does not return locally, manufacture a substitute home, or invoke a new recovery message.

No `escapedBlock:`, `cannotReturn:`, substitute-value handler, new receiver/context authority, privileged selector, or new public protocol is introduced. Existing generic Error handling and `ensure` semantics remain exactly as specified; the decision does not prohibit catching an `InvalidReturn` using **already existing** Error-handling facilities, but it authorizes **no additional** recovery semantics.

## Explicitly preserved boundaries

- Lexical home identity and activation lifetime; nested Closure and bound/extracted method behavior.
- Task/Actor/P isolation; a foreign-Task home cannot receive cross-Task non-local control transfer.
- Suspension does not by itself complete/unwind the home; a reachable suspended home is governed by the existing lifetime and Task rules.
- Evaluation/side effects, standard Error propagation, deterministic `ensure` unwinding and precedence, cancellation, and ordinary Closure expression results.
- No new overhead, materialization, lookup, or dispatch is authorized on the ordinary call or valid non-local-return path.

## Consequences and separation of ownership

This resolves **only D254's policy question**. There is no product/specification delta, no new Ixxx, no new tests required *by this decision*, and no version bump. [I115 #931](https://github.com/guillermomolina/protos/issues/931) independently owns restoration or correction of any implementation regression against the **already existing** return-home contract; this ratification is not evidence that I115 is fixed or tested.

Reconsidering Smalltalk-style recovery later requires a new explicit owner decision, with complete receiver/payload/unwind/boundary consequences surfaced before changing the normative specification.
