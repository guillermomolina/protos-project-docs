# I080 — D188 specification-reconciliation allocation evidence

Status: **ALLOCATED — READY**

Formal implementation work: `guillermomolina/protos#828` — I080

Decision authority: `guillermomolina/protos#819` — D188

Sibling next decision: `guillermomolina/protos#820` — D189

Allocation date: **2026-10-07**

## Allocation reason

D188 ratified Candidate C — hybrid Protos semantic projection — after explicit
project-owner approval.

The selected D188 contract changes observable Protos semantics and therefore
needs reconciliation into the normative `guillermomolina/protos:spec/`
authority. That mutation is implementation work rather than decision work.

The allocation follows the existing D180 -> I078 project pattern:

```text
Dxxx
  owns investigation + owner-approved decision + durable ratification

Ixxx
  owns specification / implementation reconciliation of the ratified decision
```

Therefore the previously proposed in-D188 implementation slice was corrected to
the implementation-family owner:

```text
FORMAL_IDENTIFIER=I080
ISSUE=guillermomolina/protos#828
TITLE=Reconcile D188 foreign-value semantics into normative specification
FAMILY=I
STATUS=READY
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
DECISION_AUTHORITY=D188/#819
```

## Allocation baseline

At I080 creation time:

```text
PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
PROTOS_VERSION=0.3.263-SNAPSHOT
```

That HEAD was `TEST009-AF: keep context projection failure out of PE`.
The immediately preceding D188-relevant revalidation had already established
that PLAT051-A2 does not opt foreign values into Actor/P transfer. No new D188
semantic decision was introduced by the allocation-time TEST009 change.

I080 must always execute from the actual current HEAD; this revision is
historical allocation identity only.

## Scope

I080 owns **normative specification reconciliation only**.

It must reconcile the ratified D188 contract into the correct current
`spec/` owners while preserving existing non-foreign semantics.

It does not implement:

```text
foreign import providers
java:/python:/js:/ruby: runtime support
std:interop implementation
foreign callback execution
HostAccess / sandbox authority
Context / Engine / provider topology
foreign Actor/P transfer
dependency acquisition
multi-language Native packaging
```

Those remain later AUD019 decision/implementation work.

## Live coordination result

After allocation:

```text
D188/#819=closed,status:completed
I080/#828=open,status:ready
D189/#820=open,status:ready
```

D189 is not blocked by I080. Its direct prerequisite is the ratified D188
decision, which is satisfied.

Foreign runtime/provider implementation remains unauthorized.

## Hierarchy note

I080 carries:

```text
Parent: #819
```

as textual cross-reference/bootstrap evidence.

Native GitHub Parent/Sub-issue hierarchy remains the canonical live hierarchy,
but the currently available connector surface does not expose the write action
needed to create that native parent relation. No false claim of native
reconciliation is made.

## Publication base

This evidence was prepared against project-record base:

```text
PROJECT_RECORD_BASE_REVISION=bf802aeda24c7939b26514b0a51dc205c710393f
```

The resulting project-record revision is recorded back in the live Issues after
publication.

## AI-assistance disclosure

This allocation/evidence record was materially prepared with AI assistance from
ChatGPT using the live Protos governance files, the D180/I078 precedent, the
ratified D188 contract, current live Issues, and the project owner's explicit
routing correction. No independent human review is claimed.
