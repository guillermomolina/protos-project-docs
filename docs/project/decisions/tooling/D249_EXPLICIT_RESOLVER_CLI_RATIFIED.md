# D249 — Explicit Resolver permission for standalone Protos CLI

**Status: RATIFIED — owner-approved D249-A1, 2026-11-10.** This is a non-normative tooling/CLI authority decision. It does **not** assert that the CLI flag has been implemented, that the product specification has been modified, or that JVM/Native tests have run.

- Decision: [D249/#1075](https://github.com/guillermomolina/protos/issues/1075).
- Implementation: [I164/#1109](https://github.com/guillermomolina/protos/issues/1109), independently open top-level implementation item.
- Governing prior decisions: [D223-A1/#987](https://github.com/guillermomolina/protos/issues/987) (Resolver authority) and [D247-C/#1073](https://github.com/guillermomolina/protos/issues/1073) (independent CLI Network and service permissions).
- Other independent authority: [D248-B/#1074](https://github.com/guillermomolina/protos/issues/1074) (Filesystem); [D250/#1080](https://github.com/guillermomolina/protos/issues/1080) (future Process/session delegation remains unselected).
- [Investigation, owner-approval evidence, alternatives and GITHUB021 invariant audit](../../evidence/D249/D249_A1_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_10.md).
- **Inspected product baseline (not implementation):** [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5).

## Exact selected public contract

The standalone CLI accepts **one optional boolean `--allow-resolver`**, disabled by default. It is independently selectable from `--allow-network`, `--allow-filesystem=<directory>`, `--allow-entropy`, `--allow-clock`, inspection, foreign-provider selection and `--service=<binding>`. A Resolver-only grant, without Network, is valid and supports resolution but not TCP connection authority.

The flag selects the **existing** `ProtosStandaloneHostedExecution.ResolverGrant.HOST_RESOLVER` on the application's Process-hosting runtime. With the flag, the existing live host-provisioned Resolver is installed as the bootstrap-local `resolver` slot on the first RootActor SOURCE or MODULE private initializer context **before** source execution. Without the grant the slot is absent, not `null`. The grant is not a Prelude/global binding, imported-module grant, service locator, Process accessor, or automatically inherited Actor/P/child-Process authority.

Examples of eligible invocation:

```sh
protos --allow-resolver app.protos
protos --allow-resolver -e 'print(context.hasSlot("resolver"))'
protos --allow-resolver                         # REPL
protos run --allow-resolver app
protos debug --allow-resolver app.protos
protos --allow-resolver --service=service app.protos
protos run --allow-resolver --service=service app
protos --allow-network --allow-resolver app.protos
```

These are **selected future CLI invocations**, not claims that current binaries already accept them.

Only direct SOURCE, `-e`, REPL, application `protos run`, application `debug`, and direct/workspace RootActor services may request the grant. Bundled Tools (including `test`, `lint`, `package`, formatting and other tool modes), language server and non-application help/version operations may not. A tool's own resource custody is unchanged.

The option occurs at most once, within the pre-entry option region (or after `run`/`debug` and before the entry). Duplicate flags, value-form `--allow-resolver=...`, ineligible modes and malformed pre-entry use result in CLI usage failure (exit 2). Entry-following tokens retain the existing application-arguments interpretation; they do not become privilege grants. No prompt, environment-variable grant, automatic DNS fallback or `--allow-network` implication is selected.

## Exact governing semantics retained unchanged

`spec/io/NETWORK.md` §14 (D223-A1) and `spec/io/PROCESS_IO.md` Root resolver capability provisioning continue to own all guest-visible behavior. A resolver independently returns a `Future<Array<IpAddress>>` for admitted ASCII/strict IDNA2008 DNS names, without implicit numeric literals, U-label mapping or search-domain expansion. Result arrays are fresh/frozen, structurally deduplicated, bounded to 1–64 canonical addresses, with advisory order and no TTL/cache metadata or network authority.

`ResolutionError` kinds remain `noAddress`, `unavailable`, `denied` and `limit`; invalid arguments fail the Future with `InvalidIOArgument`, post-close calls with `IOLifecycleError`. Ordinary Future cancellation, Actor-domain outcome commitment, bounded host-only workers, late-result suppression, idempotent close and host/Process retirement are unchanged.

The name service may use host DNS, local hosts or split-horizon sources. Private/loopback addresses are not universally filtered. No DNSSEC claim, DNS result authentication, automatic connection, TLS identity change, implicit retry or endpoint authorization is added. `network.connectTcp(IpEndpoint)` remains numeric and separately authorized; original hostname remains independent HTTP Host / TLS reference identity.

`Resolver` is a live nontransferable authority, never automatically supplied to imports, new Actors or P. Completed immutable `IpAddress` data are ordinary transferable values. No Protos-managed mutable cache crosses Resolver authorities/Contexts; cache internal to the host-authorized service is governed by the **already approved** D223 §14.9 host-cache exception.

## Deliberate deferrals and extension path

Do **not** introduce `--allow-resolver=<policy>`, domain allowlists, per-query prompting, named DNS providers, DNS servers/DoH/DoT/DNSSEC, TTL/guest cache, portable timeout/deadline or Actor/child-Process delegation in this implementation. These require their own future design if demonstrated necessary. Existing embedding can supply host-selected resolver authority; future restricted CLI grants may be added without weakening the independent-grant contract. D250 must remain independent.

The strongest objection to A1 is that a broad host Resolver may disclose queried names and reveal private/loopback address information without fine-grained domain restriction. A1 deliberately makes that authority explicit, default-off, local to a Process, while keeping policy enforcement at the host boundary. A new public policy DSL would charge current users for unestablished requirements.

## Required implementation / release boundary

I164 owns one coherent end-to-end implementation across CLI parsing/help, direct hosted execution, debugger, REPL, workspace/package run and service bootstraps, grant ownership/cleanup, normative CLI provisioning wording in `spec/io/PROCESS_IO.md`, user guides/examples, negative/adversarial regression tests and JVM/Native verification. The implementation must handle the currently hardcoded `ResolverGrant.NONE` direct/debug seams and `resolver=null` workspace/bootstrap seams. D249 itself does not authorize unrelated DNS protocol refactors.

A test plan must distinguish deterministic backend checks from optional external name-service integration; do not assume public DNS availability or fixed IPv4/IPv6 ordering. This record carries **no** PASS claim for builds, native-image execution, end-to-end DNS, tests or release. Protos implementation/publication remains subject to the human-executor policy in `guillermomolina/protos/AGENTS.md`.
