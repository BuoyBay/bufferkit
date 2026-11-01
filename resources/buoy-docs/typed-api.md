---
title: "bufferkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["BufferSpoolException", "ByteSpool"]
dependencies: []
symbols: ["BufferSpoolException", "ByteSpool"]
related: ["api.md", "getting-started.md", "index.md"]
---

# bufferkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## BufferSpoolException

`Public Class BufferSpoolException Inherits Exception`

An owned spool failed, was closed, or its copy destination rejected I/O.

### Constructor

`Public Sub Constructor(message As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `String` | `ByVal` | Required |

Returns: no value.

## ByteSpool

`Public Class ByteSpool`

Append-only bounded byte storage which spills to a private temporary stream.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Count` | `Integer` | Yes | No | — | — |
| `IsSpilled` | `Boolean` | Yes | No | — | — |
| `IsClosed` | `Boolean` | Yes | No | — | — |
| `HasFailed` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(memoryThreshold As Integer = 1048576, maxBytes As Integer = 1073741824, maxReadBytes As Integer = 16777216)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `memoryThreshold` | `Integer` | `ByVal` | `1048576` |
| `maxBytes` | `Integer` | `ByVal` | `1073741824` |
| `maxReadBytes` | `Integer` | `ByVal` | `16777216` |

Returns: no value.

### Append

`Public Sub Append(data As Bytes)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `data` | `Bytes` | `ByVal` | Required |

Returns: no value.

### Read

`Public Function Read(offset As Integer, count As Integer) As Bytes`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `offset` | `Integer` | `ByVal` | Required |
| `count` | `Integer` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Bytes` |

### ToBytes

`Public Function ToBytes() As Bytes`

No parameters.

#### Returns

| Type |
| --- |
| `Bytes` |

### CopyTo

`Public Sub CopyTo(destination As BinaryStream, chunkBytes As Integer = 65536)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `destination` | `BinaryStream` | `ByVal` | Required |
| `chunkBytes` | `Integer` | `ByVal` | `65536` |

Returns: no value.

### Close

`Public Sub Close()`

No parameters.

Returns: no value.

### Destructor

`Public Sub Destructor()`

No parameters.

Returns: no value.
