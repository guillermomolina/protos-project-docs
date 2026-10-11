# I090-A — Published SmallInteger and BigInteger family implementation

**Record date:** 2026-10-11
**Formal implementation owner:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)
**Ratified numeric decision:** [D197-C](../D197/D197_C_HIERARCHY_RATIFICATION_AND_IMPLEMENTATION_COST_2026_10_11.md) and [canonical decision](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)
**Platform constraint:** PLAT056-C, primitive-first and pay-as-you-grow
**PROTOS_REVISION:** `888c2b4f2390b96de5990533fdb1b306587e0002`
**Published product commit:** https://github.com/guillermomolina/protos/commit/888c2b4f2390b96de5990533fdb1b306587e0002
**Evidence classification:** revision-pinned source publication, maintainer-reported local validation; the recording agent did not execute tests.

## Scope of I090-A

The published product commit is titled **"I090: add the public SmallInteger and BigInteger families under Integer with exact signed-64 normalization"**. It adds `protos/lib/core/SmallInteger.protos` and `protos/lib/core/BigInteger.protos` and publishes the D197-C concrete family hierarchy:

```text
Number
└── Integer (common exact-integral prototype)
    ├── SmallInteger (canonical signed-64)
    └── BigInteger (outside signed-64)
```

The Core bootstrap and Prelude register and validate both prototypes. Integer common arithmetic remains inherited from the canonical `Integer` method home; concrete `recognizes` predicates distinguish signed-64 and out-of-range semantic Integer values, while `Integer.recognizes` accepts both. Signed-64 materialization and arbitrary-precision normalization choose their respective guest-visible parents. This is a semantic family distinction, not a requirement to allocate a wrapper for every small numeric operation.

The published change coordinates `ProtosNumericValueSupport`, guarded integer lookup, canonical Bytecode DSL sends, existing `long` primitive carriers, and large integer rematerialization including the IPv6 128-bit path. The source commit includes a new `protos/tests/conformance/integer/family-boundaries.protos` suite plus amendments to related conformance, JUnit, architecture and cross-Prelude tests.

The product revision also updates `spec/semantics/VALUES_AND_COLLECTIONS.md`, informative `spec/runtime/ABSTRACT_RUNTIME.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, `pom.xml`, and `CHANGELOG.md`. This record does not claim that the remaining D197 numerical tower is implemented.

## Validation provenance

On 2026-10-11 the **human executor reported**:

- `git diff --check` clean;
- all local tests PASS;
- the I090-A product commit was pushed.

The exact product SHA and published changed-path inventory were independently inspected using GitHub read access. The coordinating agent did **not** run compilation, conformance, `make test`, Java tests or benchmarks. The maintainer did not provide test logs, a per-suite run timestamp/count, compiled-graph evidence or benchmark comparisons in this handoff. Therefore this is not independent proof of the test run or of a performance improvement.

## Remaining I090 work and ownership

**I090 is not complete.** D197-C also requires exact Integer/Integer division with normalized `Fraction` results, Fraction-domain arithmetic and cross-family relations/conversions, and `Complex` semantics including signed-zero branch-cut preservation. These require further separately validated implementation slices. Existing mixed Float/Integer operand-first D196-B1 behavior and D013 override/deoptimization are constraints, not permission for opportunistic refactoring.

`I091/#869` separately owns PLAT056 numeric primitive-path and Java host-boundary containment work. `PERF040/#862` still requires its own graph/performance evidence. Neither is closed by this I090-A publication. GitHub Issues remain the authoritative live status and priority source.

## Provenance and references

- [Published implementation at exact revision](https://github.com/guillermomolina/protos/commit/888c2b4f2390b96de5990533fdb1b306587e0002)
- [I090 implementation issue](https://github.com/guillermomolina/protos/issues/868)
- [D197 ratified architecture](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)
