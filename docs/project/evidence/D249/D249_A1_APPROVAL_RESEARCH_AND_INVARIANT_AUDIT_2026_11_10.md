# D249-A1 — investigation, exact approval, falsification and GITHUB021 audit

**2026-11-10 — Decision ratification evidence, not product implementation or test evidence.**

- Decision: [D249 #1075](https://github.com/guillermomolina/protos/issues/1075).
- [Exact ratified contract](../../decisions/tooling/D249_EXPLICIT_RESOLVER_CLI_RATIFIED.md).
- Implementation issue: [I164 #1109](https://github.com/guillermomolina/protos/issues/1109), still open.
- Product inspection baseline: [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5). This revision is **not** a D249 implementation.

## 1. Explicit owner selection and provenance

In the active conversation on **2026-11-10**, after presentation of the proposed D249-A1 exact contract and the comparative investigation, the project owner replied **“Apruebo D249-A1”** and authorized Issue updates and durable evidence publication. This is direct approval of the named A1 proposal, **not** a general authorization for new DNS policies, child-Process authority, backend rearchitecture or silently changed Resolver semantics.

This evidence is agent-written documentation of an approval given in conversation; it is not an owner-authored GitHub comment. The selected candidate is the simple independent boolean `--allow-resolver` (default absent) with all eligible application entries and independent Network authority, strict rejection in Tools/LSP and no implicit Actor inheritance. [D223-A1 #987](https://github.com/guillermomolina/protos/issues/987) continues to own resolver semantics. [D247-C #1073](https://github.com/guillermomolina/protos/issues/1073) explicitly did not grant Resolver via `--allow-network`.

## 2. Governing evidence inspected

| Source | Relevant observed constraint |
| --- | --- |
| `AGENTS.md` | Design and implementation authority separation; human-executor boundary; minimum repository-source audit |
| `AGENTS.work/DESIGN.md` | GITHUB010 comparative breadth and 12 dimensions; owner approval; GITHUB021 invariant/delta comparison |
| `AGENTS.work/COORDINATION.md` | GITHUB015 ratification/closure, formal Issue uniqueness, family/status and durable publication |
| `AGENTS.work/REFERENCE.md` | Small universe, orthogonality, no unused resource cost; DOC002 role-first record placement |
| `AGENTS.work/IMPLEMENTATION.md` | Implementation discipline and blocker/retest gate for I164 |
| `spec/io/NETWORK.md` §14 | Independent host Resolver; strict names/results/errors; bounded concurrent operations, cancellation, security, close and cache rules |
| `spec/io/PROCESS_IO.md` | Bootstrap-local Resolver only under explicit host grant; no Network/Filesystem/import/Actor inheritance; service entry follows same rule |
| `spec/io/IO_CORE.md` and `spec/semantics/MODULES.md` | I/O commitment and module bootstrap/import distinction; no new grant implied by reading a module |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | Current parser accepts independent Network/Entropy/Clock; direct/debug routes hardcode `ResolverGrant.NONE` |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedExecution.java` | Already has `ResolverGrant.NONE`/`HOST_RESOLVER` and provisions host capability |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedSession.java` | Direct hosted session currently passes Resolver NONE |
| `src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageApplicationExecution.java` | Workspace execution provisions Network/Entropy/Clock but passes null Resolver to Process bootstrap |
| `docs/guide/tools/cli.md`, `docs/guide/library/networking.md` | CLI usage currently documents Network without CLI Resolver grant |
| `src/test/java/com/guillermomolina/protos/cli/ProtosCliNetworkOptionTest.java` | Negative test expects Resolver absent with Network alone |
| `src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosNetworkBootstrapAuthorityTest.java` | Independently granted Resolver/bootstrap invariant tests |

No build, test, real DNS access, native-image invocation or validation script was run as part of this design-evidence publication.

## 3. Independent prior art and approaches

At least six relevant system approaches informed the decision. They are **analogies and precedents**, not new normative Protos authority:

1. **Deno:** explicit CLI network permission typically covers both resolution and connection; easy ergonomics, but Protos explicitly separates those effects.
2. **Node.js permissions / DNS:** broad network permission surface with differentiated `dns.lookup` and `dns.resolve` backend pathways; unsuitable as an authority model for independently authorized Resolver.
3. **WASI sockets / name lookup:** host-import/resource authority and a distinguishable IP name-lookup interface support separation from TCP acquisition; specific Preview-version surfaces differ.
4. **Bun DNS:** host, c-ares and `getaddrinfo` resolution backends illustrate why Protos should not freeze a provider choice in the CLI contract.
5. **Java `InetAddress`:** host-configured name service and optional host-managed caching illustrate host/VM differences that should not become Protos API guarantees.
6. **Protos Java embedding:** already implements `ResolverGrant.HOST_RESOLVER`; strongest local evidence that simple standalone CLI authorization can reuse current infrastructure without redesigning the guest API.

Meaningful approaches: broad combined network permission (Deno/Node), explicit provider/resource authority (WASI/Protos embedding), host/backend-configured DNS (Java/Bun), and deliberate deferral.

## 4. Candidate set and 12-axis GITHUB010 scorecard

A = **independent default-off `--allow-resolver` boolean (selected)**.
B = parameterized policy/value flag.
C = implicit grant bundled into `--allow-network`.
D = no standalone grant; embedding only.
E = caller-named external host resolver/provider required by CLI.

Scores: 1 worst, 5 best; comparative judgments are not measured throughput or claims of implementation.

| GITHUB010 dimension | A | B | C | D | E | Rationale / confidence |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| Correctness/invariants | 5 | 5 | 1 | 5 | 5 | C violates D223 authority separation. HIGH |
| Protos alignment | 5 | 3 | 1 | 3 | 3 | A composes existing explicit capability, no new institution. HIGH |
| Present-need proportionality | 5 | 2 | 2 | 2 | 2 | Only A meets current CLI needs without policy DSL. HIGH |
| Incremental growth | 5 | 4 | 1 | 3 | 4 | A permits later independent restricted grants. HIGH |
| Future-option resilience | 5 | 4 | 1 | 4 | 5 | A/E leave backend open; C collapses boundaries. MEDIUM |
| Scalability | 5 | 4 | 3 | 4 | 4 | Existing Resolver backend is bounded; no new shared scheduler required. MEDIUM |
| Conceptual simplicity | 5 | 2 | 3 | 4 | 2 | A introduces one opt-in switch; D stays simple but unusable in standalone. HIGH |
| Portability/freedom | 5 | 3 | 4 | 5 | 3 | A does not fix OS DNS API or backend. HIGH |
| Runtime/resource cost | 5 | 3 | 3 | 5 | 3 | No grant means no resolver machinery; optional host backend cost only when used. MEDIUM |
| Failure/operability | 5 | 3 | 2 | 3 | 3 | Existing Future/error/close rules stay authoritative. HIGH |
| Deferral/reversibility | 5 | 3 | 1 | 4 | 3 | A adds CLI opt-in without changing Resolver identity or behavior. MEDIUM |
| Evidence maturity/risk | 5 | 3 | 1 | 5 | 3 | A matches already-existing grant/provisioning API. HIGH |

**Non-compensating gate:** C is rejected as violating D223-A1 irrespective of numeric totals; B and E carry overengineering flags because no currently established domain-specific policy or provider-selection requirement justifies their public complexity. D is an honest conservative status quo, but leaves the standalone application gap unresolved.

## 5. Adversarial falsification and acceptance boundaries

- **No authority:** neither default invocation nor `--allow-network` alone binds `resolver`; absence is not `null`.
- **Independent resolver-only authority:** `--allow-resolver` with Network absent can resolve names but cannot create a TCP capability.
- **Tool privilege isolation:** no bundled Tool, language-server, package metadata/import or app-argument trick obtains Resolver due to the flag.
- **Actor and module isolation:** imports, spawned Actors, isolated P and child Processes do not automatically inherit Resolver; normal non-transferable authority remains.
- **Bootstrap/service lifecycle:** fail/terminate/close must retire host resources, suppress late callbacks and not leak live grants. Persistent message-service entry retains the grant until Process termination.
- **DNS/security:** input admission, `ResolutionError` kinds, cancellation outcome commitment, bounds, private/loopback results, IP order/IPv4/IPv6 and host cache limitations are exactly D223-A1, not new D249 policy.
- **JVM/Native:** test deterministic grant/denial and cancellation with controlled backend; real name-service tests are environment-conditional and must not rely on stable external DNS, address ordering or public connectivity.

A major implementation falsification target is the current workspace path with explicit `resolver=null`: parser-only implementation would appear to succeed while `protos run` and package services remained unauthorized. It must be fixed by I164.

## 6. GITHUB021 approved-invariant and delta check

| Pre-existing selected invariant | D249-A1 effect | Result |
| --- | --- | --- |
| D223: Network and Resolver independently authorized | Separate explicit boolean; permits Resolver without Network | PRESERVED |
| D223: absent bootstrap slot without host grant | Default OFF and slot absent | PRESERVED |
| D223: no authority from import, Prelude, Process or service locator | Only initial RootActor module context receives grant | PRESERVED |
| D223: no automatic Actor/P transfer | No new propagation or transfer API | PRESERVED |
| D223 §14.9 approved host-managed DNS cache exception | No public cache/TTL or new cross-authority Protos state | PRESERVED |
| D223: numeric `connectTcp(IpEndpoint)` and advisory IPs | No connect-by-name or SSRF policy inference | PRESERVED |
| D247-C: Network and service are distinct CLI selections | Resolver is third independent opt-in | PRESERVED |
| D248-B: filesystem is independent/confined | Resolver does not imply filesystem | PRESERVED |
| D250: child Process/session authority remains unselected | No automatic child-Process inheritance or attenuation | PRESERVED |

**DECISION_APPROVAL_PROVENANCE=PASS** (exact owner statement in active conversation, 2026-11-10). **DECISION_INVARIANT_CONSISTENCY=PASS** (no conflicting previously approved invariant identified). No invariant is reopened. A1 adds only a CLI host grant selection and the associated entry eligibility/usage behavior. The strong counterargument—coarse DNS disclosure authority—is accepted as an explicit default-off grant rather than smuggled into Network; domain policies are deferred to separately justified design.

## 7. Incremental-design and future stress questions

- **Smallest sufficient:** boolean app-only CLI selector over existing host Resolver; no policy parser or new host name-service facility.
- **Pay for what you need:** simple programs and Tools pay no DNS authority or backend cost. Only selected application sessions request provisioning.
- **Grow as you need:** specialized policy/provider grants can later coexist as separate opt-ins or embedding configuration while preserving Resolver interface and independent authority.
- **What if deferred:** the application/CLI/packaging plumbing is still required later; no advantage to keeping ordinary hostname resolution embedder-only.
- **What would cause regret:** environments needing per-domain DNS restrictions could consider an unrestricted flag too broad. Escape path: independent narrowed provider grants, preserving explicit capability semantics; do not retroactively imply restricted policy from this boolean.
- **Concurrency/OS/Native:** bounded workers and Actor-domain completion follow D223; host DNS differences stay behind provider; no new Truffle execution or Bytecode DSL contract.
- **Cancellation/long-lived service:** no new guest threads or implicit Task per lookup; Process/host lifetime owns revocation.
- **Distribution:** child Process grants remain a D250 design, not an implication of CLI selection.
- **Compatibility:** additive permission, existing invocations unchanged; no default-on migration.

## 8. Publication and verification boundary

The D249 decision record is durable in `guillermomolina/protos-project-docs` and references [I164 #1109](https://github.com/guillermomolina/protos/issues/1109) for future implementation. This record names the exact inspected Protos baseline and deliberately records no implementation or validation outcome. Human execution of focal tests, `make test`, `make check`, Native integration and product publication remains outstanding under I164, and must not be conflated with D249's completed design ratification.
