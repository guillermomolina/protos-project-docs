# D270-D — Comparative research and owner approval evidence

2026-11-26. Non-normative selection evidence, not final ratification.
[D270 #1201](https://github.com/guillermomolina/protos/issues/1201), [I184 #1204](https://github.com/guillermomolina/protos/issues/1204).
Intake baseline PROTOS_REVISION=1afc4ceeb4b6df9878fb2d2f168235277f8e7e70 (issue-declared; main reads were not pinned to this revision).
Owner's active-interaction response to the presented D270-D recommendation: “ok aprubeo la recomendación.” The same message authorizes direct bounded publication to this repository. Approval does not cover unpresented boundary decisions.

## Comparative packet
Six precedents, three distinct approaches (destructive transport termination, resource-scoped deadlines, ownership/cancellation):
- [Python asyncio transports](https://docs.python.org/3/library/asyncio-protocol.html#asyncio.WriteTransport.abort): abort discards buffered data; graceful close differs.
- [Node.js net](https://nodejs.org/api/net.html): destroy versus explicit resetAndDestroy; no mandatory TCP RST needed.
- [Go net.Conn](https://pkg.go.dev/net#Conn): Close unblocks I/O, resource deadlines affect pending operations; incompatible as a replacement for existing Protos close.
- [Tokio AsyncWriteExt](https://docs.rs/tokio/latest/tokio/io/trait.AsyncWriteExt.html): cancellation safety varies with operation; partial output cannot be undone.
- [Erlang gen_tcp](https://www.erlang.org/doc/apps/kernel/gen_tcp.html): send_timeout_close and socket close; connection retirement after uncertain output.
- [Java SocketChannel](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/channels/SocketChannel.html): asynchronous channel closure can interrupt pending blocking I/O.
Host considerations: [Linux close](https://man7.org/linux/man-pages/man2/close.2.html) may leave blocked I/O holding underlying open file description; descriptor reuse hazard; [Winsock closesocket](https://learn.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-closesocket) distinguishes graceful/abortive closure and outstanding overlapped I/O. BSD/Darwin and exact backend portability still require reconciliation. Links identify comparative sources; this record does not certify fresh full reads of every external source.

Candidates: A external protection only (insufficient for internal or outbound I/O); B redefine close (invariant violation); C universal Abortable (unjustified generalization); D explicit TCP abort (selected, smallest sufficient); E destructive per-operation deadlines (conflates ownership/time/policy).
Presented preliminary scoring, in GITHUB010 criterion order (correctness, alignment, present need, growth, future options, scalability, simplicity, portability, runtime cost, operability, reversibility, maturity):
A 3,3,4,2,3,3,5,4,4,2,3,5 = 41;
B 1,1,4,2,2,4,2,3,3,2,1,4 = 29;
C 3,2,1,3,4,3,2,2,2,3,4,3 = 32;
D 5,5,5,5,5,5,4,4,5,5,5,3 = 56;
E 2,2,2,3,3,3,2,3,3,3,2,3 = 31.
Use table totals, not erroneous preliminary chart. Scores provisional; full per-cell rationale/confidence audit remains pending. B disqualified regardless of arithmetic; C fails proportionality; A remains complementary external defense. D avoids charging unrelated I/O or ordinary Futures; extension to other transports can be evaluated independently.

## Source evidence actually inspected
GraalPy official repository, moving default branch (exact upstream revision not verified):
- [SocketBuiltins.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/socket/SocketBuiltins.java): CloseNode.close lines 371–389, @Builtin, @GenerateNodeFactory, @Specialization, invalidates fd before host close and releases/reacquires GIL; ShutdownNode.shutdown lines 1031–1047 delegates POSIX shutdown and raises portable Python error.
- [EmulatedPosixSupport.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/EmulatedPosixSupport.java): close line 432 and shutdown line 3359 @TruffleBoundary; EmulatedSocket close lines 4163–4172 calls clientChannel.close/serverChannel.close, neverPartOfCompilation; shutdown lines 4348–4357 calls shutdownInput/shutdownOutput.
- [NativePosixSupport.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/NativePosixSupport.java): @TruffleBoundary close lines 787–795 and shutdown lines 2664–2671, native invocation and errno mapping.
These support resource invalidation and host close/shutdown, NOT a proof of equivalent async custody. PosixSupportLibrary.java lookup failed. Other relevant Truffle languages and actual Protos backend must be inspected before any Java production change. No invented infrastructure.

## GITHUB021 and attempted falsification
Selected D preserves D269 non-owning observation, existing four Future states, committed prefix/no rollback, zero logical read consumption, cooperative Actor termination, explicit capability/isolation, late acquisition custody, Error identity, ordinary close absent explicit abort, TLS authenticated EOF, borrowed HTTP authority. Explicit destructive TCP cutover during closing is the intentional new delta. Exact in-flight backend effect frontier, half-close interactions, release failure precedence, cleanup and wrapper behavior must be reconciled, with material new choices returned to owner.
Counterexamples: peer stops reading after prefix; pending read with no input; cancel wins before cutover; committed result callback delayed; two abort calls; close then abort; descriptor reuse; Actor/Process death; release failure; TLS missing close_notify; borrowed HTTP; quota reuse before retirement; non-suspending handler; late acquired socket; large connection counts and non-TCP programs. No guarantee of hard real-time progress or peer receipt/reset.

## Validation and limitations
Planned deterministic admission/commit/retirement races, real nonreading TCP peer, partial writes, read zero-consumption, fresh Futures and recorded Error identity, close/abort/cancel and half-close, late callbacks, descriptor reuse, backend release failures, Actor/Process custody, TLS/HTTP authority and quota, isolation, bounded resource retention and no unrelated hot-path cost. JVM and installed Native Linux x86_64 CLI; no ARM. Human executes focal/integrated tests. No product changes, builds or tests performed.
Inspected AGENTS.md, AGENTS.work/DESIGN.md, REFERENCE.md, COORDINATION.md, spec/AGENTS.md and normative IO_CORE, BYTE_IO, NETWORK, TLS, HTTP, FUTURES_AND_TASKS, ACTORS, DISTRIBUTED_RUNTIME, ERRORS; output truncation means complete whole-document review not certified. Full exact normative owner reconciliation, revision-bound source audit, invariant/delta check and ratification remain pending. D270 remains open and I184 blocked.
