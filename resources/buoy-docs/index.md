---
title: "BufferKit"
summary: "Bounded byte spools that spill to private temporary storage."
category: "Overview"
order: 0
keywords: ["bytes", "spool", "buffer", "temporary", "spill"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# BufferKit

ByteSpool owns append-only byte contents. It keeps logical contents in memory
until an append exceeds the configured threshold, then uses an owned temporary
stream. Append and Read copy bytes; callers cannot alias internal storage.

The spool is synchronous and single-threaded. Storage has a maximum total count;
range reads and ToBytes have a separate materialization cap. CopyTo reads one
bounded chunk at a time into the caller's BinaryStream, checks write/flush errors,
and leaves that destination open at its resulting position.

Owned I/O failures make the spool terminally failed. Call Close explicitly to
observe closing errors; the destructor performs nonthrowing best-effort cleanup.
Invalid arguments leave an open spool usable. This is not a text editor, general
stream interface, compression engine or persistent file format.

See [Getting started](getting-started.md) and [API reference](api.md).
