# I130-C2 — Native Image TLS execution gate and explicit trust-source provenance: Agent / Human Executor

**Kind:** IMPLEMENTATION (not research).
**Edit repository:** `guillermomolina/protos` **only**. The agent receives the product repository already at local HEAD, which contains published I130-C1 [`fb7c984574a11612f0527127c454d6783cbd6a8f`](https://github.com/guillermomolina/protos/commit/fb7c984574a11612f0527127c454d6783cbd6a8f) / `0.3.418-SNAPSHOT`. Preserve any unrelated local changes. Parent [I130 #977](https://github.com/guillermomolina/protos/issues/977), approved [D220–S12 / #975](https://github.com/guillermomolina/protos/issues/975). **No new Issue**. C1 completion [evidence](../../evidence/I130/I130_C1_0_3_418_COMPLETION_2026_10_29.md).

## Agent: paste this entire edit-only prompt to Claude

You are the **I130-C2 IMPLEMENTATION AGENT** inside the local `guillermomolina/protos` repository already at HEAD. **This is FILE EDITING ONLY, not investigation.** The file-by-file changes and all necessary external conclusions are given below.

Read and obey the local applicable `AGENTS.md` and `AGENTS.work/IMPLEMENTATION.md` instructions solely as required to edit; the normative contract is `spec/io/TLS.md` §§3.2, 9, 10, 11. Read only the named target files and directly necessary adjacent source definitions. Do not browse the web, inspect external repositories, run commands (including git status, diff, rg, tests or compilers), install anything, fetch/pull/checkout, create branches, commit or push. Do not edit any repo other than this product repository. No speculative Java/JSSE/HTTP architectural changes. Hand back edited files and human-only checks.

**Precise goal:** make the *actual native executable* run the three existing TLS guest suites as a required Native regression gate, add one JVM regression proving that the selected explicit Java `cacerts` source is not replaced by ambient default JSSE trustStore properties, and document exactly what the gate proves and does **not** prove. Keep real native build/provider/CA-distrust proof a human acceptance condition. This is a single cohesive edit batch in **three existing files**.

### 1. Edit `build/native/test-native.sh`

This script already defines `root`, `native_bin=${root}/target/native/protos`, `tmp_dir`, and runs the numeric/regex/URI native Test Tool with a `PROTOS_HOME=${root}` environment. Preserve every existing smoke, debugger/LSP, interpreter-only, numeric, regex, inspect and final status condition without weakening them.

- Immediately after the existing `native_numeric_cases` loop, insert a deterministic native TLS loop with three **exact** relative test files, in this order:
  1. `protos/tests/library/tls/client-admission.protos`
  2. `protos/tests/library/tls/client-handshake.protos`
  3. `protos/tests/library/tls/identity.protos`
- Define `native_tls_status=0`, `native_tls_cases=0`. For each path, call the **actual AOT binary**, `PROTOS_HOME=${root} "${native_bin}" test --file "${root}/${tls_file}"`, redirecting stdout/stderr to the unique log `"${tmp_dir}/tls-${native_tls_cases}.log"`. Capture its exit status before any other command. Treat exit nonzero, missing summary, zero tests passed, or any failures as a failing gate. Match the existing strong summary regex exactly: `^[[:space:]]*[1-9][0-9]* passed, 0 failed[[:space:]]*$`. On a failing case emit the whole log to stderr, `NATIVE_TLS_CASE_FAILURE=<relative path>`, and set `native_tls_status=1`; **never silently skip an unavailable suite**. Increment count each time.
- Add clear, truthful status counters beside existing `NATIVE_NUMERIC_CONFORMANCE_*` output: `NATIVE_TLS_CONFORMANCE_STATUS=${native_tls_status}` and `NATIVE_TLS_CONFORMANCE_FILES=${native_tls_cases}`. The parent script must fail with `NATIVE_TLS_CONFORMANCE=FAIL` when any native TLS suite fails or fewer/more than three exact cases ran. Only emit `NATIVE_TLS_CONFORMANCE=PASS` when all three have passed; never print a global PASS before this gate. Keep native final `NATIVE_REGRESSION_SUITE=PASS` conditional upon TLS success.
- Do not invent a `NATIVE_TLS_AUTHENTICATED_PEER=PASS` or `NATIVE_TLS_CACERTS_PROVENANCE=PASS` marker. The scripted guest TLS fixtures exercise real Native JSSE engine reachability, ClientHello and failure handling, but **do not perform a successful full wire handshake with authenticated remote peer**.

### 2. Edit `src/test/java/com/guillermomolina/protos/execution/host/tls/ProtosTlsTrustTest.java`

The existing class uses package declaration `com.guillermomolina.protos.execution`, JUnit 5 and the existing fixture `ROOT`. Preserve its license and all current tests.

- Add exactly one deterministic JVM test named `javaCacertsIgnoresAmbientTrustStoreProperty`.
- First call `ProtosTlsTrust.javaCacerts()` to capture the explicit baseline certificate list. Save `System.getProperty("javax.net.ssl.trustStore")` and, inside a `try/finally`, temporarily set that exact property to a deliberately nonexistent truststore path (e.g. `"/nonexistent/protos-i130-c2-default-truststore"`). Call `ProtosTlsTrust.javaCacerts()` again and assert equality with baseline; restore the property in `finally` (clear if originally unset, otherwise restore exactly).
- Do **not** modify `java.home`, the JVM default `SSLContext`, global `Security` providers, the existing `src/test/resources/tls` fixture files, or `ProtosTlsTrust.java`. Avoid touching `javax.net.ssl.trustStorePassword` and any other ambient property. This bounded regression tests the explicit file selection in a running JVM. It **does not** prove Native build-time trust/distrust freshness.

### 3. Edit `dist/README.md`

In the existing Native Image build/smoke discussion, add a short subsection titled **TLS under Native Image**:

- The human runs `make -C build/native build` and `make -C build/native test` on the real Native binary. `test-native.sh` now checks the three TLS guest files as mandatory gates; a pass exercises fixture-driven engine creation, X.509 DER handling, ClientHello and negative/fail-closed paths in the native executable.
- Explain that `Trust.anchors([...])` is explicit DER material supplied by the caller; `Trust.javaCacerts()` is an **opt-in** request for the exact `java.home/lib/security/cacerts` file, resolved at each admitted handshake by `ProtosTlsTrust.javaCacerts()`. `javax.net.ssl.trustStore`, `jssecacerts`, default `SSLContext` and OS anchors are **not** alternate or fallback trust sources for this API. `Trust.system()` currently fails closed.
- State clearly: GraalVM Native Image may capture **default JSSE certificate roots**, security-provider availability and blocked/distrusted certificate information at **image-build time**. This does not mean Protos' explicit file path necessarily uses the captured default: test/describe those as *separate trust mechanisms*. The native scripted suites alone do not prove the runtime file provenance, post-build CA updates, blacklist freshness or a positive TLS 1.2/1.3 authenticated real-peer handshake. Such acceptance remains **UNVERIFIED** until separately demonstrated with a controlled native peer and observable trust-source change; never describe the three-suite pass as complete HTTPS/native interoperability.
- Keep this documentation descriptive and bounded; no new public APIs, silent CA fallback, automatic network/DNS authority or security promises.

### Source-grounded external facts (no browsing/research required)

- **Protos normative** `TLS.md` §3.2 selects explicit Java `cacerts`; §11 warns about Native Image build-time CA/provider/distrust provenance. §10 bounds encrypted records at 18,432 bytes; C1 already classifies conclusive oversized inbound headers to `TlsError.kind="limit"`.
- **Actual Java implementation:** `ProtosTlsTrust.javaCacerts()` looks up `System.getProperty("java.home")`, builds `Path.of(javaHome,"lib","security","cacerts")`, opens that file and parses certificate entries via `KeyStore`. It never consults `javax.net.ssl.trustStore`, `jssecacerts` or `SSLContext.getDefault()`. `protos/lib/tls/internal/TrustSource.protos` alone selects explicit Trust descriptors and maps unavailable/empty material to `TlsError("trustUnavailable")`.
- **Test scope:** `client-admission.protos` checks explicit roots, admission and conditional `javaCacerts` fail-closed. `client-handshake.protos` covers scripted peer/error bounds, fragmented overlong record, cancellation and custody. `identity.protos` covers DNS/IP references. **No successful on-wire TLS peer** appears in these guest fixture suites; JVM integration tests provide separate positive TLS peers.
- **Build mechanics:** `build/native/Makefile` has `build` (Maven native profile) and `test` (invokes `build/native/test-native.sh` against `target/native/protos`). `dist/build_native.py` packages that executable; do not introduce a second binary or rebuild pipeline.
- **Primary upstream behavior:** Oracle GraalVM Native Image certificate-management documentation for GraalVM 25 describes default trust roots and a distrusted-certificate list that may be captured during image building; JCA security providers are subject to Native Image reachability/configuration. A failing native provider is evidence to diagnose **after human validation**, not permission to add broad `-H:AdditionalSecurityProviders` or unconditional initialization flags now.
- **Truffle comparisons:** GraalPy's `ssl` / `asyncio` separation keeps guest stream, verification policy and cancellation above backend TLS mechanisms; GraalJS/Node's guest API is likewise distinct from crypto runtime; TruffleRuby's OpenSSL/IO keeps resource policy at its guest boundary; Espresso exposes a guest Java runtime but does not bypass Native reachability requirements; Sulong's native TLS interop would require explicit handle custody. None implies that Protos must create another public TLS engine or move its Protos-first Trust/Identity/Task policy to Java.

**Outside this patch:** new TLS engine architecture, changes to `pom.xml` native-image configuration, `spec/` normative changes, HTTP modules, native TLS server, identity/IDNA2008 A-label implementation and fake test PASS. Treat the successful Native certificate-chain/trust/distrust provenance proof as a distinct **human-gated remaining requirement**, not silently completed by this patch. If a code change beyond three files is essential, STOP and report the precise blocker.

**Agent output:** files edited, each exact change, tests CREATED/REUSED but NOT RUN, caveats and human commands; no commands or Git actions.

## Human Executor — exclusive validation/publication

1. Review the three-file diff and confirm no altered pre-existing native tests, test threshold, normative contract or backend ownership.
2. Run `git diff --check`, then focal JVM `mvn -q test -Dtest='ProtosTlsTrustTest,ProtosTlsClientEngineTest,ProtosTlsClientIntegrationTest'` and guest TLS suites in the normal JVM Test Tool. If any failure, return only first concrete error to the agent for a file-only repair.
3. With the installed native-image toolchain and repository-local build prerequisites, run **`make -C build/native build`** and **`make -C build/native test`**. Record real native executable path, relevant GraalVM/JDK version, `NATIVE_TLS_CONFORMANCE_STATUS`, `NATIVE_TLS_CONFORMANCE_FILES`, and full `NATIVE_REGRESSION_SUITE` PASS/FAIL. Do not relabel a skipped build PASS.
4. Independently establish whether a positive Native client/controlled peer can validate a chain using explicit `Trust.anchors`, and then whether `Trust.javaCacerts` uses a changeable runtime file or captured image material and whether blacklist/distrust is build-time fixed. If there is **no controlled positive/native provenance probe**, report those items **UNVERIFIED**, not PASS; retain I130/C2 closure hold and escalate a bounded follow-up with exact missed proof. Never test against arbitrary public TLS servers or infer provenance from JVM-only tests.
5. After relevant gates actually PASS, run `make test`, `make check` and report `git diff --check`. Only the human may finalize `pom.xml` / `CHANGELOG.md` version under actual HEAD, stage exact owned files, commit and push per local workflow. Do not amend `spec/PROTOS_SPEC_CHANGELOG.md` without a normative spec change.
6. Report exact product SHA and native-result limitations. Coordinator then creates new immutable evidence and comments on I130. **I130 and D220 stay OPEN** until full accepted S12 conformance, including Native provenance and strict IDNA2008, is proven or explicitly re-decided by Guillermo.

**STATUS:** C1 PUBLISHED / C2 EDIT-ONLY HANDOFF READY / NO C2 BUILD OR NATIVE PASS CLAIMED.
