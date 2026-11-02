# Bounded append-only byte spool, private temp storage

`Import bufferkit` provides `ByteSpool`, bounded append-only binary storage that
moves from memory to a private temporary stream when its logical byte count
exceeds a configurable threshold. It uses system C/Win32 APIs directly and has no
third-party native dependency. It is synchronous and **single-threaded**: callers
must not use or destroy a spool concurrently.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add bufferkit --git https://github.com/BuoyBay/bufferkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## API at a glance

| Member | Contract |
| --- | --- |
| `New ByteSpool(memoryThreshold = 1048576, maxBytes = 1073741824, maxReadBytes = 16777216)` | Threshold is 0 through min(maxBytes, 64 MiB); total is 0 through 1 TiB; read allocation limit is 1 through 64 MiB |
| `Append(data As Bytes)` | Copies bytes; Nil is rejected; empty data does not spill; append beyond maxBytes rejects before mutation |
| `Read(offset, count) As Bytes` | Independent copy of an exact zero-based range; count must fit maxReadBytes; empty range at Count is valid |
| `ToBytes() As Bytes` | Read the whole spool subject to maxReadBytes; rejects oversized materialization |
| `CopyTo(destination As BinaryStream, chunkBytes = 65536)` | Copy from byte zero to destination's current position, then flush; chunks must be 1 through min(maxReadBytes, 1 MiB) |
| `Close()` | Release memory and temporary file; idempotent, including after a reported close error |

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer bufferkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
