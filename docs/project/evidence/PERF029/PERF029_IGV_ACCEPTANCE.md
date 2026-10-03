# PERF029 — external IGV acceptance

## Scope

This record preserves the external compiler/IGV acceptance for
`guillermomolina/protos#783` after publication of the PERF029 product repair.

It is non-normative diagnostic evidence. PERF024 / #756 remains the owner of the
subsequent cross-runtime Protos/GraalJS/GraalPy physical inventory.

## Exact authorities

```text
PROTOS_REVISION=63df450263feb5bfbfb3160aecf33128f456e08c
PROTOS_VERSION=0.3.172-SNAPSHOT
PRODUCT_RECORD_REVISION=209c34249766389e29b06bc36d3bc70d04c01aaa
BENCHMARK_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
DIAGNOSTIC=igv
RUN_MODE=prepared
GRAALVM_VERSION=25.4.4.1.1
```

The benchmark worktree was clean. The Protos acceptance used a clean detached
worktree at the exact published product revision because the maintainer's active
`/workspaces/protos` checkout contained unrelated in-progress filesystem work.

## Diagnostic regime

```text
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
language=protos
diagnostic=igv
workload=primitive-closure-call
```

The existing generic diagnostic surface was used unchanged:

```text
truffle/jvm_diagnostic.py igv protos primitive-closure-call
```

No PERF029-specific benchmark or diagnostic modification was introduced.

## Result

```text
diagnostic=igv
cache=miss
timing_evidence=NON_PRIMARY
language=protos
run_mode=prepared
protos_revision=63df450263feb5bfbfb3160aecf33128f456e08c
protos_version=0.3.172-SNAPSHOT
correctness=PASS result=1
bgv_files=2
```

Diagnostic artifact directory:

```text
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/37a94853d7a16dc3abe217546e68a3befea4237cf728f49742f07d6c742123aa
```

Produced BGV identities:

```text
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/37a94853d7a16dc3abe217546e68a3befea4237cf728f49742f07d6c742123aa/graal_dumps/TruffleHotSpotCompilation-2193[ProtosSemanticBytecodeRootNodeGen@12dae768].bgv
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/37a94853d7a16dc3abe217546e68a3befea4237cf728f49742f07d6c742123aa/graal_dumps/TruffleHotSpotCompilation-2619[ProtosSemanticBytecodeRootNodeGen@12dae768].bgv
```

The run log contains no occurrence of either previously known bailout token:

```text
7799|Pi
5866|AnyNarrow
```

It also contains no occurrence of the previously identifying source/exception
discriminators:

```text
PermanentBailoutException
ProtosFrameLexicalBindingAuthority.putBinding
BindClosureFrameParameter.perform
LocalRangeAccessor.isCleared
```

The graphs themselves were not inspected in this acceptance slice.

## Mechanical classification

```text
PERF029_IGV_ACCEPTANCE=PASS
KNOWN_BAILOUT_A=REMOVED
KNOWN_BAILOUT_B=REMOVED
PROTOS_BGV_AVAILABLE=YES
CORRECTNESS=PASS
RESULT=1
BGV_FILES=2
```

PERF029 has therefore satisfied its external compiler/IGV gate. The next
diagnostic work belongs to PERF024 / #756:

```text
NEXT_WORK=raw cross-runtime physical inventory
RUNTIMES=Protos,GraalJS,GraalPy
WORKLOAD=primitive-closure-call
INPUTS=IGV+JFR
INTERPRETATION=NONE during initial inventory
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
maintainer-executed diagnostic output and the published repository authorities.
