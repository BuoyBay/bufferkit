# BufferKit 0.1.0

`Import bufferkit` provides `ByteSpool`, bounded append-only binary storage that
moves from memory to a private temporary stream when its logical byte count
exceeds a configurable threshold. It uses system C/Win32 APIs directly and has no
third-party native dependency. It is synchronous and **single-threaded**: callers
must not use or destroy a spool concurrently.

## API

| Member | Contract |
| --- | --- |
| `New ByteSpool(memoryThreshold = 1048576, maxBytes = 1073741824, maxReadBytes = 16777216)` | Threshold is 0 through min(maxBytes, 64 MiB); total is 0 through 1 TiB; read allocation limit is 1 through 64 MiB |
| `Append(data As Bytes)` | Copies bytes; Nil is rejected; empty data does not spill; append beyond maxBytes rejects before mutation |
| `Read(offset, count) As Bytes` | Independent copy of an exact zero-based range; count must fit maxReadBytes; empty range at Count is valid |
| `ToBytes() As Bytes` | Read the whole spool subject to maxReadBytes; rejects oversized materialization |
| `CopyTo(destination As BinaryStream, chunkBytes = 65536)` | Copy from byte zero to destination's current position, then flush; chunks must be 1 through min(maxReadBytes, 1 MiB) |
| `Close()` | Release memory and temporary file; idempotent, including after a reported close error |
| `Count`, `IsSpilled`, `IsClosed`, `HasFailed` | Read-only committed byte count and lifecycle observations, still available after Close |

At exactly the threshold, contents remain in memory; the first append exceeding
it spills all current contents. A zero threshold forces every nonempty append to
disk. Spilled storage stays on disk. Mutating supplied or returned Bytes never
changes the stored contents. The logical threshold is not an exact process RSS
budget: Bytes backing capacity, the caller's input and allocated results consume
additional memory. Range reads allocate at most maxReadBytes plus 64 KiB scratch;
CopyTo materializes one configured chunk at a time. Appends to disk use the input
pointer directly and do not create a second whole-input copy.

## Errors and ownership

Invalid constructor arguments, Nil inputs/destinations, excessive appends and
invalid/overflowing ranges throw `InvalidArgumentException` without poisoning the
spool. Owned temporary-file open, unlink, fdopen, seek, short read/write, flush
or close failures throw `BufferSpoolException` and set HasFailed. The first
failure consumes the temporary handle and makes further content operations
invalid; call Close to finish the lifecycle. Count then describes only previously
committed appends, not recoverable content. Close marks the spool closed before
closing its handle, so a failed close is never retried on an already-consumed
native handle. The destructor provides best-effort nonthrowing cleanup; explicit
Close is required to observe close errors.

Read scratch-allocation failure throws BufferSpoolException without poisoning
otherwise unchanged storage. Copy destination errors also throw that exception
but leave the source usable. A failed copy can leave a prefix in the caller's
destination. CopyTo checks BinaryStream.LastError after each write and its final
flush, does not close that stream and does not roll back its bytes or position.

## Temporary storage and portability

POSIX uses atomic `open(O_CREAT|O_EXCL|O_RDWR|O_CLOEXEC, 0600)` in
System.Folders.Temporary, immediately unlinks the new name, then transfers the
already-open descriptor to fdopen. Names include process ID, monotonic time and a
bounded retry index; atomic exclusive creation prevents replacement or following
an existing final-component symlink. It never reopens a chosen path. If unlink
fails, it closes the descriptor and reports failure before writing payload; an
empty private file may remain. It never retries pathname deletion after that
failure. The temporary directory and its ancestors must be trusted for the
current user's process; this is not a privileged cross-user sandbox.

Windows uses CreateFileW CREATE_NEW with share mode zero,
FILE_ATTRIBUTE_TEMPORARY and FILE_FLAG_DELETE_ON_CLOSE in the user's temporary
folder. It transfers the owned handle through _open_osfhandle/_fdopen using the
same CRT as its stdio operations. All Windows UCRT bindings resolve actual exports
with Optional Declare: the installed SDK's static aliases otherwise mix legacy
MSVCRT FILE creation with UCRT stdio and cause access violations. It does not use tmpfile's drive-root behavior,
reopen the filename, or expose a path/handle. Permission and disk availability
errors are explicit. No application-wide umask or process setting is changed.

Positions use 64-bit fseek long on the supported POSIX LP64 targets and
_fseeki64 on Windows. All six OS/architecture branches are compiler-checked.
Sealed-consumer execution, descriptor cleanup and native I/O fault tests were
performed on Linux x86_64. Native Windows x64 checks, builds and executions also
passed, including Unicode temporary paths, exclusive sharing, delete-on-close
and 200 destructor handle cleanups; see the Windows validation report.
The macOS arm64 run also passed
native compilation/execution and independently checked exclusive mode-0600,
CLOEXEC temporary-file creation and descriptor cleanup. Intel macOS, Linux arm64
and Windows ARM64 runtime behavior remains unverified.

Maintainers run the private validation suite before release. See the repository README for public build instructions.

This is a byte storage primitive. It supplies no text editing, encoding inference,
compression, asynchronous dispatch, persistence or automatic streaming adapter
for existing whole-buffer HTTP/parser APIs.
