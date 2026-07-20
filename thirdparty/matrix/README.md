# Bundled copy of vani-matrix

This directory is a bundled copy of [vani-matrix](https://github.com/enthusiasticgeek/vani-matrix)
v0.1.0 (`src/lib.vani` + `vani.toml`, unmodified).

## Why a bundled copy instead of a normal dependency?

The vāṇी compiler's `vanic publish` does not yet resolve transitive
dependencies: a published package's tarball builder
(`copy_dir_vani` in the compiler) explicitly skips any directory literally
named `vendor` or `target`, and `vanic add`/`vanic build` never re-fetches
a dependency's own dependencies for the consumer. A `path = "../vani-matrix"`
dependency works for local development (this repo and vani-matrix are
sibling directories on the author's machine) but would resolve to a
nonexistent path for anyone who installs `probability` via `vanic add`.

Bundling the source directly into `thirdparty/matrix/` (a name the tarball
builder does *not* skip) makes the published `probability` package
self-contained and installable by anyone, at the cost of duplicating
vani-matrix's source until the compiler supports real transitive
dependency resolution.

## Keeping this in sync

If vani-matrix gains new functions or fixes that vani-probability's v0.4.0+
code depends on, refresh this copy from the canonical repo:

```sh
# Run from vani-probability's repo root, with vani-matrix cloned as a
# sibling directory (../vani-matrix).
cp ../vani-matrix/vani.toml thirdparty/matrix/vani.toml
cp ../vani-matrix/src/lib.vani thirdparty/matrix/src/lib.vani
```
