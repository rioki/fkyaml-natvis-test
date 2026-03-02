# fkYAML vcpkg natvis Reproduction

Minimal reproducible example for a configuration failure when consuming
`fkYAML` via vcpkg manifest mode.

## Problem

When linking against `fkYAML::fkYAML`, CMake fails during configure with:

```
Cannot find source file:
.../vcpkg_installed/<triplet>/fkYAML.natvis
```

The imported target `fkYAML::fkYAML` references a `.natvis` file via
`INTERFACE_SOURCES`, but the referenced file does not exist in the
installed vcpkg package.

This breaks configuration even when using non-Visual Studio generators
such as Ninja.

## Environment

- Windows 10 / 11 x64
- CMake ≥ 3.21
- vcpkg (62159a45e1)
- Generator: Ninja
- Triplet: `x64-windows`

## Steps to Reproduce

1. Checkout rioki/fkyaml-natviz-test or build a CMake that lookf fkYAML.

```
find_package(fkYAML CONFIG REQUIRED)
```

2. Configure using the vcpkg toolchain (manifest mode is automatic):

```
cmake -S . -B build -G Ninja -DCMAKE_TOOLCHAIN_FILE=C:\path\to\vcpkg\scripts\buildsystems\vcpkg.cmake
```

3. Observe configuration failure during `find_package(fkYAML CONFIG REQUIRED)`.

## Expected Behavior

- Configuration should succeed.
- Either:
  - The referenced `.natvis` file should exist in the installed package, or
  - The `.natvis` file should not be added to `INTERFACE_SOURCES`, or
  - It should be conditionally added only for MSVC / Visual Studio generators.

## Actual Behavior

CMake fails with:

```
Cannot find source file:
.../vcpkg_installed/x64-windows/fkYAML.natvis
```

The failure occurs during configure, before any build step.

## Workaround

Add this snipet of code after the `find_package(fkYAML CONFIG REQUIRED)`

```
get_target_property(_src fkYAML::fkYAML INTERFACE_SOURCES)
if(_src)
  list(FILTER _src EXCLUDE REGEX ".*\\.natvis$")
  set_property(TARGET fkYAML::fkYAML PROPERTY INTERFACE_SOURCES "${_src}")
endif()
```

This will strip the offening natvis file.

## Notes

This repository exists solely to provide a minimal, self-contained
reproduction case.

