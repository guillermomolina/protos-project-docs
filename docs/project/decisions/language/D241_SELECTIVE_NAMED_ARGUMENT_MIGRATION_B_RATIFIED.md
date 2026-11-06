# D241 — Selective named-argument migration of temporary configuration bags (B ratified)

**Status:** OWNER-RATIFIED, 2026-11-06. **Implementation:** NOT STARTED. Durable, non-normative design record; Protos `spec/` remains the normative authority after publication of an Ixxx.
**Design issue:** [D241 #1058](https://github.com/guillermomolina/protos/issues/1058).
**Comparative audit:** [method-by-method evidence](../../evidence/D241/D241_METHOD_BY_METHOD_REVIEW_AND_APPROVAL_2026_11_06.md), [full issue research](https://github.com/guillermomolina/protos/issues/1058#issuecomment-6665341495).
**Inspected product:** [guillermomolina/protos@384966eec789831bdc954f44d3d17e891fd1cb19](https://github.com/guillermomolina/protos/commit/384966eec789831bdc954f44d3d17e891fd1cb19) (I145-A, Maven 0.3.446). This is a read-only research baseline, not an implementation of D241.
**Owner approval provenance:** Following the research packet and exact method-level migration/retention recommendation, including explicit generic-Error/InvalidIOArgument compatibility disclosures and the proposed UrlClient authority fields, the owner stated on 2026-11-06: **“apruebo el cambio con el alcance propuesto. Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs.”** Earlier in the same decision the owner corrected the attempted blanket exclusion of Filesystem, HTTP and TLS, and directed D241 itself—not a research Ixxx—to investigate method by method.

## 1. Selected model (D241-B)

Replace **temporary, fixed-field, call-only configuration carrier objects** with explicit D237 named arguments on the *actual selected callable*. Objects remain real first-class values for reusable policy, recursive graphs, dynamic Maps, HTTP message data, capabilities and Actor payloads. Validated/frozen **internal configuration snapshots are retained** and need not be public objects. No parser change, new Core type, implicit Object→keywords conversion, named-rest capture, automatic foreign kwargs, method overload, or new keyword-only formal category.

**D237 MIX-C remains unchanged:** labels are associated with non-rest formal names of the dynamically selected Closure after receiver/argument evaluation; positional arguments retain fixed ordinals. Source signatures are not enforced keyword-only. Public formal names are a contract. Only explicitly published Protos-semantic native signatures may accept such names.

## 2. Public APIs selected for conversion

| API | Exact selected options / fields | Preservation |
|---|---|---|
| `CommandLine.option(descriptor)` | Replace with 9 formals `key, long, short, minValues, maxValues, minOccurrences, maxOccurrences, valueName, help`. **All nine required**, even nullable fields. | Fresh, frozen canonical OptionSpec; validations |
| `CommandLine.positional(descriptor)` | Replace with 5 formals `key, minOccurrences, maxOccurrences, valueName, help`. **All five required**. | Fresh frozen canonical PositionalSpec |
| `filesystem.open(path, options)` | Keep `path` and the `open(path)` default form. Add named `read, write, create, createNew, truncate`. | `read=true`; others `false`; exact Boolean/combination checks, before-I/O snapshot and failed Future on invalid **bound values** |
| `network.listenTcp(localRequest)` | Replace request object with three required formals `ipVersion, address, port`; explicit `null` allowed for address/port. | Version/port validity, failed Future on invalid bound values, prior-to-backend admission |
| `http.Host.owning(listener, handlerModule, handlerBinding, options)` | Replace top-level bag with named `workers, maxConnections, maxPendingAccepts, limits`. | Defaults 1, 64, `workers`, `null` respectively; `limits` remains a reusable object, all worker quotas/custody |
| `http.Host.owningWithFactory(listener, factoryModule, factoryBinding, initialization, options)` | Same four named options; `initialization` remains its separate positional payload. | No transfer of handler Closure, listener custody and Worker init copies |
| `tls.Client.handshake(duplex, serverIdentity, trust, options)` and `handshake.owning` | Replace options bag by `minimumVersion, alpn, maxHandshakeBytes, maxQueuedWriteBytes`. | Defaults `"1.2"`, `"http/1.1"`, `1048576`, `33554432`; explicit `alpn: null` disables; strict preflight and borrowed/owning lifecycle |
| `http.UrlClient.requestOne(authorities, url, request, limits)` | Replace the authority bag with **mandatory named** `network, resolver, trust`; keep `url, request, limits` as independent data arguments. Recommended usage: `requestOne(url, request, limits, network: net, resolver: dns, trust: roots)`. | All three capabilities must be supplied explicitly, including `null` where the existing scheme/domain allows it; no ambient authority, preserved HTTP request and reusable limit policy |

The selected names, requiredness, defaults, retained value roles and public call behavior are fixed. Internal helper method arrangements, source adaptation and exact private tuple layout are implementation details, not permission to change this API contract.

## 3. APIs retained without configuration-bag migration

- `CommandLine.command(descriptor)` receives a **recursive descriptor graph** of subcommands, options and positionals; keep raw nested descriptor objects/Arrays, identity-sharing/cycle detection, depth-independent 4096-level iterative normalization, fresh canonical frozen results. `parse(specification, arguments)` and `renderHelp(rootSpec, commandPath)` keep their domain inputs. Internal command normalization must not assume it can still invoke the new `option(rawDescriptor)` selector.
- `http.Server.serveOne` and `serveOne.owning` retain the **9-field reusable server `limits` policy**, shared with Host. `http.Client.exchangeOne`/owning and `UrlClient.requestOne` retain the **10-field reusable client `limits` policy**. Keep full HTTP request/response/headers/route entries as domain records and `Router.compile(routes)` as a route-table Array.
- `network.connectTcp(endpoint)` retains canonical `IpEndpoint`; `tls.Trust.anchors(certificates)` retains certificate data; `logger.with(extraFields)`, `logger.log` details and `LogEvent` retain dynamic-key Map fields.
- `Logger.construct(minimumLevel,sink,timeSource)`, `ColoredFormatter.construct(mode,autoEnabled)`, `TextSink.construct(formatter,writer)` and `Test.construct(name,body)` already declare individual parameters; no duplicated alternative selectors.
- Actor message/payload graphs, Filesystem Paths/Files, Collections, JSON/TOML/CSV/Regex/URI/SemVer/crypto/math/datetime/text, and private Test/Package Tool inert tuples/documents are **not** temporary fixed options merely because their values contain fields.

## 4. Explicitly approved incompatibility / failure precedence

**Breaking public signatures:** old one-object argument entry calls for selected methods are retired. Internal/private source callers and user-facing tutorials/tests must be migrated intentionally. CLI option/positional previously did not globally reject unrelated extra own descriptor fields; UrlClient authorities likewise required three fields without globally rejecting every extra own slot. An unknown named label is now rejected, not silently tolerated.

**D237 error timing:** an unknown name, duplicate/colliding label or positional/named collision signals the **generic D237 `Error` before callee entry**, not an `InvalidIOArgument` produced inside the library. This is explicitly accepted even when the old options object would have caused a **failed `Future<InvalidIOArgument>`** on Filesystem/Network/TLS/UrlClient, or synchronous `InvalidIOArgument` on Host. **Validly bound options with invalid semantic values** must preserve their pre-existing domain-specific `Error` or failed-Future behavior and before-effects validation. Do not add a language-level fallback to disguise this difference.

**Required omitted vs explicit `null`:** CLI and Network listen retain mandatory presence checks. A private unique identity sentinel as a Closure parameter default is an allowed source-only mechanism, but no new `undefined`, public argument-introspection primitive or implicit defaulting of previously required fields is authorized. A present nullable `null` suppresses a default.

**Object construction is not a textual keyword list:** an object body may use its just-created own slots; named argument value expressions execute in the caller. Do not rewrite calls by simple textual substitution.

## 5. Owner invariants / GITHUB021 check

| Existing authority | Ratified treatment |
|---|---|
| D237/I143: selected Closure, stable positional ordinals and strict labels | PRESERVED; no Object→keywords or new named-rest |
| D203: defaults, absence vs null | PRESERVED by required presence checks |
| D013/D239/I145: ordinary Object.call→construct and selected-method dispatch | PRESERVED |
| LIB011/AUD006 canonical CLI graphs, snapshots and iterative depth | PRESERVED, **except explicitly ratified two public entry breaks** |
| Filesystem/Network: no backend effects on invalid semantic option tuples | PRESERVED after successful name association; unknown-name Error is approved pre-entry delta |
| HTTP Host quotas/Actor transfers; shared client/server limits policy | PRESERVED; Host top-level options and UrlClient authority bundle are approved input changes |
| TLS trust/custody/cleanup and explicit ALPN null | PRESERVED |
| Logger dynamic Map fields, Actor identity/transfer, ordinary object rules | PRESERVED |

**GITHUB021: PASS within the disclosed exact scope.** The owner's correction against blanket excluding snapshot-based IO configuration is incorporated; earlier observable input calls are intentionally replaced only where listed. No prior norm beyond the explicitly approved delta is reopened.

## 6. Candidate comparison and anti-overengineering

- **A status quo:** safe and cheap but preserves unwanted temporary input bags.
- **B selective replacement (SELECTED):** uses already-ratified D237, preserves reusable objects, intentionally pays API migration and error-shape cost.
- **C explicit dual entry selectors:** avoids source breaks but doubles public surface and ongoing maintenance; no current universal requirement.
- **D global implicit Object→named conversion:** REJECTED; conflicts with dynamic dispatch, ordinary object identity and D237 strict named binding.

The issue research includes **12 required GITHUB010 dimensions**, confidence, strongest counterargument, scalability/Actor/interop stress, source migration and seven-language prior art (Python, Ruby, JavaScript, Kotlin, Swift, Lua, Rust). **Present need:** improve call-only fixed option input without more language machinery. **Incremental path:** add future named APIs case-by-case; keep reusable policy objects now. **Cost of deferral:** growing number of callsites, not a fundamental VM rewrite. **Strongest objection:** breaking source and failure taxonomy mainly for ergonomic clarity, an explicitly accepted tradeoff. **Escape path:** a separately approved named adapter can be added for an actual compatibility requirement later, not prebuilt globally.

## 7. Implementation gate

This durable decision does not itself edit product source or specifications and does not prove tests. Publish independent Ixxx implementations for (1) CLI, (2) native Filesystem+Network, (3) HTTP Host+UrlClient, (4) TLS, with source migration, directly affected `spec/` updates, tests, guides and validation. Re-evaluate current HEAD; do not silently change D237. Retain the project's human-executor rules for tests/build and product commits, and bump versions only after tests, with no tests after the final bump. No speedup is presumed from avoiding a literal config object.

**Approval checkpoint closed; implementation evidence absent until separate product publication.**