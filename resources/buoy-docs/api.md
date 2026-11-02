---
title: "BufferKit API"
summary: "ByteSpool limits, range reads, chunked copies and temporary ownership."
category: "Reference"
order: 20
keywords: ["ByteSpool", "BufferSpoolException", "CopyTo", "HasFailed"]
symbols: ["ByteSpool", "ByteSpool.Append", "ByteSpool.Read", "ByteSpool.ToBytes", "ByteSpool.CopyTo", "ByteSpool.Close", "ByteSpool.Count", "ByteSpool.IsSpilled", "ByteSpool.IsClosed", "ByteSpool.HasFailed", "BufferSpoolException"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

| API | Behavior |
| --- | --- |
| ByteSpool(memoryThreshold = 1048576, maxBytes = 1073741824, maxReadBytes = 16777216) | Memory threshold 0..min(maxBytes,64 MiB); total 0..1 TiB; read cap 1..64 MiB |
| Append(data As Bytes) | Append a copy; reject Nil and excess total bytes before mutation |
| Read(offset, count) As Bytes | Exact independent zero-based range; count must fit remaining bytes and read cap |
| ToBytes() As Bytes | Materialize all contents subject to read cap |
| CopyTo(destination As BinaryStream, chunkBytes = 65536) | Copy whole contents to current destination position and flush, leaving destination open; chunks 1..min(read cap,1 MiB) |
| Close() | Consume temporary ownership and memory, then report any closing error; repeat calls do nothing |
| Count / IsSpilled / IsClosed / HasFailed | Read-only observations remain accessible after failure/Close |

An empty append does not spill. Read(Count,0) is valid. Supplied/returned Bytes
are detached. Thresholds bound logical data, not exact RSS or backing allocation.
Each range read uses its result allocation plus at most 64 KiB scratch; CopyTo
materializes one chunk at a time.

## Empty, full and closed spools

| State | Consumer behavior |
| --- | --- |
| Open and empty | ToBytes and Read(0,0) return empty Bytes. An empty append is accepted without spilling. |
| Open at maxBytes | Existing bytes remain readable and copyable. Empty appends are accepted; a nonempty append throws InvalidArgumentException without changing Count or setting HasFailed. |
| After Read, ToBytes or CopyTo | Contents remain stored and Count is unchanged. These operations do not consume bytes or free capacity for later appends. |
| Closed | Append, Read, ToBytes and CopyTo throw BufferSpoolException, including empty operations. Close remains harmless to repeat; metadata remains readable. |

ByteSpool has no clear, truncate or reopen operation. Create a new spool for the
next independent payload; Close releases storage but does not reset Count.
A zero-capacity spool requires memoryThreshold = 0 and accepts only empty data.

CopyTo validates its chunk size even for an empty spool, and flushes the
destination even when there are no bytes to write. If maxReadBytes is below
65536, pass an explicit chunkBytes no greater than maxReadBytes: the default
65536-byte chunk is rejected rather than reduced automatically.

Invalid arguments throw InvalidArgumentException and do not poison the spool.
Owned I/O failures throw BufferSpoolException, set HasFailed, release the native
handle, and reject later content operations. Count retains the committed byte
count but contents are not recoverable. A read scratch-allocation error and a
destination write/flush failure also throw BufferSpoolException without poisoning
unchanged source storage. Failed destination copies can leave partial output.

POSIX atomically creates an owner-only temporary file, immediately unlinks its
name and transfers the descriptor to stdio. An unlink failure closes the handle
before payload is written; an empty private file may remain. Windows atomically
creates an exclusive delete-on-close file and transfers its handle to the CRT.
Windows UCRT functions resolve actual exports at runtime with Optional Declare,
avoiding SDK import aliases that mix incompatible MSVCRT and UCRT FILE pointers.
Neither backend reopens a chosen filename. Temporary directory ancestors must be
trusted. Destructors provide best-effort cleanup and suppress close errors;
explicit Close is required to observe those errors. The package changes no
process-wide settings.

[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
