# AUD009-E2 — Standard Library text codecs and data formats complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#647`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`1b4065f0f79a5c2837b4462f00b3b8e481d11b98`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#647`, issue comment
`5739839412`.

Owner approval provenance: `guillermomolina/protos#647`, issue comment
`5739854296`, 2026-09-19.

Derived implementation route:

- `LIB019 / guillermomolina/protos#648` — remove redundant `std:text/*`
  codec facade modules while preserving Core Encoding/TextReader/TextWriter.

Native parent linkage is verified: GitHub reports LIB019 / #648 as a native
sub-issue of AUD009-E2 / #647.

## Purpose

AUD009-E2 reviewed:

```text
std:text/UTF8
std:text/UTF16LE
std:text/UTF16BE
std:text/Latin1

std:json/JSON
std:csv/CSV
std:toml/TOML
std:uri
```

The audit distinguishes redundant convenience aliases from format-specific
capability with independent semantics, scalability value and reintroduction cost.

AUD009-E2 itself authorizes no normative or implementation change.

## Final classification

```text
STDLIB_TEXT_UTF8=REMOVE_NOW_RECONSIDER_LATER
STDLIB_TEXT_UTF16LE=REMOVE_NOW_RECONSIDER_LATER
STDLIB_TEXT_UTF16BE=REMOVE_NOW_RECONSIDER_LATER
STDLIB_TEXT_LATIN1=REMOVE_NOW_RECONSIDER_LATER

CORE_ENCODING_FAMILIES=KEEP
CORE_TEXT_READER_WRITER=KEEP

STDLIB_JSON=KEEP
JSON_TREE_MODEL=KEEP
JSON_PARSE_ENCODE=KEEP
JSON_EVENT_STREAMING=KEEP
JSON_TEXT_IO_ADAPTERS=KEEP

STDLIB_CSV=KEEP
CSV_PARSE_ENCODE=KEEP
CSV_ROW_STREAMING=KEEP
CSV_TEXT_IO_ADAPTERS=KEEP

STDLIB_TOML=KEEP
TOML_SEMANTIC_MODEL=KEEP
TOML_PARSE_ENCODE=KEEP
D087_PRIVATE_BOOTSTRAP_BOUNDARY=KEEP

STDLIB_URI=KEEP
URI_PARSE_FORMAT_RESOLVE=KEEP

GENERIC_SERIALIZER_HIERARCHY=ABSENT_RETAIN_ABSENCE
UNIVERSAL_DATA_NODE=ABSENT_RETAIN_ABSENCE
FORMAT_OBJECT_BINDING=ABSENT_RETAIN_ABSENCE
AMBIENT_CODEC_REGISTRY=ABSENT_RETAIN_ABSENCE
```

No JSON/CSV/TOML/URI mechanism is classified
`REMOVE_NOW_RECONSIDER_LATER` or `REMOVE_PERMANENTLY`.

## std:text facade removal

The four `std:text/*` modules contain no independent codec algorithm, authority
or representation. Their operations are one-to-one aliases over retained Core
capabilities.

For each codec:

```text
module.encode(text)         -> Encoding.<codec>.encode(text)
module.decode(bytes)        -> Encoding.<codec>.decode(bytes)
module.reader(source)       -> TextReader(source, Encoding.<codec>)
module.owningReader(source) -> TextReader.owning(source, Encoding.<codec>)
module.writer(target)       -> TextWriter(target, Encoding.<codec>)
module.owningWriter(target) -> TextWriter.owning(target, Encoding.<codec>)
```

No production consumer was found at the evidence baseline, and JSON/CSV/TOML/URI
do not depend on these facade modules.

The approved removal does **not** remove:

```text
Encoding.UTF8
Encoding.UTF16LE
Encoding.UTF16BE
Encoding.Latin1
TextReader
TextWriter
owning/non-owning I/O construction
```

### Reconsideration trigger

Repeated real application/library use showing that direct explicit
`Encoding.*` plus `TextReader/TextWriter` composition creates material
readability or ergonomic friction.

### Reconsideration scope

A future text-codec convenience API may be designed from then-current evidence.
The removed four-module/six-operation facade is not preselected.

## JSON remains

The complete current JSON surface remains:

```text
nullValue
boolean
string
number
array
object
parse
encode
eventParser
eventWriter
readEvents
writeEvents
```

Its explicit format-specific data model preserves exact decimal numbers,
duplicate-name rejection, Unicode/surrogate rules, deterministic object order,
cycle rejection and strict encode/decode semantics without runtime-specific
JSON values or reflection.

