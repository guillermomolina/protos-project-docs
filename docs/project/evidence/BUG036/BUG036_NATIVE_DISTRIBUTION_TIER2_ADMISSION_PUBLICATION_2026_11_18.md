# BUG036 — Native distribution extracted-ZIP guest Tier-2 JIT admission (2026-11-18)

**Non-normative project evidence, revision-bound.** BUG036 fixes the obsolete interpreter-only release validator. Completing this defect is not an approval to publish any previously failed or new public release candidate.

## Product publication

- Owning Issue: [BUG036 / protos#1130](https://github.com/guillermomolina/protos/issues/1130).
- Exact product commit: [`guillermomolina/protos@cac6502df4cb8b88b73fbe7ebf83adf946b8d2b8`](https://github.com/guillermomolina/protos/commit/cac6502df4cb8b88b73fbe7ebf83adf946b8d2b8), subject `BUG036: require native guest Tier-2 JIT in distribution validation`.
- The published change owns exactly three `dist/` paths: `dist/validate_native.py` (positive Tier-2 gate), newly added `dist/tests/runtime/test_validate_native_guest_jit.py` and removed `dist/tests/runtime/test_validate_native_plat045.py`. No product Java classes, guest source semantics or native runtime optimizer implementation were introduced.
- Baseline context: earlier [I115 / #931](https://github.com/guillermomolina/protos/issues/931) introduced the actual guest Native Tier-2 support and positive `scripts/native-test.sh` gate. BUG036 converges the *distribution* validator on that existing behavior.

## Original defect

During [DIST024 / protos#1129](https://github.com/guillermomolina/protos/issues/1129), the maintainer-selected development baseline [`bfcaf04301a821b96d45b39f93ad17c80cf05ce2`](https://github.com/guillermomolina/protos/commit/bfcaf04301a821b96d45b39f93ad17c80cf05ce2) (`0.4.156-SNAPSHOT`) was frozen into the release-only candidate [`a6e778d7230e4d6626c1dfa8c7fc5831a22910d5`](https://github.com/guillermomolina/protos/commit/a6e778d7230e4d6626c1dfa8c7fc5831a22910d5) (`0.4.156`). Native image compilation, ZIP creation and initial extracted functional checks passed, but the old validator demanded an interpreter-only fallback warning and no successful optimization:

```text
NATIVE_DIST_INTERPRETER_ONLY_STATUS=0
NATIVE_DIST_FALLBACK_RUNTIME_MARKERS=0
NATIVE_DIST_GUEST_JIT_POLICY=UNSUPPORTED_UPSTREAM_ORACLE_GRAAL_14579
NATIVE_DIST_OPT_DONE=0
release preparation failed: native-complete-admission failed with status 1
```

This gate contradicted the already established current Protos requirement that the extracted Native image genuinely compile guest code at Tier 2. The failed candidate was **not** published and must not be silently reused.

## Published correction

The new `validate_forced_guest_jit` runs an actual guest probe with the **relocated, extracted** `libexec/protos-native` executable using the existing optimizer options: `TraceCompilation=true`, `CompileImmediately=true`, `BackgroundCompilation=false`, `CompilationFailureAction=Print`. The probe checks:

- exit status zero and exact expected guest stdout `1`;
- successful Tier-2 `opt done` observations for **both** `ProtosBytecodeRootNodeGen` and `ProtosBuiltinCallRootNode`, not merely one generic compilation counter;
- no fallback runtime, failed optimization, compilation errors or `FrameWithoutBoxing` materialization failure;
- no unexpected stderr or silently suppressed fallback warning.

The replacement Python unit tests exercise the positive case, missing roots, Tier-1-only compilation, nonzero exit, wrong/missing/extra output, fallback warning, compiler/optimization/frame errors and unexpected diagnostics. The other released Native ZIP gates (relocation, ELF/ABI, filesystem, REPL, LSP, DAP, Test Tool, metadata and checksums) remain independently required.

## Observed validation and provenance limits

- The maintainer's earlier fixed-validator work included a reported **39/39 Python validator tests PASS** before the published BUG036 commit; the exact test-run checkout SHA was not captured here.
- On **2026-11-17**, a reported real extracted, relocated Native ZIP from `target/distributions/protos-0.4.156-SNAPSHOT-native-linux-x86_64.zip`, under a temporary relocation installation, reached **`NATIVE_DIST_GUEST_JIT_CHECK: PASS`**, with native probe exit 0, expected guest result, and zero failed optimization/compilation/`FrameWithoutBoxing` markers. This verifies positive admission in a real ZIP validation attempt; it **does not establish that the ZIP was built from the final published `cac6502df` commit** or substitute for the new release's final candidate proof.
- The same extracted validation progressed to the full installed Test Tool `test --jobs 16` gate and then encountered a separate deep-recursion/Case completion problem. That independent defect was subsequently fixed in [BUG037 / #1131](https://github.com/guillermomolina/protos/issues/1131) at [`afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb`](https://github.com/guillermomolina/protos/commit/afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb). It did not invalidate the separately completed Tier-2 admission step.
- On **2026-11-18**, the maintainer confirmed the published Protos source and clean diff, and later reported **all local tests pass** after BUG038; this is a maintainer statement, not an independently captured exact-post-push extracted release archive transcript.

## Disposition and release boundary

BUG036's *code defect* is addressed by the exact published positive Tier-2 gate and regression tests. **DIST024 remains a separate release-execution item**: the failed `0.4.156` candidate is not authorized or complete, and selecting a fresh development baseline/public version and preparing a new exact JVM + Native candidate requires an explicit maintainer decision, independent release preparation, complete extracted validation and separate publication approval.

Do not infer `DIST005_RELEASE_PREPARATION: PASS`, `DIST005_RELEASE_PUBLICATION: PASS`, a GitHub Release/tag, or a Native asset published from this BUG036 snapshot. These remain release-specific gates owned by DIST024.

**Evidence sources:** exact published Protos commit and existing GitHub issue history, plus maintainer-reported validator, extracted-Native and later local-suite results. No build/test or release publication was performed while writing this record.
