# AUD009-E4 — Standard Library I/O and numeric networking complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#650`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`1b4065f0f79a5c2837b4462f00b3b8e481d11b98`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Initial checkpoint proposal: `guillermomolina/protos#650`, issue comment
`5739895992`.

Corrected ProcessStreams proposal: `guillermomolina/protos#650`, issue comment
`5739924042`.

Owner approval provenance: `guillermomolina/protos#650`, issue comment
`5739931586`, 2026-09-19.

Derived implementation routes: **I066 via ratified D172 / #644**

## Final classification

```text
STDLIB_IO_FILES=KEEP
FILES_READ_ALL_BYTES=KEEP
FILES_WRITE_ALL_BYTES=KEEP
FILES_READ_ALL_TEXT=KEEP
FILES_WRITE_ALL_TEXT=KEEP

STDLIB_IO_PROCESS_STREAMS=KEEP
PROCESS_STREAMS_STDIN_READER=KEEP
PROCESS_STREAMS_STDOUT_WRITER=KEEP
PROCESS_STREAMS_STDERR_WRITER=KEEP

STDLIB_NETWORK_IP_ADDRESSES=KEEP
IP_ADDRESSES_V4=KEEP
IP_ADDRESSES_V6=KEEP
IP_ADDRESSES_PARSE=KEEP
IP_ADDRESSES_FORMAT=KEEP

STDLIB_NETWORK_IP_ENDPOINTS=KEEP
IP_ENDPOINTS_PARSE=KEEP
IP_ENDPOINTS_FORMAT=KEEP
```

No E4 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## std:io/Files remains

`std:io/Files` is not a thin alias layer.

Its whole-file operations encapsulate:

- explicit Filesystem authority;
- File acquisition;
- Future/cancellation handling;
- late successful-acquisition observation;
- close under normal completion, Error and cancellation;
- bounded read/write sequencing;
- private Bytes snapshot before write effects;
- explicit Encoding;
- text encoding failure before write/truncate effects.

Current Package Tool users include ProjectMetadata and ContentIdentity through
`readAllBytes`.

Classification: **KEEP**.

## std:io/ProcessStreams remains

The current executable bodies are compact direct compositions:

```text
TextReader(process.stdin(), process.stdinEncoding())
TextWriter(process.stdout(), process.stdoutEncoding())
TextWriter(process.stderr(), process.stderrEncoding())
```

However LIB004 shows that this module identity was deliberately selected as the
initial Process-I/O Standard Library seam rather than arising incidentally.

The design called Process text adapters the strongest early candidate and placed
them first in the intended convenience-library sequence.

The domain can therefore grow with real Process stream ergonomics while keeping:

- Process explicit;
- Process-selected Encoding explicit;
- borrowing semantics explicit;
- Core free of optional convenience growth.

This KEEP result does not authorize shell, exec, subprocess control,
ProcessBuilder, ResourceScope, ambient current Process or hidden authority.
LIB004 explicitly keeps those concerns separate.

Classification: **KEEP**.

## Numeric networking remains

`std:network/IpAddresses` and `std:network/IpEndpoints` retain strict,
authority-free numeric construction, parse and canonical format behavior.

AUD009-D3 already established that the retained networking foundation should not
be dismantled merely because its first larger client/server consumers have not
yet landed.

E4 preserves the Standard Library representation layer while leaving:

- D172 / #644 authoritative for long-term IpAddress/IpEndpoint ownership and
  Core-vs-stdlib placement;
- D173 / #645 authoritative for Network/TCP provisioning and higher-level
  Standard Library integration.

Classification: **KEEP**.

## D172 ratification reconciliation

D172 / #644 subsequently selected **Candidate C**.

E4's KEEP classification for the two numeric networking modules remains
unchanged and becomes the public ownership boundary for the canonical numeric
families:

```text
std:network/IpAddresses               KEEP
    IpAddress                         ADD / canonical family exposure
    v4/v6/parse/format                KEEP

std:network/IpEndpoints               KEEP
    IpEndpoint                        ADD / canonical family exposure
    parse/format                      KEEP

Prelude IpAddress / IpEndpoint        REMOVE
private canonical runtime substrate   KEEP
```

The module instances themselves do not become Actor-local canonical family
identities. They expose the canonical frozen standard family objects backed by
the smallest general runtime/Standard-Library support seam.

Implementation migration is owned by I066.

## Deliberate absences remain absent

```text
ambient Filesystem/current directory         ABSENT / RETAIN ABSENCE
ambient current Process                      ABSENT / RETAIN ABSENCE
default Encoding for Files text helpers      ABSENT / RETAIN ABSENCE
public generic withOpen / Resource.use       ABSENT / RETAIN ABSENCE
filesystem copy/publish/atomic-write policy  ABSENT / RETAIN ABSENCE

DNS / Resolver                               ABSENT / RETAIN ABSENCE
hostname endpoint parsing                    ABSENT / RETAIN ABSENCE
Network authority in parse/format            ABSENT / RETAIN ABSENCE
TLS / HTTP / WebSocket                       ABSENT / RETAIN ABSENCE
UDP                                          ABSENT / RETAIN ABSENCE
generic socket options                       ABSENT / RETAIN ABSENCE
network deadlines/timeouts                   ABSENT / RETAIN ABSENCE
```

## Audit lesson carried forward

The owner explicitly identified a cross-slice audit hazard:

> a module must not be classified for removal merely because its current body is
> thin if the module identity was intentionally selected as the growth boundary
> for a coherent future library domain.

E4 applied that correction to `std:io/ProcessStreams`.

That lesson also raises a legitimate retrospective question about the earlier E2
classification of `std:text/*`; that matter is routed back to E2 separately and
does not alter the E4 classification.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=1b4065f0f79a5c2837b4462f00b3b8e481d11b98

STDLIB_IO_FILES=KEEP
STDLIB_IO_PROCESS_STREAMS=KEEP
STDLIB_NETWORK_IP_ADDRESSES=KEEP
STDLIB_NETWORK_IP_ENDPOINTS=KEEP

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_E4_CLASSIFICATION=COMPLETE
```

AUD009-E4 is complete.
