# Protos Project Documentation

This repository contains the durable, non-normative project documentation for
[Protos](https://github.com/guillermomolina/protos).

The canonical project-document corpus is preserved under
[`docs/project/`](docs/project/README.md), including durable work records,
decisions, architecture records, governance rationale, registries, evidence,
and historical snapshots.

## Authority boundary

This repository is a project-record store. It is **not** the operational
governance or control plane for Protos.

The authoritative operational surfaces remain in
[`guillermomolina/protos`](https://github.com/guillermomolina/protos):

- GitHub Issues own live work state;
- the Protos Development Project owns scheduling and prioritization;
- formal work identifiers are governed from `protos`;
- project-owner approvals and design decisions are coordinated there;
- `protos/spec/**` remains the normative language and Standard Library
  authority;
- product implementation remains in the repository that owns that product.

Records in this repository may document those decisions and their evidence, but
they do not independently create or change project authority.

## Repository structure

The durable corpus currently uses the role-first structure:

```text
docs/project/
  work/
  decisions/
  architecture/
  governance/
  registries/
  evidence/
  history/
```

See [`docs/project/README.md`](docs/project/README.md) for the detailed
information architecture.

## License

This repository is licensed under the Adaptive Public License 1.0. See
[`LICENSE.TXT`](LICENSE.TXT).
