# I160 — Rebuilt Native Image Password acceptance and issue closure (2026-11-10)

## Context and source of evidence

This is a supplementary, **human-reported execution result** for the already published I160/D245-C implementation, not a new product build or a claim of independent execution by the documenting agent.

- Work item: [I160 #1093](https://github.com/guillermomolina/protos/issues/1093); parent [D245 #1071](https://github.com/guillermomolina/protos/issues/1071).
- Implementation evidence: [I160 implementation and JVM acceptance (2026-11-09)](I160_D245_C_ARGON2ID_PASSWORD_IMPLEMENTATION_PUBLICATION_2026_11_09.md).
- Product implementation published at [`9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd`](https://github.com/guillermomolina/protos/commit/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd), version `0.3.458-SNAPSHOT`. This is the **implementation publication revision**, not a measured exact HEAD of the new validation environment.
- The user executed the requested commands in a Protos working environment on 2026-11-10 and supplied the terminal output in the I160 closure conversation. The exact Git HEAD or binary digest during that run was **not supplied**.

## Rebuilt Native Image validation — owner reported

The first command was `make build-native`, which finished with Maven **BUILD SUCCESS** (`Total time: 03:32 min`, `Finished at: 2026-11-10T04:15:03Z`). This produced the local executable `target/native/protos`, subsequently invoked in the same provided run.

| Native invocation | Reported result |
| --- | --- |
| `PROTOS_HOME="$PWD" target/native/protos test --file protos/tests/library/crypto/password/vectors-and-nfc.protos` | **3 passed, 0 failed** |
| `PROTOS_HOME="$PWD" target/native/protos test --file protos/tests/library/crypto/password/invalid-records.protos` | **9 passed, 0 failed** |
| `PROTOS_HOME="$PWD" target/native/protos --allow-entropy protos/examples/crypto/password-argon2id.protos` | Printed three `true` values: valid password, invalid-password rejection, current-record profile |

**Password Native result: 12/12 suite assertions PASS, plus end-to-end secure-Entropy example PASS**, on the locally rebuilt Native executable. No run of a separately installed distribution was reported; this evidence specifically satisfies the **rebuilt Native executable** alternative, not an installed-binary claim. Shell exit codes were not printed separately, but the terminal transcript returned to the prompt after successful test summaries and example output.

The Native runtime emitted its expected Truffle fallback-runtime/interpreter-only warning. It is a runtime-compilation/performance warning, not a suite failure; no guest native JIT or performance result is claimed.

## Closure reconciliation

- Previous I160 evidence already records successful focused Java/Protos suites, full `make test`, full `make check`, clean staged diff and published source.
- This separate reported run discharges the originally open **rebuilt Native Image Password/Argon2id** gate without asking for new general tests.
- Live GitHub native issue hierarchy verified: `I160 #1093/parent = D245 #1071`, and reciprocal `D245 #1071/sub_issues` contains I160. I160 has no listed `blocked_by` dependencies at the time of inspection.
- **I160 may be closed as `completed`** on the combination of earlier publication/JVM acceptance and this freshly reported rebuilt Native acceptance.
- Do not infer that the broader `make test-native` suite ran here, that an installed distribution was independently tested, or that this run proves complete performance, cryptographic certification, or any other unrelated acceptance.

No product files, tests or version numbers were changed by the closure documentation.
