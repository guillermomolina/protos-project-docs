# PLAT079-C — Protos-native lexer direction selected

**Date:** 2026-11-27
**Status:** OWNER-SELECTED DIRECTION; exact contract and full decision closure OPEN.
**Authority:** Non-normative platform architecture checkpoint.
**Live coordination:** [PLAT079 #1206](https://github.com/guillermomolina/protos/issues/1206).
**Origin:** [INC005 #1031](https://github.com/guillermomolina/protos/issues/1031); first consumer [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12).

## Approval provenance and scope

In the active design interaction, after the comparative proposal explicitly identified PLAT079-C and its limitations, the project owner stated **"APRUEBO PLAT079-C"**. This approves the presented direction, not an unpresented exact public API, implementation, completed GITHUB010 audit, measured viability or a general self-hosting cutover. The owner additionally authorized bounded publication in guillermomolina/protos-project-docs and requested that new formal work be allocated only when needed, using I for implementation.

The approved direction supersedes the earlier unselected architectural direction in PLAT079. It does not close the open investigation.

## Selected architecture

- Implement an authentic reusable lexer for Protos in ordinary Protos, using normative lexical grammar rather than a website-only approximate scanner. The existing Java lexer/parser/canonicalizer and Truffle Bytecode DSL remain the execution and bootstrap path unchanged.
- Separate lexical recognition, original source positions and trivia, lexical values, diagnostics and presentation classification. Use one recognition core; strict rejection and presentation recovery must not become competing grammars. Preserve the source rather than reconstructing it from decoded values.
- The first concrete consumer is WEB016 syntax highlighting. Keep the lexer independent of HTML, CSS, themes, ANSI and the website. Separate classification and renderer; never execute the source being highlighted.
- Require differential conformance with the existing Java lexer in the common input domain and independently check against normative grammar. Unicode 17.0.0 XID and NFC, maximal symbolic runs, exact newline/comment/string rules and six reserved words are not optional.
- No parser, canonicalizer, compiler, privileged Java bridge, special bootstrap, TextMate interpreter, regex expansion or FFI dependency in the first direction. Future parser and self-hosting remain possible, not scheduled or preimplemented.
- Preserve Java/Truffle execution, existing primitive model (long/double/boolean), ordinary message semantics and specialized hot paths. No new Java production infrastructure. If later evidence exposes an unavoidable new Java mechanism, return to the design gate with a verified equivalent in another Truffle language, preferably GraalPy; this approval does not authorize it.
- WEB016 target remains Linux amd64 Native; JVM/Native behavior of the reusable lexer must be compared. ARM64 work is explicitly excluded.

## GITHUB021 direction-level invariant review

| Applicable invariant | Review |
|---|---|
| Normative specification authority; no syntax or observable semantic changes | Preserved; Java is a differential oracle, not the language authority. |
| Ordinary Protos; no new privileged host infrastructure | Preserved by direction; Unicode strategy must demonstrate this. |
| Java/Truffle runtime, primitive and hot-path constraints | Preserved; no execution frontend cutover or hot-path changes. Executable validation is pending. |
| D187 regex restrictions | Preserved; no lookaround/backreference additions or substitution of approximate Unicode properties. |
| D266 Markdown and D267 safe HTML | Preserved; parser supplies code fences as data, HTML serializer escapes text. No new contract selected for those components. |
| WEB016 Linux amd64 scope | Preserved. |
| WEB016 locked TextMate authority | Explicit prospective change limited to Protos highlighting, not already applied to the website. Reconcile website AGENTS.md and exact revision provenance before switching. VS Code grammar and non-Protos fenced languages are unchanged by this selection. |
| Incremental allocation and bootstrap | Preserved; Java remains P0, guest lexer P1; no parser or compiler allocation now. |

No previously approved language invariant is reopened by this direction. The exact contract must repeat this review before final ratification.

## Open contract and implementation gate

Remain OPEN for the complete applicable normative audit, exact corpus/highlighting baseline, Unicode data generation/reuse strategy (no manually divergent tables), source-scalar positions and representation, strict/recovery rules, deterministic limits and failure behavior, API/module boundary, theme parity and safe HTML handling (including NUL and physical line-ending limitations). Measure sequential string traversal and actual cold/warm JVM and installed Native amd64 behavior; do not infer compilation or speed from GraalVM alone. Define differential, source-fidelity, adversarial and deployment acceptance gates.

This direction does not settle exact guest-facing API names or representations, limits, public Standard Library semantics, persistent compiler artifacts, source layout replacement, LSP integration, or a parser migration. If exact public semantics require independent D-family work, route through the design gate rather than treating this PLAT checkpoint as specification authority.

No production implementation or dependent release is authorized by this direction record. The next actionable work remains in PLAT079; allocate one I-family implementation owner when the concrete bounded slice and applicable gates are ready, without preallocating future parser/compiler work.

See [comparative research and approval evidence](../../evidence/PLAT079/PLAT079_C_COMPARATIVE_RESEARCH_AND_OWNER_APPROVAL_2026_11_27.md). No product source/specification/version change or executable test result is claimed.
