# D254-A — Owner approval and no-change invariant reconciliation (2026-11-10)

## Provenance

The owner explicitly selected **A** in the ongoing D254 conversation: “ok elijo A para [D254 … #1103] (que es sin cambios)” and authorized live GitHub issue coordination and commit/push of durable documentation in `guillermomolina/protos-project-docs`.

The selected option is the exact A candidate already published in [D254 #1103](https://github.com/guillermomolina/protos/issues/1103): retain mandatory `InvalidReturn` for escaped and foreign-Task homes, without changed semantics. This record documents that approval, not a claim that the originally requested exhaustive GITHUB010 comparative study or 12-dimensional scoring was performed in this action.

## Evidence inspected

- D254 #1103's candidate definitions, normative links and prior-art references.
- `guillermomolina/protos/AGENTS.md` and `AGENTS.work/DESIGN.md`, `AGENTS.work/COORDINATION.md`; documentation repository `AGENTS.md`.
- Current normative `spec/semantics/CALLABLES.md` §§13–14: lexical return-home identity, escaped completed home => `InvalidReturn`, foreign-Task home => `InvalidReturn`, `return` sugar equivalence.
- Current normative `spec/semantics/EXECUTION_AND_CONTROL.md` § `ensure`: cleanup timing/transfer precedence, suspension distinct from unwind.
- Current normative `spec/semantics/ERRORS.md`: `InvalidReturn` Error family and existing handler semantics.
- The decision and evidence role-first navigation convention in `guillermomolina/protos-project-docs`.

## GITHUB021-style invariants/delta accounting for choice A

| Existing invariant | D254-A result |
| --- | --- |
| Captured lexical home is the unique `^` return target | Preserved without reinterpretation |
| Completed home is unreachable | Existing `InvalidReturn` preserved |
| Foreign-Task home is unreachable | Existing `InvalidReturn` preserved |
| No switch to ordinary local Closure return | Preserved |
| `return value` is equivalent to `^value` | Preserved |
| Reachable home lifecycle, suspension and cancellation | No change |
| `ensure` cleanup and precedence | No change |
| Ordinary Error-handler behavior | No change; no novel recovery protocol |
| Actor/P isolation and receiver/capture identity | No change |
| Hot-path overhead and portability | No additional mechanism authorized |

**Observable semantic delta: NONE.** No previously ratified invariant is reopened; no new receiver, selector, recovery payload, substitute result, or crossing of Task boundaries is approved. No normative Protos file is edited by this record.

## Implementation boundary and validation

[I115 #931](https://github.com/guillermomolina/protos/issues/931) must separately restore the already specified behavior if its implementation diverges. D254-A does not certify its regression status. No builds, tests, runtime validation, or performance measurements were executed for this documentary decision. Closing D254 signifies **owner-settled no-change design**, not an implementation test pass.

## Prior-art distinction retained, not adopted

D254's cited TruffleSOM/SOMns original-receiver `escapedBlock:` recovery and TruffleSqueak context-oriented `cannotReturn:` are materially different mechanisms. Both are deliberately **not** adopted. This decision does not assert exhaustive cross-language research was completed.
