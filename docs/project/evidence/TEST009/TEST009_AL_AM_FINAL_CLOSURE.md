# TEST009-AL/AM — final compilerability closure

Status: **PUBLISHED — TEST009 COMPLETE**

Formal work: `guillermomolina/protos#795`

Publication date: **2026-10-07**

## Final product checkpoints

The last causal cleanup slice is:

```text
TEST009_AL_REVISION=f3c594a3815409b080b7b3389b0b838eeec6cbba
TEST009_AL_COMMIT=TEST009-AL: drop redundant List.copyOf in prepared structured-Boolean call
```

The finite infrastructure-closure slice is:

```text
TEST009_AM_REVISION=65a23bc25bd30dfd65843468dba3812a78121c06
TEST009_AM_COMMIT=TEST009-AM: keep strict Truffle compilation out of routine check
```

The maintainer reports:

```text
GIT_DIFF_CHECK=PASS
LOCAL_VALIDATION=PASS
PUBLICATION=PUSHED
```

No observable Protos language or specification semantics are changed by AM.

## TEST009-AL result

AL removed one causally demonstrated, redundant host-side copy from the prepared
structured-Boolean call path:

```text
PreparedBooleanCall
  -> redundant List.copyOf(...)
  -> removed
```

The repair was intentionally bounded to the selected owner and did not authorize
a new sweep of every residual Truffle performance warning.

AL is retained as the final product cleanup publication in the TEST009
compilerability excavation line.

## Closure-policy correction

TEST009 originally distinguished two classes of checks:

```text
bounded / sufficiently cheap deterministic guard
  -> ordinary make check

expensive dynamic compilerability diagnostic
  -> explicit / periodic target
```

The full strict Truffle compilerability gate was later aggregated into ordinary
`make check`, despite the earlier TEST009-F publication having deferred that
aggregation pending clean runtime/cost evidence.

Operational experience showed that the full-corpus strict gate is not a suitable
routine-development prerequisite. It is deliberately aggressive: it compiles the
Test Tool corpus with immediate Truffle compilation and treats performance
warnings as compilation errors.

AM restores the original finite aggregation policy.

Before AM:

```text
make check
  -> toolchain
  -> check-local-range-index-pe
  -> check-local-range-operands-pe
  -> check-local-accessor-pe
  -> check-bytecode-api-pe
  -> check-generated-bytecode-bci-pe
  -> check-truffle-compilation
```

After AM:

```text
make check
  -> toolchain
  -> check-local-range-index-pe
  -> check-local-range-operands-pe
  -> check-local-accessor-pe
  -> check-bytecode-api-pe
  -> check-generated-bytecode-bci-pe
```

The strict gate remains explicitly available:

```text
make check-truffle-compilation
```

and its recipe remains unchanged and fail-closed. The diagnostic surfaces also
remain available:

```text
make diagnose-truffle-compilation
make truffle-root-catalog
make diagnose-truffle-root
```

The maintained self-test now protects the new topology by asserting that
`check-truffle-compilation` is not a prerequisite of `check`.

## Final policy result

TEST009 does **not** establish this as a global Protos invariant:

```text
ALL_PERFORMANCE_WARNINGS=0
ALL_DEOPTS=0
ALL_BAILOUTS=0
```

The strict all-warning mode is retained as an audit/diagnostic instrument, not
as an unbounded generator of product cleanup work.

The ordinary aggregate retains mechanically bounded regression checks, while
future performance optimization is routed from measured workload behavior:
profile the workload, identify hot compilation units, determine whether they
remain interpreted, inspect deoptimizations, then performance warnings, and
finally the Graal graph for the selected compilation unit when the preceding
evidence justifies it.

## Closure

```text
STATIC_FAMILY_GUARDS=PRESERVED
GENERATED_BCI_DYNAMIC_GUARD=PRESERVED
STRICT_FULL_CORPUS_AUDIT=PRESERVED_EXPLICITLY
STRICT_FULL_CORPUS_AUDIT_IN_MAKE_CHECK=NO
NEW_COMPILERABILITY_WARNING_SWEEP_AUTHORIZED=NO

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PRODUCT_RUNTIME_CHANGE_BY_AM=NO

TEST009_COMPLETE=YES
NEXT_TEST009_SLICE=NONE
```

The next performance investigation is intentionally separate from TEST009 and
is centered on the admitted `primitive-return-literal` workload rather than on
global warning-count reduction.

## AI-assistance disclosure

This durable closure record was materially prepared with AI assistance from
ChatGPT using the published TEST009-AL and TEST009-AM commits, the TEST009 issue
contract, current Makefile topology, and maintainer-reported local validation.
No independent human review is claimed.
