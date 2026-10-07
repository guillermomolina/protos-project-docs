# I055 — D159 implementation closure record

Status: **PRODUCT PUBLISHED — durable closure evidence**

GitHub issue: `guillermomolina/protos#656`  
Governing decision: D159, ratified Candidate B  
Product revision: `3fb863b44f1bf5b259abbb8fd44a45635a72f8fd`  
Implementation version: `0.3.60-SNAPSHOT`  
Specification revision: `0.1.432`

## Published outcome

I055 implements D159 Candidate B in the product revision above.

The published product removes the public `Future.detach()` operation, removes
the corresponding detached runtime state and parent-detachment path, and makes
task-backed child ownership strict for the child's complete lifetime. Returning,
storing, wrapping, or passing the child Future does not transfer or remove that
ownership edge. Normal child terminalization, child draining, cancellation,
cleanup, Actor lifetime, adoption, `Future.then`, and `Future.all` remain
within their existing semantic boundaries.

No replacement background-work API, public Task/scope, daemon-task model, or
new Actor lifetime abstraction is introduced.

## Related cancellation defect exposed by I055

I055 validation exposed a pre-existing `Future.then()` source-wait
cancellation defect. A continuation cancelled while waiting on a still-pending
source could re-enter the source-wait path and repeatedly requeue instead of
honoring cancellation.

The product revision fixes that defect by observing cancellation at the private
source-wait resume boundary before registering another wait or propagating the
source outcome. This is required to preserve the already-specified downstream
cancellation behavior while D159 removes detachment.

This repair remains part of I055 rather than receiving a separate BUG identity.
Under the current Issue/slice policy it is a same-slice regression repair:
it was exposed by I055, was necessary for I055's required cancellation
preservation, and has no independent scheduling, blockage, dependency, or
decision boundary.

## Blocker reconciliation

`B008 — Structured ownership when a task-backed Future escapes an activation`
is **CLOSED** in the durable blocker ledger. Its normative dependency is D045 /
specification revision `0.1.382`.

D045's task-scoped ownership rule remains the baseline consumed by I055:
ordinary synchronous activations do not establish implicit concurrency scopes,
and return/store/wrap/pass does not itself transfer ownership. D159 tightens the
lifetime side of that model by removing the former explicit detachment escape:
a task-backed child now remains owned by the surrounding asynchronous task scope
until terminalization.

No blocker is reopened by I055.

## Validation and evidence boundary

The stable implementation and specification evidence is the exact product
revision:

`PROTOS_REVISION=3fb863b44f1bf5b259abbb8fd44a45635a72f8fd`

The product commit contains the implementation version/changelog, normative
reconciliation, runtime changes, Java regressions, Protos conformance, and
fixture reconciliation owned by I055.

Interactive execution evidence established during finalization included the
focused `Future.then()` cancellation regression, the affected Java Future
protocol surface, and the Protos conformance path. The transient runaway
continuation observed during implementation was eliminated before publication;
it is not closure evidence by itself.

This record does not manufacture CI or test-run identities that were not
published as stable artifacts. Product validation remains recoverable from the
published tests and exact product revision together with the owning Issue's
final closure summary.

## Closure coordinates

```text
I055_STATUS=IMPLEMENTED
D159_CANDIDATE=B
FUTURE_DETACH=REMOVED
TASK_CHILD_OWNERSHIP=STRICT_COMPLETE_LIFETIME
B008=CLOSED
D045_TASK_SCOPED_OWNERSHIP=PRESERVED
THEN_SOURCE_WAIT_CANCELLATION=FIXED_WITHIN_I055
SEPARATE_BUG_IDENTITY=NOT_REQUIRED
PROTOS_REVISION=3fb863b44f1bf5b259abbb8fd44a45635a72f8fd
```
