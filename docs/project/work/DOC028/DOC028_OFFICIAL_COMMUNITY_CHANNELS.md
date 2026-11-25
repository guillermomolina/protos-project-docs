# DOC028 — Official community channels

## Published product evidence

- Owning issue: [DOC028 / #1170](https://github.com/guillermomolina/protos/issues/1170).
- Exact Protos revision: [510a2c71e9c62cc3c147dea447c70d26e18d9d36](https://github.com/guillermomolina/protos/commit/510a2c71e9c62cc3c147dea447c70d26e18d9d36).
- Commit subject: `DOC028: Integrate official Reddit and X channels into README and announcement workflow`.
- Verified through the published main commit and its file delta.

## Delivered scope

- [README.md](https://github.com/guillermomolina/protos/blob/510a2c71e9c62cc3c147dea447c70d26e18d9d36/README.md): linked Reddit and X badges and Community entries.
- [AGENTS.work/RELEASE.md](https://github.com/guillermomolina/protos/blob/510a2c71e9c62cc3c147dea447c70d26e18d9d36/AGENTS.work/RELEASE.md): mandatory English announcement preparation for GitHub Discussions, Reddit and X after authorized public release publication, canonical release links, prerelease/evidence qualifications, human publication and actual URL evidence.
- [AGENTS.work/DOCUMENTATION.md](https://github.com/guillermomolina/protos/blob/510a2c71e9c62cc3c147dea447c70d26e18d9d36/AGENTS.work/DOCUMENTATION.md): corresponding project-news procedure, flair guidance, concise X posts and no duplicate news archives.

Official identities:

- Reddit: https://www.reddit.com/r/Protos/
- X: https://x.com/ProtosLang
- Website: https://protos.guillermomolina.com/

GitHub Releases remains the release authority. [GitHub Discussions — Announcements](https://github.com/guillermomolina/protos/discussions/categories/announcements) is the canonical news authority under [WEB011](https://github.com/guillermomolina/protos-website/issues/7). The website consumes these sources dynamically. The initial DOC028 reference to the former docs/news archive and WEB004 is superseded by WEB011; the retired archive and materialization machinery are not restored.

## Validation and boundaries

- Product revision and exact file delta: verified remotely.
- Product git diff --check: PASS, reported by the maintainer.
- Product local tests: PASS, reported by the maintainer; command, counts and logs were not supplied.
- No Java, Protos runtime, specification, Maven version or dependency changes in the commit.
- The same commit also adds python3 to .devcontainer/Dockerfile. This is an incidental environment change, not a community-channel feature.
- Badge markup and destinations verified in the published diff; independent live badge rendering and social post publication were not verified in this evidence step.
- No new release, social automation or website deployment was performed by this coordination step.

## Companion implementation handoff

The website and companion README edits were not part of the published Protos commit. Remote inspection found neither official link in the website Community page or Starlight configuration, or in the README files of protos-website, protos-benchmarks, protos-vscode-extension, protos-realworld-example-app and protos-devcontainer.

These independently publishable deliverables are transferred to [I175 / #1171](https://github.com/guillermomolina/protos/issues/1171). DOC028 evidence covers the main Protos repository and announcement procedure only; it is not evidence of website deployment or companion repository completion.

This record preserves the publication evidence and scope boundary, not live issue or scheduling state.
