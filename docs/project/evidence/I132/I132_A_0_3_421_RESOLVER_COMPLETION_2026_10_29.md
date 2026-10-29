# I132-A — Protos-first Resolver and explicit name resolution (0.3.421)

**Date:** 2026-10-29. **Role:** immutable, non-normative implementation/publication evidence for [I132 #990](https://github.com/guillermomolina/protos/issues/990), consuming the owner-selected D223-A1 design in [D223 #987](https://github.com/guillermomolina/protos/issues/987). Live Issue state and normative semantics remain in `guillermomolina/protos`.

**Exact product revision:** [guillermomolina/protos@aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9](https://github.com/guillermomolina/protos/commit/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9).
**Published commit subject:** `I132-A: implement Protos-first D223-A1 Resolver and explicit name resolution (0.3.421)`.
**Implementation version:** `0.3.421-SNAPSHOT`. **Normative specification revision:** `0.1.480` ([`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/PROTOS_SPEC_CHANGELOG.md)).
**Normative owners:** [`spec/io/NETWORK.md` §14](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/NETWORK.md), [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/PROCESS_IO.md), with bounded cross-references in `IO_CORE.md`, `FUTURES_AND_TASKS.md`, `ACTORS.md`, `MODULES.md` and `ERRORS.md`.

## Published implementation

The 56-path product commit contains 8 specification paths, 4 Standard Library/Core Protos paths, 4 Resolver Protos conformance sources, 2 bundled Test Tool suite-planning sources, 20 production Java paths, 13 Java test/audit paths, and 5 implementation metadata, guide or example paths.

- `std:network/Resolvers` exports the authority-free `Resolver` protocol and `ResolutionError`. Only an explicit host-provisioned, bootstrap-local `resolver` grant permits a lookup; importing a module, receiving a `network` capability or having socket permission never implies it.
- `Resolver.protos` owns guest-visible API and argument rules, and shared `std:network/internal/DnsNames` owns strict NR-LDH/IDNA2008 A-label admission. The shared Unicode-facts facility supplies the narrow host facts without adopting a second name-validation policy in Java; TLS identity retains its separate two-label and wildcard rules.
- An authorized `lookupAddresses` produces a standard operation Future with fresh, frozen, canonical, deduplicated `IpAddress` results. At most 64 distinct addresses and 256 raw answers are admitted. `ResolutionError.kind` has exactly `noAddress`, `unavailable`, `denied` and `limit`; invalid arguments, closed use and cancellation retain their respective Future/error contracts.
- Each Resolver has its own bounded `InetAddress` backend (at most four lazy workers and 64 queued requests), lifecycle and actor-targeted result publication. A Resolver grant binds to one Process and is retired at Process termination, failed bootstrap or provisioning-host closure. Backend workers never execute Protos guest code.
- An `IpAddress` returned by lookup is transferable ordinary data; the live Resolver capability is not transferable to another Actor or P. Existing numeric `Network.connectTcp(IpEndpoint)`, TLS reference hostname and HTTP `Host` remain distinct. The [example](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/examples/io/hostname-resolution.protos) illustrates explicit composition and requires two independent host grants.
- The explicit D122 Test Tool suite graph now registers `protos/tests/library/network/resolvers/{surface,admission,results,lifecycle}.protos` through the corpus and suite-planning sources. The published changelog records 11 Resolver logical cases under the normal `make test` Protos phase. This work does **not** purport to audit all other historical corpus paths; that separate inventory is [TEST010 #991](https://github.com/guillermomolina/protos/issues/991).

## Validation evidence and provenance

```text
WORK_ITEM=I132-A
PROTOS_REVISION=aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9
PRODUCT_VERSION=0.3.421-SNAPSHOT
GLOBAL_SPEC_REVISION=0.1.480
COMMIT_CHANGED_PATHS=56
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
EXTERNAL_PUBLIC_DNS_TEST=NOT_CLAIMED
NATIVE_IMAGE_BUILD_OR_EXECUTION=NOT_CLAIMED
```

Guillermo reported on 2026-10-29 that the exact I132-A change was committed and pushed, `git diff --check` was clean, and local `make test` and `make check` both passed. That report supersedes an earlier *pre-fix* `make test` failure caused by a missing audited `nativeClosure` count for the Resolver host bridge in `ProtosCoreNativeBoundaryArchitectureTest`. The final product commit includes the audit correction. The complete Protos phase was reported passing in the final run, rather than being inferred from the earlier Java-only run.

These are **human-reported gates**, not commands rerun by the documenting agent. No live public DNS, OS name-service behavior across platforms, or independent Native Image compatibility test is claimed by this record.

## D223 decision consistency: closure still gated

The original D223-A1 owner approval, recorded in [D223 #987](https://github.com/guillermomolina/protos/issues/987) on 2026-10-29, selected independent Resolver authority and included this isolation invariant: `Backend and caches must not cross resolver-authority/Context domains observably.`

The published `NETWORK.md` §14.9 narrows the cache guarantee to **Protos-managed** mutable cache/result state, explicitly excluding caches inside the host-authorized OS/JVM name service. This is a real observable qualification with the `InetAddress` adapter: a JVM-wide cache can influence multiple Resolver grants. The published specification changelog characterizes the qualification as owner-approved, but the available D223 approval record does not establish **separate explicit owner approval of that exact change**. Neither a passing test suite nor the product commit alone supplies that design provenance.

```text
NORMATIVE_PRODUCT_PUBLICATION=PASS
ORIGINAL_D223_A1_OWNER_SELECTION=RECORDED
D223_HOST_CACHE_INVARIANT_DELTA=EXPLICIT_OWNER_REAPPROVAL_NOT_YET_VERIFIED
GITHUB021_DECISION_INVARIANT_CONSISTENCY=PENDING
D223_DURABLE_RATIFICATION=NOT_CLAIMED
I132_FULL_ISSUE_CLOSURE=AWAITING_D223_GATE
```

The owner must explicitly approve the published §14.9 host-cache exception (or direct a specification/implementation reconciliation) before a D223 ratification record can assert `DECISION_INVARIANT_CONSISTENCY=PASS` and before I132's own ratification-dependent final acceptance condition can be marked complete. This document records completed implementation work without silently redefining the approved D223 decision. Upon resolution, the separately revision-bound D223 closure record and live Issue updates should cite both the product SHA above and the exact project-documentation commit SHA.

## Scope that remains separate

D223-A1 intentionally defers connect-by-name, Happy Eyeballs, public TTL/cache APIs, `std:dns` records, DNSSEC/DoH/DoT, name-search expansion, Actor delegation, deadlines/timeouts, routing, URI/REST/redirect behavior and TLS trust-policy changes. They are neither implemented nor implied by the I132-A validation report.
