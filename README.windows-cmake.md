# Windows standalone CMake build

This repository tracks pkgconf 3.0.7 and keeps the upstream Meson build files.

For a native Windows x64 build that produces a single `pkg-config.exe` with the MSVC runtime linked statically:

```powershell
cmake -S . -B build-msvc -G "Visual Studio 18 2026" -A x64
cmake --build build-msvc --config Release --parallel
```

The executable is written to:

```text
build-msvc\Release\pkg-config.exe
```

The CMake build uses the upstream Windows implementation and manifest, links `libpkgconf` statically, and uses MSVC `/MT`. It should therefore require only Windows system DLLs.

For upstream-supported cross-platform builds, use Meson as documented in `README.md`.
