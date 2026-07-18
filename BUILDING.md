# Building libmsxca

## Requirements

- **CMake** 3.14 or later
- **C compiler** — gcc, clang, MSVC (on Windows)

No other dependencies. libmspack is bundled (no external libmspack install needed).

## Quick build

```sh
cmake -S . -B build
cmake --build build
```
This produces:
| Platform | Output |
|----------|--------|
| Linux | `build/libmsxca.so` |
| macOS | `build/libmsxca.dylib` |
| Windows (mingw64) | `build/libmsxca.dll` |

### Clang with Windows steps
1. Open the CMake GUI and set paths
2. Press Configure, select Visual Studio 18 2026
3. Set plaform to x64
4. Select "Specify native compilers", click Next
5. On the compilers tab, set "C" and "C++" to `clang-cl`, the "Fortran" one to `flang`, click on finish
6. Once Configure is done, click on Generate, then "Open Project" will launch VS with the generated project
7. In VS, set Solution config to Release and arch to x64.
8. Running Build Solution will create `msxca.dll` in `builddir/Release`. Builddir is your specified path in Step 1.


## Build with tests

```sh
cmake -S . -B build -DBUILD_TEST=ON
cmake --build build
cd build && ctest
```

The test program (`test_msxca`) takes a file path as an argument and attempts
to decompress it as an LZXNATIVE blob:

```sh
build/test_msxca path/to/compressed.blob
```

## Cross-compiling for Windows (from Linux)

```sh
# Requires mingw64 toolchain
cmake -S . -B build-win64 \
    -DCMAKE_SYSTEM_NAME=Windows \
    -DCMAKE_C_COMPILER=x86_64-w64-mingw32-gcc
cmake --build build-win64
```

Produces `build-win64/msxca.dll`.

## Install (optional)

```sh
cmake --install build --prefix /usr/local
```

Installs:
- `lib/libmsxca.so` (or `.dylib` / `.dll`)
- `include/msxca.h`

## Input format notes

Any raw LZXNATIVE file (.xbc/.bin) (magic `0FF512EE`) can be passed directly to
`msxca_decompress`. This includes:

- **`.xbc` files** — produced by the original Snailium ArchiveManager tool.
  These are raw LZXNATIVE files and work directly with no preprocessing.
- **LNK4 container entries** — these embed LZXNATIVE blobs at known offsets
  inside a larger file (the LNK4 file table). Extract the blob from the
  container first, then pass it to `msxca_decompress`.

In both cases the compressed data is identical: 48-byte big-endian header,
then chunk-framed LZX blocks. The only difference is how the blob is
addressed (standalone file vs. offset inside a container).
