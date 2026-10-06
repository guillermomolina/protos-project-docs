# TEST009-V2 — Protos stops at the BGV capture boundary

## Published product

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-V2
PROTOS_REVISION=21437984ceb311ff89d19563ea97da342e37f8da
PROTOS_VERSION=0.3.237-SNAPSHOT
COMMIT_SUBJECT=TEST009-V2: stop single-root diagnostics at BGV capture

MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
PUBLICATION=PUSHED
```

TEST009-V2 completes the Protos-side repository-boundary correction identified
after TEST009-V. It does not change Protos semantics, the strict compilerability
gate, or compiler policy.

## Final Protos capture contract

The canonical single-root workflow remains:

```text
truffle-root-catalog
  -> choose one stable source/span-attributed selector
  -> diagnose-truffle-root with engine.CompileOnly
  -> retain diagnostic log / one requested expansion view / fresh BGV
  -> stop
```

A successful capture now exposes a machine-readable handoff:

```text
TRUFFLE_ROOT_DIAGNOSTIC=<TARGET_COMPILATION_*>
TRUFFLE_ROOT_CAPTURE_READY=YES
TRUFFLE_ROOT_CAPTURE_BOUNDARY=BGV
TRUFFLE_ROOT_BGV=<path>
```

with one `TRUFFLE_ROOT_BGV` line per retained dump.

Non-evidence results report:

```text
TRUFFLE_ROOT_CAPTURE_READY=NO
```

and do not expose a BGV handoff.

## Fail-closed artifact evidence

A `TARGET_COMPILATION_*` result is accepted as a Protos-side capture only when
the retained BGV evidence is present and usable as a file artifact.

V2 adds explicit rejection for:

```text
MISSING_BGV
EMPTY_BGV
```

and records `capture_ready` in `report.json`.

This remains file-level capture validation only. Protos does not parse or
interpret the BGV structure.

A real selected-root `CodeTooLarge` outcome remains valid compiler diagnostic
evidence when the required trace/expansion evidence and fresh non-empty BGV
exist. V2 does not repair `CodeTooLarge`.

## Repository boundary

The final boundary is now explicit in code, tests, Makefile help and the
non-normative Truffle/Graal investigation reference:

```text
PROTOS_OWNS=CAPTURE
PROTOS_CAPTURE_BOUNDARY=BGV

PROTOS_REQUIRES_IGV=NO
PROTOS_REQUIRES_IGVUTIL=NO
PROTOS_REQUIRES_DOCKER_ANALYZER=NO
PROTOS_PARSES_BGV=NO
PROTOS_DEPENDS_ON_PROTOS_BENCHMARKS=NO
```

The retained BGV is the handoff artifact for downstream Graal graph analysis.
That analysis intentionally belongs outside the Protos repository/tool contract.

The broad no-`CompileOnly` candidate-discovery probe used during V acceptance
is not part of the canonical workflow. Candidate roots are selected from the
cheap catalog using source/span attribution.

## Published files

The V2 publication changed:

```text
CHANGELOG.md
Makefile
docs/design/TRUFFLE_GRAAL_OPTIMIZATION_INVESTIGATION_REFERENCE.md
pom.xml
tools/test_truffle_root_diagnostic.py
tools/truffle_root_diagnostic.py
```

The product version advanced from `0.3.236-SNAPSHOT` to
`0.3.237-SNAPSHOT`.

## Validation reported by maintainer

```text
ALL_LOCAL_TESTS=PASS
GIT_DIFF_CHECK=PASS
```

The published tests pin, among other things:

- `CompileOnly` single-root behavior remains the authority;
- fresh BGV capture is required;
- missing and zero-byte BGV files fail closed;
- the handoff stops at BGV;
- no IGV, Docker, extra analyzer process or BGV parser is introduced into the
  Protos diagnostic path.

## Next repository boundary

The Protos side is now complete for this tooling boundary.

The next implementation is intentionally in:

```text
REPOSITORY=guillermomolina/protos-benchmarks
PURPOSE=VERSION_ALIGNED_PREBUILT_HEADLESS_BGV_ANALYZER
```

The existing benchmark/tooling repository already owns a headless
`org.graalvm.igvutil.IgvUtility` analyzer pattern. The follow-up should align
that analyzer with GraalVM 25.4.4.1.1 and make normal BGV analysis consume a
prebuilt/pinned image rather than building Graal/IGV during ordinary use.

After that tooling is available, TEST009 can proceed to the actual single-root
`CodeTooLarge` graph diagnosis without adding another Protos-side capture
mechanism.

## Coordination

```text
TEST009_STATE=OPEN_IN_PROGRESS
NEW_FORMAL_ISSUE_REQUIRED=NO
NEW_SUB_ISSUE_REQUIRED=NO

TEST009_V2=COMPLETE
PROTOS_CAPTURE_BOUNDARY_READY=YES

NEXT_SLICE=TEST009-V3
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos-benchmarks
NEXT_SCOPE=PREBUILT_VERSION_ALIGNED_HEADLESS_BGV_ANALYZER

TEST009_W_RESERVED_FOR=REAL_CURRENT_CODE_TOO_LARGE_SINGLE_ROOT_DIAGNOSIS
```
