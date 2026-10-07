# AUD019 — Intake and Project routing evidence

Evidence date: **2026-10-07**

Issue: `guillermomolina/protos#818` — **AUD019 — Foreign-library interop and polyglot import architecture audit**

This record is coordination/intake evidence only. It is **not** the AUD019 research result and does not select any interop/import architecture.

## Requested audit boundary

AUD019 is research-only and investigates one coherent foreign-interoperability architecture spanning:

- a first-class explicit `std:interop`-style lower-level surface; and
- ergonomic foreign-library imports whose result can be consumed through a natural Protos-facing surface where semantics permit.

No implementation, parser/module change, HostAccess broadening, dependency-manager change, FFI selection or new public API is authorized by the issue itself.

## Owner scope correction retained

Issue comment `6029778896` clarified that Protos-written bindings/providers are optional rather than preferred or required.

The research target is:

```text
USER_SURFACE=ordinary import(...) + natural Protos-facing use
IMPLEMENTATION=smallest faithful/efficient bridge regardless of implementation language

PROTOS_WRITTEN_BINDINGS_REQUIRED=NO
PROTOS_WRITTEN_BINDINGS_ALLOWED=YES
PROTOS_WRITTEN_BINDINGS_PREFERRED=NOT_PRESELECTED
RUNTIME_OR_GENERATED_SHIM_ALLOWED=YES
SYNTHETIC_PROTOS_FACING_FOREIGN_FACADE=FIRST_CLASS_CANDIDATE
```

## Invalid prior-reference correction retained

Issue comment `6029818395` identifies two issue-authoring references that are invalid and must not be used as authority:

```text
PLAT410 / #659 = INVALID_REFERENCE
PLAT411 / #660 = INVALID_REFERENCE
```

`#659` and `#660` are unrelated work. AUD019 must discover real prior NFI/Sulong/native/polyglot/interop authority from current repository state and durable records. `PLAT013/#250` remains a verified relevant prior authority.

## Project routing defect

The Issue had reached:

```text
FAMILY=family:AUD
STATUS=status:in-progress
ASSIGNEE=guillermomolina
PRIORITY=UNSET
```

Repository coordination policy requires every formal top-level `status:in-progress` item to have a resolved scheduling Priority. `scripts/project_status_sync.py` fails closed on an in-progress formal Issue whose effective Priority is unresolved.

The Project `Work queue` view is canonically filtered to open repository Issues whose Project Status is one of:

```text
In progress
Review
Ready
```

Therefore the missing Priority prevented the Issue synchronization transaction from completing and explains why AUD019 existed as a repository Issue but was absent from the Project Work queue.

## Repair

The live Issue was assigned normal planned scheduling priority:

```text
PRIORITY=priority:p2
PROJECT_PRIORITY=P2
```

This is the repository's ordinary planned-work priority, not an escalation to P1/P0 and not a later/opportunistic P3 classification.

Adding the label emits the repository `issues:labeled` event consumed by `.github/workflows/project-status-sync.yml`, which reruns formal intake and the bounded Project synchronization for `#818`.

Post-repair live scheduling tuple:

```text
ISSUE=818
FAMILY=family:AUD
STATUS=status:in-progress
ASSIGNEE=guillermomolina
PRIORITY=priority:p2
WORK_QUEUE_FILTER_MATCH=YES
```

The Project itself remains derived presentation; Issue labels/status/priority remain the scheduling authority according to project coordination policy.

## Current work state

```text
AUD019_STATUS=IN_PROGRESS_RESEARCH
TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO
RESEARCH_OUTPUT_DESTINATION=docs/project/work/AUD019/
PROJECT_ROUTING_DEFECT=MISSING_PRIORITY
PROJECT_ROUTING_DEFECT_REPAIRED=YES
```

The eventual AUD019 research record must supersede this intake-only evidence with the source-backed capability inventory, candidate comparison, recommendation pending explicit owner approval, and routed Dxxx/PLATxxx/LIBxxx/TOOLxxx follow-up work required by `#818`.
