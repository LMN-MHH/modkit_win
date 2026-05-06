# modkit_win

This is a private Windows-oriented fork of Oxford Nanopore Technologies'
`modkit`.

The official project, documentation, releases, and issue tracker remain at:

https://github.com/nanoporetech/modkit

This fork is only intended to record the local changes needed to compile and run
`modkit` on Windows. It should not be treated as an upstream replacement.

## What Changed

Compared with the original `modkit` source, this fork adds Windows build support
around the `rust-htslib`/`hts-sys` dependency chain:

- Pinned the Rust toolchain in `rust-toolchain.toml` to Rust `1.90.0`.
- Added the `x86_64-pc-windows-gnu` Rust target.
- Added `.cargo/config.toml` for the local MSYS2 UCRT64 toolchain.
- Patched `hts-sys` to use the Windows port branch:
  `https://github.com/theAeon/hts-sys`, branch `hts-win`.
- Vendored `rust-htslib` under `vendor/rust-htslib` and patched the workspace
  to use it.
- Disabled default `rust-htslib` features in the workspace crates to avoid the
  `curl`/`openssl-sys` path that caused Windows build problems.
- Enabled `bzip2` and `lzma` support explicitly for `rust-htslib`.
- Added Windows-specific linker arguments for htslib dependencies:
  `ws2_32`, `bcrypt`, `userenv`, and `systre`.
- Adjusted a few tests for Windows path, line-ending, compressed-FASTA, and
  small floating-point-output differences.
- Added a fallback for `validate` input path resolution when
  `canonicalize()` fails on an existing path, which can happen on mapped
  Windows network drives.

## Prerequisites

Install Rust with `rustup`, then install the pinned toolchain and Windows GNU
target:

```powershell
rustup toolchain install 1.90.0
rustup target add x86_64-pc-windows-gnu --toolchain 1.90.0
```

Install MSYS2, then install the UCRT64 build dependencies. In an MSYS2 shell:

```bash
pacman -S --needed \
  mingw-w64-ucrt-x86_64-gcc \
  mingw-w64-ucrt-x86_64-cmake \
  mingw-w64-ucrt-x86_64-pkg-config \
  mingw-w64-ucrt-x86_64-zlib \
  mingw-w64-ucrt-x86_64-bzip2 \
  mingw-w64-ucrt-x86_64-xz \
  mingw-w64-ucrt-x86_64-libsystre
```

This checkout expects the tools to live under:

```text
C:\msys64\ucrt64
```

If your MSYS2 installation is elsewhere, update `.cargo/config.toml`.

## Build

From the repository root in PowerShell:

```powershell
$env:Path = "C:\msys64\usr\bin;C:\msys64\ucrt64\bin;$env:Path"
$env:PKG_CONFIG_PATH = "C:\msys64\ucrt64\lib\pkgconfig"
$env:PKG_CONFIG_ALLOW_CROSS = "1"

cargo build --target x86_64-pc-windows-gnu
```

The debug executable will be:

```text
target\x86_64-pc-windows-gnu\debug\modkit.exe
```

For a release build:

```powershell
cargo build --release --target x86_64-pc-windows-gnu
```

The release executable will be:

```text
target\x86_64-pc-windows-gnu\release\modkit.exe
```

## Test

Run the full test suite with the same environment:

```powershell
$env:Path = "C:\msys64\usr\bin;C:\msys64\ucrt64\bin;$env:Path"
$env:PKG_CONFIG_PATH = "C:\msys64\ucrt64\lib\pkgconfig"
$env:PKG_CONFIG_ALLOW_CROSS = "1"

cargo test --target x86_64-pc-windows-gnu
```

At the time this README was written, the full suite passed on Windows with the
UCRT64 toolchain.

## Example Run

```powershell
$env:Path = "C:\msys64\usr\bin;C:\msys64\ucrt64\bin;$env:Path"

.\target\x86_64-pc-windows-gnu\debug\modkit.exe pileup `
  "path\to\reads.bam" `
  "path\to\output.bed" `
  --log "path\to\modkit.log"
```

For normal `modkit` usage, command behavior, and output formats, refer to the
official documentation:

https://nanoporetech.github.io/modkit/

## License

This fork preserves the upstream license and copyright.

Modkit is distributed under the terms of the Oxford Nanopore Technologies, Ltd.
Public License, v. 1.0. See `LICENCE.txt`.
