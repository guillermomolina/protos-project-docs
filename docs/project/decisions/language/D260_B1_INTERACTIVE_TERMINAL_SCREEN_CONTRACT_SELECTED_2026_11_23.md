# D260-B1 — Protos-first interactive terminal Screen contract

**Status: EXACT OWNER-APPROVED DESIGN (2026-11-23); PRODUCT NORMATIVE SPECIFICATION AND LIB032 IMPLEMENTATION NOT YET PUBLISHED.**

This is a **non-normative** durable decision record. The Protos specification under `guillermomolina/protos/spec/` remains the sole authority for observable Standard Library semantics. The owner's approval is not proof of implementation or native backend feasibility.

- [D260 #1154](https://github.com/guillermomolina/protos/issues/1154) owns this decision.
- [LIB032 #1152](https://github.com/guillermomolina/protos/issues/1152) owns eventual implementation.
- [PLAT078 #1155](https://github.com/guillermomolina/protos/issues/1155) separately owns native backend/ABI feasibility: **ncursesw is the first candidate, termbox2 the mandatory comparator, neither selected definitively**.
- [D260 B1 approval and GITHUB010/GITHUB021 evidence](../../evidence/D260/D260_B1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_23.md).
- **PROTOS_REVISION inspected:** [9c56aba2c8b84a9c44440cb361db85b53fcf30b4](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4); no new D260 normative specification is published at this revision.
- **Exact approval provenance:** after the question "¿Apruebas D260-B1 con el contrato público y las reglas exactas expuestas arriba, manteniendo en PLAT078 la obligación de demostrar su viabilidad antes de implementar LIB032?", the owner answered **"Apruebo D260-B1"**, then explicitly authorized issue coordination and direct commit/push of the documentation evidence on 2026-11-23.
- The earlier owner-approved direction was Protos-first, existing trusted D259/PLAT077 FFM, no terminal-specific Java infrastructure, no initial widgets/GUI/WUI and conditional A/B backend viability.

## Exact selected public B1 surface

A source-backed, ordinary Protos MODULE imported as:

```protos
Screen: import("std:terminal/screen")
```

**Import is inert:** it neither acquires a TTY nor loads a native library or grants FFM authority. Its public operations:

| Operation | Selected public behavior |
| --- | --- |
| `Screen.openControlling()` | Explicit acquisition of a new Screen on the process's controlling terminal, or failure |
| `screen.size()` | Fresh, frozen ordinary object with exact Integer `rows` and `columns` from one coherent observation |
| `screen.clear()` | Clear pending logical cells; return the same Screen |
| `screen.put(row, column, text, foreground: "default")` | Write text into pending logical cells only; return same Screen |
| `screen.present()` | Request physical presentation of pending logical state; return same Screen on success |
| `screen.nextEvent(timeoutMs)` | Synchronous input/poll with mandatory timeout; return one frozen event or `null` |
| `screen.close()` | Standard `Closable` operation returning a fresh Future for each invocation |

No public `WINDOW*`, `SCREEN*`, C ABI structs, numeric native pointer, escape sequence writer, mutable public Grid, mandatory Frame, callback loop, new Core syntax, primitive, prelude binding, Java class, Truffle node or alternate foreign provider.

## Authority, Screen identity and concurrency

`openControlling()` requires a genuine controlling TTY; neither availability of `process.stdout()` nor redirection to a file/pipe implies a terminal. Absent/unsupported terminal fails, with **no ANSI/TextWriter fallback**. The host must still explicitly allow native library aliases under existing D259 `--allow-library alias=target`. Trusted library C code remains `TRUSTED_IN_PROCESS`, not confined by guest Filesystem grants.

Each successful Screen is a fresh identity even if a native address is reused. **Initial B1 permits at most one active Screen in the entire OS host process**; concurrent acquisitions that contend with that Screen fail with `ScreenError(kind: "busy")`. This is not an exclusivity guarantee against external processes. The Screen and its live foreign resources remain Actor-owned, non-transferable and non-proxied; its ordinary copied event/size data can transfer normally. No Actor or Context inherits a Screen automatically. Adding `openOn(device)` or multiple physical screens is deferred.

## Logical geometry, drawing and Unicode

Coordinates are **zero-based `row, column`**: `(0,0)` is top left, `(rows - 1, columns - 1)` is bottom right. `size()` is an observation, not a guarantee that physical size will remain unchanged.

`clear` and `put` modify a **logical cell image**, not immediately the physical terminal. `present` is the explicit publication boundary; the backend may optimize differences and is not required to make one native call per cell. A failed physical presentation may have emitted a prefix; no rollback is promised. It leaves the Screen unusable for further drawing but still allows close/restoration.

`put` validates semantic Integer coordinates within bounds, a genuine String and an allowed foreground before any logical mutation. It writes on exactly **one row**; it does not wrap, scroll, normalize Unicode or interpret ANSI/ESC/newlines/tabs/C0/C1 controls as terminal instructions. Invalid or unsupported text is rejected without modifying logical cells. Right-edge clipping must not render half of a wide grapheme; overwriting a previous double-width continuation cell must clear that earlier grapheme's full footprint.

Core `String.size()` counts Unicode **scalar values**, not grapheme clusters or terminal cells. B1 requires grapheme-aware visual cell occupancy: narrow ASCII one column, unequivocally wide text two columns, combining marks associated with a valid base when supported. Unsupported standalone combining marks, unsupported ZWJ/emoji sequences and ambiguous representations must be rejected rather than silently corrupt cells. The terminal/backend policy for ambiguous width may differ by host but must be internally consistent within a Screen session. No claim is made that all terminal emulators display all emoji identically. Unicode 17.0.0 is the Protos text reference; the public grapheme capability topology is independently unsettled, so PLAT078 must demonstrate a valid route without inventing production Java or bypassing existing contracts.

Foreground is **one optional String**, exactly one of `"default"`, `"black"`, `"red"`, `"green"`, `"yellow"`, `"blue"`, `"magenta"`, `"cyan"`, `"white"`; monochrome implementations may render the default appearance. B1 **does not** access private data of ratified opaque `std:text/style` / `std:text/styled_text` values and does not change D190/D191. No Style/StyledText input, background, bold, underline or truecolor is selected.

## Synchronous input and resize

`nextEvent(timeoutMs)` takes an exact Integer in `0..2147483647` milliseconds. Zero polls; positive numbers request bounded waiting. It never implicitly calls `present`, returns a Future or introduces Actor suspension/reentry. Its result is `null` on timeout/no recognized event, or one fresh frozen ordinary object:

- `{kind: "text", text: String}`: one Unicode scalar.
- `{kind: "key", key: String}`: one of `"up"`, `"down"`, `"left"`, `"right"`, `"enter"`, `"escape"`, `"backspace"`, `"tab"`.
- `{kind: "resize", rows: Integer, columns: Integer}`: one coherent dimension snapshot.

Events follow the backend-recognized order; consecutive resize observations may coalesce. On a recognized resize the logical geometry changes and the previous pending logical image is invalidated and cleared; an application recomposes before presenting. The size may change again after the snapshot. Native input is synchronous, occupying the Actor execution segment under `ACTORS.md` §24J; an asynchronous/cancellable variant is **future work**, not hidden machinery in B1.

## Errors and close lifecycle

`std:terminal/screen` exports an ordinary module-local `ScreenError` prototype immediately delegating to `Error`; safe String slots `operation` and `kind` distinguish screen-domain failures. Kinds are `"noTerminal"`, `"unsupported"`, `"busy"`, `"render"`, `"input"`, `"restore"`. Invalid public argument domains signal ordinary `Error` before modifying logical image. Existing FFM `ForeignError` for denied/unavailable library, native binding failure, etc. remains in place and must not be flattened into `ScreenError`; operations on closed resources use `IOLifecycleError`.

`close()` obeys `IO_CORE.md` `Closable`: it irrevocably cuts over to unusable before returning its **fresh** Future, releases/restores once, resolves to the Screen on success, and is logically idempotent. A failed close records one exact Error object for all subsequent close observations in that isolation domain, with no unsafe retry of native destruction. Closing does not close `process.stdin()` or `process.stdout()`. Ordinary `ensure` is the deterministic cleanup mechanism; GC, fatal process termination and SIGKILL never guarantee restoration.

## Intentional limits and downstream gates

Deferred: public Grid/Frame, subwindows, cursor API, scroll, mouse, rich key combinations, advanced colors/styles, widgets/layout/GUI/WUI, remote terminals, callback event loop, asynchronous input and additional physical sessions. Those should be additive, separately ratified when needed. A future backend can replace ncursesw/termbox2 without changing the B1 Screen surface; **no backend has been finally selected**.

**PLAT078 must demonstrate** real PTY/TTY acquisition, correct Unicode width/clipping, FFM ABI signatures and Native Image envelope, lifecycle and signal restoration, duplicate ownership and multicontext behavior on JVM/Native; absence of that evidence forbids a silent weakening of B1. The strongest objection to B1 is the still-unproven ability to meet grapheme and one-process-screen invariants with the current FFM.

**No product `spec/` change has been published.** This document records exact owner selection only. D260 remains open pending normative specification publication and GITHUB015 coordination, PLAT078 remains open, and LIB032 remains blocked for implementation. No product tests or native runtime gates are claimed.
