---
title: "BufferKit getting started"
summary: "Append binary data and read bounded ranges with explicit cleanup."
category: "Guides"
order: 10
keywords: ["ByteSpool", "Append", "Read", "Close"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Place bufferkit-0.1.0.buipkg in the consumer's packages directory and import
bufferkit. Create a ByteSpool with explicit memory, total and read limits.

Append accepts Bytes and copies their contents. At exactly the memory threshold,
contents stay in memory; exceeding the threshold spills. Read(offset, count)
returns a copied zero-based range; ToBytes materializes everything only if it
fits the configured read limit. Use CopyTo for larger contents. When maxReadBytes
is below 65536, pass CopyTo an explicit chunkBytes at or below that read limit;
the default chunk size is 65536 and is validated even for an empty spool.

Reads and copies do not consume stored bytes or make room for more appends.
Reaching maxBytes leaves the existing contents usable, but further nonempty
appends are rejected. For a new independent payload, create a new ByteSpool:
there is no reset operation, and Close permanently ends content access.

Retain each spool on one thread. Use Try/Finally with an explicit Close call so
errors during content generation still release temporary storage. Close can
report a native closing error but always consumes ownership, and repeated Close
is harmless. Never infer successful export from cleanup alone.

For a complete checked console program, see examples/andrewkeller/buffer/main.bui
in the Buoy project. Read [API reference](api.md) for exact limits and failure
states.
