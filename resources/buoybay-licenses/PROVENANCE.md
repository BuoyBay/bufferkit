# BufferKit source study

Inspired by Andrew Keller's REALbasic-Common-KFS-BSD at commit
`9d0b41fad7f75ab5c91098feed20a9fdb8a5745a`, particularly
[BigStringKFS.rbbas](https://github.com/andrewkeller/REALbasic-Common-KFS-BSD/blob/9d0b41fad7f75ab5c91098feed20a9fdb8a5745a/Common/Data%20Structures/BigStringKFS.rbbas)
and its temporary-file support. The source combines strings, memory blocks,
streams and files and selects disk storage at some construction/consolidation
boundaries. Its complete library is licensed under BSD three-clause terms with
per-file contributor notices.

BufferKit is independently written Buoy code; no upstream implementation or
tests were transcribed. It provides a smaller binary append/read model with
explicit byte limits, bounded materialization, private temporary ownership and
terminal I/O failure. No claim of Xojo API or BigStringKFS behavioral parity is
made. In particular it does not reproduce overloaded string operations, mutable
external storage references or deferred deletion pools.

System binding contracts were checked against primary documentation:

- [POSIX open](https://pubs.opengroup.org/onlinepubs/9799919799/functions/open.html)
- [POSIX unlink](https://pubs.opengroup.org/onlinepubs/9799919799/functions/unlink.html)
- [CreateFileW](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilew)
- [_open_osfhandle ownership transfer](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/open-osfhandle)
- [_fdopen stream ownership](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/fdopen-wfdopen)

The POSIX implementation uses exclusive creation and immediate unlink instead
of tmpfile because the installed Buoy SDK stubs omit tmpfile. POSIX fseek's long
is 64-bit on Buoy's supported LP64 targets; Windows explicitly uses _fseeki64.
No SDK/runtime/compiler files were changed to add missing stub symbols.

Native Windows validation additionally found SDK import-library aliases that
selected legacy MSVCRT _fdopen/_open_osfhandle alongside UCRT stdio. Windows UCRT
bindings now use Optional Declare to resolve all owned stream operations from
the actual same UCRT module. This independently implemented compatibility fix was
verified by native compiler builds/executions and PE import inspection; it is not
derived from the upstream REALbasic implementation.
