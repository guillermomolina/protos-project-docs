# TOOL001-F2E4 activation checkpoint

Date: 2026-10-03

Nature: immutable coordination/evidence snapshot; non-normative

Formal owner: `TOOL001-F2E4` / `guillermomolina/protos#92`

## Purpose

Retain the exact coordination state that reactivates the final external
immutable-package execution sequence after the earlier F2E3 and PLAT012
closures. Live scheduling authority remains in GitHub Issues/Project; this
record does not own status or priority.

## Exact authorities and repository state

- F2E3 closure revision: `5d797f89cb6f6189165f9f40ac812ddacbead03b`.
- PLAT012 durable ratification revision:
  `6dc24c0f37363f8a5603c59f3fc6661db763f1b6`.
- Protos `main` observed for the durable activation publication:
  `12a42ff718144162ee72bb321b3eca7d70ce3cc9`.
- Live implementation owner: `guillermomolina/protos#92`.
- Dependency-gated successor: `guillermomolina/protos#93`.
- Phase owner: `guillermomolina/protos#412`.
- Root TOOL001 coordination: `guillermomolina/protos#47`.

An earlier #92 activation comment quoted the GitHub code-search indexed
revision `1e8fbb27ee3966ccc48a57e04308c17a58995bdc`. A follow-up comment corrected
the repository `main` revision to the exact value above. The implementation
agent must still begin from whatever current repository HEAD exists when work
starts; this checkpoint does not freeze a future implementation base.

## Live state at activation

The project owner selected F2E4 as the next/high-priority TOOL001 continuation.
GitHub live coordination was reconciled to:

```text
TOOL001-F2E3   CLOSED
TOOL001-F2E4   IN_PROGRESS / priority:p1 / owner guillermomolina
TOOL001-F2E5   BLOCKED_BY_DEPENDENCY: TOOL001-F2E4
```

The P1 label is a temporary scheduling promotion from F2E4's prior P3 state.
It does not alter Package Tool semantics, PLAT012 architecture or F2E5's
independent blocked state.

## Preserved implementation boundary

F2E4 remains mechanical implementation under ratified D053 and PLAT012 A+:

- defensively detach generation-2 `PackageExecutionPlan` data;
- use complete exact external package identity as the host authority key;
- define canonical external `ModuleKey` as exact immutable package instance plus
  internal logical module;
- reconcile detached external package identities 1:1 with already-verified
  run-local custody;
- load source lazily from the same verified immutable backing through a narrow
  host-only resource projection;
- construct source path-independently through
  `ProtosModuleSource.fromCharacters(...)`;
- do not reopen source/store paths, solve/fetch/mutate lock state, introduce a
  global custody registry, or create a guest Process/Actor/Activation merely to
  read immutable source bytes; and
- leave public run wiring and final run teardown integration to F2E5.

Any newly exposed substantive semantic or durable architecture choice remains a
new Dxxx/PLATxxx gate rather than being inferred inside F2E4.

## Evidence boundary

This checkpoint records coordination only. It claims no F2E4 product change,
no validation result, no implementation version and no closure evidence.
F2E5 remains blocked until F2E4 is implemented, published, validated and closed
under its GitHub closure rule.
