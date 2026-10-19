# PLAT066 second implementation wave — owner authorization and bounded scope

**Date:** 2026-10-19. **Source:** active maintainer conversation authorizing a continuation: “hagamos un plan con lo que hay ahora ... acepto esto, como nueva evolución del PLAT, pero no lo cerremos ... hay que reabrir el I para implementar esta nueva ola de acercamiento a graalpy”. Follow-up explicit correction: **“no, asume que NORMAL / DEBUG no es un problema, tu has de cuenta que solo hay un modo”**. The latter is a design/implementation premise: **ONE execution mode**, no mode guards, no coordination dependency on removal work.

**Actual live state at approval:** I115/#931 was already OPEN and `status:ready`, PLAT066/#924 OPEN, PLAT066-C/#930 OPEN; D207/#932 OPEN; D013/#171 CLOSED/RATIFIED. Hence “reopen I” means resume/expand same I115, **not** fabricate closed→open history or create a new I-family work item. Original C parent issue is not closed automatically. Permission given in this conversation to commit/push evidence to `guillermomolina/protos-project-docs` despite any default AGENTS restriction; product repository remains human-executor only.

**Approved:** group new C=PIC local-call invalidation + definition/Context target proof + ReturnHome certification; D=B5+ caller conditional simplification + numeric carrier/guest return proof. Physical changes may span Java runtime/compiler and existing `InvokeClosureOneTail` (not bodies of other DSL operations; no new operations). The user explicitly wants another wave after this if needed; PLAT066 remains OPEN.

**Not approved:** replacing dynamic ordinary `call` semantics (D013), altering observable ReturnHome/Task/Context/ABI/guest numeric values, closing PLAT066, claiming an unmeasured GraalPy graph match. D207's reconsideration of the D013 language contract remains separate.

**Source/comparator:** `guillermomolina/protos@27cb31d20e517d8377e58963ea595e5ec16bfd15`, `oracle/graalpython@graal-25.4.4`, `truffleruby/truffleruby@b60b0d04dd11454012d0e8a6588514410ffacdcb`. Verified historical TOP_LEVEL After TruffleTier `1080 -> 707`, residual approximately `125` **inclusive** source ancestry, not guaranteed savings. No build, tests, new benchmarks or product edits were executed for this evidence publication.

**Durable links:** [plan](../../work/PLAT066/PLAT066_WAVE2_FIVE_POINT_CONVERGENCE_2026_10_19.md), [agent](../../work/I115/I115_C_AGENT_HANDOFF_2026_10_19.md), [human executor](../../work/I115/I115_C_HUMAN_EXECUTOR_2026_10_19.md), [first editing batch](../../work/I115/I115_C_FIRST_EDIT_BATCH_2026_10_19.md).
