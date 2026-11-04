# PLAT069 — Owner-directed pause pending I115 shape/storage convergence

- **Evidence date:** 2026-11-04
- **Formal work owner:** [PLAT069 / guillermomolina/protos#1048](https://github.com/guillermomolina/protos/issues/1048)
- **Direct prerequisite:** [I115 / guillermomolina/protos#931](https://github.com/guillermomolina/protos/issues/931) (still open)
- **Product main revision inspected:** [`a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b`](https://github.com/guillermomolina/protos/commit/a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b)
- **Experimental I115 branch revision inspected:** [`e7124cbcbf2adaeffc1cf063adc1ebeea66d2357`](https://github.com/guillermomolina/protos/commit/e7124cbcbf2adaeffc1cf063adc1ebeea66d2357)
- **Adopted platform:** GraalVM / Truffle 25.5.5.0.3 through [DIST017/#968](https://github.com/guillermomolina/protos/issues/968)
- **Evidence class:** read-only source comparison plus explicit project-owner coordination instruction; **not** design ratification, test acceptance, completed exhaustive inventory, or product publication.

## Owner instruction and disposition

On 2026-11-04, the project owner instructed that **PLAT069 be paused until I115 is finished**. Leave PLAT069 **OPEN** with the single `status:paused` label, preserving `family:PLAT`, its top-level scope, acceptance criteria, and intentionally unset priority. Do not close the investigation, approve the earlier proposed incremental-adoption strategy, create implementation slices, or infer any benchmark win.

The pause is an explicit ordering decision: **complete and publish I115 first; reassess the released GraalVM 25.5 capabilities on the resulting actual Protos implementation**. No change is requested to I115's own `status:ready`, the independently owned PLAT070/#1049 `unwindExceptions` research, or the completed DIST017 compatibility migration.

## Source-level reason for the pause

The earlier assertion that Protos "does not use Truffle shapes" was too broad because it confused the product `main` snapshot with the active experimental I115 branch.

| Checked surface | Exact evidence | Finding |
| --- | --- | --- |
| Current product `main` | [`ProtosObjectValue.java` at `a7aa54bc`](https://github.com/guillermomolina/protos/blob/a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b/src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java) | Ordinary object class implements `TruffleObject`; it does not extend Truffle `DynamicObject`. |
| Current product `main` | [`ProtosValueLookup.java` at `a7aa54bc`](https://github.com/guillermomolina/protos/blob/a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b/src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java) | Its shared-inherited-lookup comment explicitly distinguishes exact-receiver PIC own-slot access backed by string-keyed `LinkedHashMap` storage from a shared ordinal layout. |
| Active `I115` branch | [`ProtosMapBackedLexicalBindingAuthority.java` at `e7124cbc`](https://github.com/guillermomolina/protos/blob/e7124cbcbf2adaeffc1cf063adc1ebeea66d2357/src/main/java/com/guillermomolina/protos/runtime/lexical/ProtosMapBackedLexicalBindingAuthority.java) | Contains experimental `Shape.newBuilder().build()`, `Store extends DynamicObject`, `DynamicObject.GetNode`/`PutNode`/`RemoveKeyNode`, and `store.getShape().makePropertyGetter(key)`. The `LinkedHashMap` tracks names and optional PIC cells; values reside in the `DynamicObject` store. |
| Active `I115` branch | [`ProtosValueLookup.java` at `e7124cbc`](https://github.com/guillermomolina/protos/blob/e7124cbcbf2adaeffc1cf063adc1ebeea66d2357/src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java) | Exposes experimental guarded selections with `DynamicObject`, `PropertyGetter`, and `PutNode`; older map-based explanatory comments also remain and need final-state reconciliation. |

These findings show **different physical representations at two distinct revisions**. They do **not** prove that I115's experimental branch is merged, tested in its latest state, semantically ratified, or fast. Conversely, the product main snapshot cannot be used as a lasting reason to declare 25.5's object/shape APIs inapplicable.

## 25.5 facilities affected by the final-state check

The exact released upstream baseline for the investigation is [`oracle/graal@e3cb74fceaf6ee4f50366d5ab2bbf602e464475d`](https://github.com/oracle/graal/tree/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d), not a moving `master` revision.

- **`DynamicObject.SetAllPropertyFlagsNode` (GR-79950):** revisit only after I115 settles its authoritative storage, mutation, and property-flag requirements. Its existence does not justify wholesale flag rewrites.
- **`PropertyGetter.getShape()` (GR-80124):** revisit against the actual post-I115 `PropertyGetter` fast path, assumption/shape invalidation, and polymorphic receiver behavior. The experimental branch makes this potentially relevant, but no use is approved.
- **Other 25.5 capabilities:** preserve PLAT069's full original inventory and all independent owners. This pause is not a blanket `NO_ACTION` disposition or a completed comparison.

## Resume gate

1. I115/#931 reaches an owner-confirmed completed state with an exact published product revision (or the owner explicitly directs an earlier resumption).
2. Re-read the **published** ordinary-object storage, getter/setter, selector PICs, shape transitions, invalidation, and relevant tests at that revision. Contrast them against the experimental `I115` evidence above; never substitute branch evidence for product-main evidence.
3. Then evaluate `SetAllPropertyFlagsNode` and `PropertyGetter.getShape()` at the exact released 25.5 API identity and compare with relevant Truffle peers, including GraalPy.
4. Resume PLAT069's GITHUB010 comparative inventory, negative findings, counterexamples, and independent adoption dispositions. Any substantive architecture selection still requires explicit project-owner approval.
5. Reconcile the live Issue status and any canonical native dependency only through the governed GitHub coordination workflow. No automatic state transition on I115 closure.

## Validation and boundaries

- **Source inspection:** GitHub read-only at the exact product and experimental branch SHAs above.
- **Product code/specification/version:** unchanged by this coordination checkpoint.
- **Tests, builds, benchmarks, `git diff --check`:** **NOT RUN** for this evidence-only coordination change; no runtime/performance claim is made.
- **Live owner:** the [PLAT069 Issue](https://github.com/guillermomolina/protos/issues/1048), not this historical evidence record.
- **GitHub dependency:** `PLAT069 blocked by I115` is an exact, direct prerequisite. Record the intent in the Issues; if the available connector cannot mutate native GitHub Issue dependencies, explicitly report the missing native edge rather than claiming it exists.