Streaming/event support remains because it removes compulsory whole-tree
materialization and provides a bounded incremental path suitable for future
network/HTTP data flows.

Classification: **KEEP**.

## CSV remains

The complete current CSV surface remains:

```text
parse
encode
rowParser
readRows
writeRows
```

The module stays deliberately small and format-specific, with strict String-row
semantics and scalable row streaming.

No global dialect registry, schema/type conversion, autodetection, ambient
locale, filesystem/network authority or encoding detection is introduced.

Classification: **KEEP**.

## TOML remains

The public TOML 1.1 semantic module remains:

```text
string
integer
float
boolean
offsetDateTime
localDateTime
localDate
localTime
array
table
parse
encode
```

Package Tool and Test Tool demonstrate real TOML need, while D087 intentionally
keeps their private TOML 1.0 bootstrap parser separate from public Standard
Library resolution.

The public module therefore is not redundant with the private bootstrap parser.

AUD005 / #451 continues to own post-closure TOML correctness/hardening work and
is not duplicated or pre-empted by E2.

Classification: **KEEP**.

## URI remains

The complete current URI surface remains:

```text
parse
format
resolve
```

It is strict RFC 3986 generic syntax over ordinary seven-slot data, without
WHATWG repair, IDNA, DNS, Network/filesystem authority, default ports or scheme
registries.

It is a small mature substrate for later networking/HTTP composition.

Classification: **KEEP**.

## Deliberate absences remain absent

```text
generic Serializer/Deserializer hierarchy      ABSENT / RETAIN ABSENCE
universal structured-data Node                 ABSENT / RETAIN ABSENCE
reflection/object binding                      ABSENT / RETAIN ABSENCE
ambient codec/charset registry                 ABSENT / RETAIN ABSENCE
encoding autodetection                         ABSENT / RETAIN ABSENCE

JSON generic object mapper                     ABSENT / RETAIN ABSENCE
JSON implicit JCS/canonical mode               ABSENT / RETAIN ABSENCE

CSV global dialect registry                    ABSENT / RETAIN ABSENCE
CSV schema/type conversion                     ABSENT / RETAIN ABSENCE
CSV sniffing/permissive repair                  ABSENT / RETAIN ABSENCE

TOML source-preserving Document                ABSENT / RETAIN ABSENCE
TOML public streaming/events                   ABSENT / RETAIN ABSENCE
TOML generic datetime coupling                 ABSENT / RETAIN ABSENCE

URI WHATWG repair                              ABSENT / RETAIN ABSENCE
URI scheme registry                            ABSENT / RETAIN ABSENCE
URI implicit normalization                     ABSENT / RETAIN ABSENCE
URI DNS/network effects                        ABSENT / RETAIN ABSENCE
```

## Strongest attempted removals

```text
remove std:text facade modules
    survives -> REMOVE_NOW_RECONSIDER_LATER

keep only std:text encode/decode
    rejected -> leaves a second partial facade with the same duplication problem

remove JSON due no current production import
    rejected -> independent mature capability, high reintroduction cost and
                direct future network/HTTP relevance

remove JSON streaming
    rejected -> loses bounded incremental processing

remove CSV due no current production import
    rejected -> small five-operation library with scalable streaming

remove CSV streaming
    rejected -> makes whole-table retention compulsory

remove public TOML in favor of D087 private parser
    rejected -> different dialect, ownership and bootstrap purpose

remove URI until HTTP exists
    rejected -> small generic substrate and near-term network composition value

generic Serializer/Node hierarchy
    rejected -> larger institution than independent format modules
```

## Required routing

```text
FEATURE=std:text codec facade modules
OUTCOME=REMOVE_NOW_RECONSIDER_LATER
IMPLEMENTATION_REMOVAL_OWNER=LIB019 / guillermomolina/protos#648
NATIVE_PARENT_EXPECTED=#647
NATIVE_PARENT_STATUS=VERIFIED

JSON=KEEP
CSV=KEEP
TOML=KEEP
URI=KEEP
```

## Closure state

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=1b4065f0f79a5c2837b4462f00b3b8e481d11b98
LIB019_IDENTIFIER_ALLOCATION=CONFIRMED_AS_#648
LIB019_NATIVE_PARENT=#647 PASS
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_E2_CLASSIFICATION=COMPLETE
AUD009_E2_COORDINATION_CLOSURE=PASS
```

AUD009-E2 is complete.
