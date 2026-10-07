# Protos Project Documentation — Agent Guidelines

## Repository purpose

This repository is the canonical durable, non-normative project-record store
for Protos.

Operational project authority remains in
[`guillermomolina/protos`](https://github.com/guillermomolina/protos).

## Authority delegation

Before performing substantive project work, follow the canonical project policy
in the root `AGENTS.md` of `guillermomolina/protos`.

This local file does not duplicate or supersede that policy.

In particular:

- do not allocate, select, close, or reinterpret formal Protos work identifiers
  independently in this repository;
- do not treat publication of a durable record here as project-owner approval;
- do not define normative Protos language or Standard Library semantics here;
- do not move live Issue, scheduling, priority, assignment, or approval
  authority into this repository;
- when a substantive decision is required, coordinate and obtain approval
  through the authoritative workflow in `guillermomolina/protos`.

## Direct repository publication

This repository is the sole Protos repository with standing authorization for
agent-direct repository-content publication.

When the active task requires a bounded change here, an authorized agent SHOULD
perform the repository edit and its normal commit/push directly through the
available repository publication mechanism. Do not hand the maintainer shell
commands or Markdown ZIPs merely so the maintainer can apply, commit, or push
this repository's documentation change.

This standing authorization changes the execution/handoff boundary only. It does
not authorize unrelated edits, semantic decisions, formal-work allocation,
approval claims, destructive history rewrites, force-pushes, or bypassing the
authority and validation rules delegated from `guillermomolina/protos`.

## Local documentation rules

The canonical durable corpus is under `docs/project/**`.

Preserve the role-first information architecture documented in
`docs/project/README.md` and its associated path contract.

Distinguish maintained references from historical evidence:

- maintained references may be updated when repository locations or current
  navigation change;
- literal historical paths, snapshots, old migration records, and other
  evidence must not be mechanically rewritten when the old spelling is part of
  the evidence.

Do not create duplicate authoritative copies of records that already have a
canonical location.

When a record is revision-coupled to Protos product state, use exact commit
revisions rather than relying only on moving branch references.

## Validation

Documentation-only changes must at minimum run:

```text
git diff --check
```

Run any additional link, reference, or repository-policy checks required by the
corresponding work item in `guillermomolina/protos`.
