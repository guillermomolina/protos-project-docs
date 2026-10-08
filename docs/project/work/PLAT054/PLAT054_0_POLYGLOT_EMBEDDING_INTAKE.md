# PLAT054 — Standard Polyglot embedding intake and owner-approved direction

Date: 2026-10-08
Issue: https://github.com/guillermomolina/protos/issues/838
Parent performance owner: https://github.com/guillermomolina/protos/issues/831

## Proven blocker

The requested common Java preparation `context.eval(source); context.getBindings(language).getMember("truffleRun").execute()` is not currently implemented by raw Protos Context embedding. Current `ProtosLanguage.parse` returns a CallTarget whose frame ABI requires `ProtosActivation`; existing standalone Process bootstrap supplies that activation. The earlier implementation agent stopped without changing any product files. The user reported local tests passing and clean git diff; this is not evidence that raw Context.eval or bindings works.

## Explicit project-owner direction

1. One implicit Process per embedder-owned Polyglot Context, lazy upon first eval, disposed on Context close.
2. Core by default from distribution/language home, plus a public host override; not mandatory external Core configuration.
3. Stdio and app args from Truffle Env; host-authorized environment/filesystem; network denied by default.
4. One persistent Process per Context, while each eval obeys existing new module-instance semantics rather than silently becoming REPL; bindings address last successfully completed entry module.
5. Host-side bindings are read-only, while internal Protos slot writes keep ordinary semantics.
6. Do not impose always-on per-execute serialization; respect existing Context/RootActor concurrency constraints.
7. Executable Values retain the originally captured Closure across later evals.
8. PAY AS YOU GROW: no per-execute Process bootstrap, RootTask, ProtosTask, rich activation, scheduler.

## Ratification gate

These are owner-approved **candidate invariants**, not a claim that GITHUB010 ratification, normative writing, or required conflict audit is complete. PLAT054 owns that work and should verify against PLAT001, PLAT046, PERF033 and the current normative MODULES / PROCESS_IO / CALLABLES / ERRORS. If any existing approved invariant conflicts, explicitly route that conflict back to owner rather than silently overwrite it. Do not introduce a second export registry or broaden Protos lexical visibility; only module top-level slots are eligible. Implement a standard interop scope when approved.

## Outcome and routing

```text
IMPLEMENTATION_PERFORMED=NO
ISSUE_CREATED=PLAT054/#838
PERF032_STATUS=OPEN
NEXT_SLICE=PLAT054-0
NEXT_SLICE_TYPE=INVESTIGATION_DESIGN_RATIFICATION
NEXT_REPOSITORY=NONE
BENCHMARK_CHANGE_REQUIRED_NOW=NO
```

No repository tests or measurements were executed by this documentation coordination step. User-reported tests PASS and git diff check CLEAN are retained as user statements and not independently verified.
