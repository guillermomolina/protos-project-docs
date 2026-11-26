# BUG042 — CPrime I/O carrier publication and validation evidence

Date: 2026-11-26

Live coordination: [BUG042 #1186](https://github.com/guillermomolina/protos/issues/1186).

## Exact product publication

- Protos revision: [43f65f627d86260b6bd76ad3b8877f71146b4f97](https://github.com/guillermomolina/protos/commit/43f65f627d86260b6bd76ad3b8877f71146b4f97), verified as published main at inspection.
- Commit: `BUG042: Make CPrime buffered reader and I/O release outcome carriers valid Truffle values`.
- Implementation version: `0.4.225-SNAPSHOT` (previously `0.4.224-SNAPSHOT`).
- Exact commit diff: four paths — `CHANGELOG.md`, `pom.xml`, and `src/main/java/com/guillermomolina/protos/execution/host/resources/{ProtosBufferedByteReaderCPrimeExecution,ProtosIoReleaseCPrimeExecution}.java`. No test, specification or external-child custody path changed.

## Defect and correction

The initial implementation report attributed 42 test errors to two CPrime root result carriers. The published changelog identifies automatic root instrumentation encountering a host ClassCastException when observing their completion.

The exact published diff adds the `TruffleObject` marker interface to `ProtosBufferedByteReaderCPrimeExecution.LowerOutcome` and `ProtosIoReleaseCPrimeExecution.Outcome`. Their existing payloads, constructors, visibility and control paths are unchanged. They remain internal runtime transports and export no guest members or operations; the release carrier retains its existing Java public nested-class declaration, not a public guest API. The existing [text-writer carrier](https://github.com/guillermomolina/protos/blob/43f65f627d86260b6bd76ad3b8877f71146b4f97/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosTextWriterCPrimeExecution.java) independently confirms the local `LowerOutcome implements TruffleObject` precedent.

No new semantic contract is introduced. Read commitment/cancellation and close/ownership behavior are intended to remain governed by IO_CORE.md and BYTE_IO.md. The isolated diff does not change these paths' logic. The complete APL Part 5 notice is restored in the buffered-reader file, and both Java source headers were re-read at the exact revision and match the Java notice in AGENTS.work/REFERENCE.md.

## Validation and limitations

- Maintainer reports the corrected revision was pushed, `git diff --check` is clean, and all local tests pass (2026-11-26).
- These are maintainer-reported post-patch results, not independently executed by the coordinating agent. The earlier pre-patch `make check` is not counted as post-patch evidence.
- The exact original failing revision, 42-error test inventory and exception transcript, separate negative-control regression result, external Truffle-language precedent citation, and individual post-patch commands/test counts are not supplied. No separate regression source was added by this commit. The existing local text-writer precedent was independently verified. No remote CI or Native Image acceptance is claimed.
- The four-path publication is isolated from BUG041. No dependency or parent is inferred.

The publication and owner-reported local validation are established; outstanding acceptance evidence is explicitly retained in the live Issue. This durable record is evidence, not a normative specification or independent lifecycle authority.
