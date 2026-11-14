# opencode-termux

Builds the [opencode](https://github.com/anomalyco/opencode) CLI for **Android aarch64 (Termux)**.

Bun has no Android compile target (the official `bun-linux-aarch64-android` build uses NDK r27c and
calls `close_range(2)`, which is missing on pre-5.9 kernels such as Android 11), so the CLI is compiled
as `linux-arm64-musl` and shipped with a bundled musl runtime. A wrapper script starts
`ld-musl-aarch64.so.1` with `--library-path`, which works on any Android kernel/API level — no root,
no proot, no glibc shim.

Termux login shells export `LD_PRELOAD=/data/data/com.termux/files/usr/lib/libtermux-exec-ld-preload.so`.
That library is bionic-only and cannot be loaded by the musl loader, so the wrapper unsets
`LD_PRELOAD`/`LD_LIBRARY_PATH` before exec'ing the musl runtime. Child processes spawned by opencode
therefore do not inherit termux-exec's path rewriting; tools are spawned via `SHELL`/`PATH`
(`/data/data/com.termux/files/usr/bin/...`), which works as usual.

## How it works

The `android` GitHub Actions workflow:

1. Checks out `anomalyco/opencode` at the pinned commit in `.github/workflows/android.yml` (`UPSTREAM_REF`) with `fetch-depth: 1`.
2. Installs with Bun 1.4.4 (dependency cache keyed on `bun.lock` + `patches/**`, plus a cache for the Bun compile runtime).
3. Compiles with `OPENCODE_TARGET=linux-arm64-musl`, `BUN_COMPILE_RELEASE=bun-v1.4.4`, `OPENCODE_CHANNEL=dev`.
4. Bundles Alpine's musl/libstdc++/libgcc aarch64 runtime as `lib/` and adds a portable wrapper `opencode`.
5. Uploads the result as the `opencode-termux-arm64` workflow artifact.

Trigger it manually with `workflow_dispatch`, optionally overriding `upstream_ref` to build another
branch, tag or commit.

## Install in Termux

```sh
# download the artifact from the latest successful run (Actions tab, or with gh)
gh run download -R berkchy/opencode-termux -n opencode-termux-arm64
tar -xzf opencode-termux-arm64.tar.gz -C "$PREFIX/opt"
ln -sf "$PREFIX/opt/opencode-termux/opencode" "$PREFIX/bin/opencode"
opencode --version
```

Layout after extraction:

```
opencode-termux/
  opencode         # wrapper (sh script, resolves symlinks)
  opencode.bin     # the actual musl static-ish binary
  lib/             # ld-musl-aarch64.so.1, libc.musl, libstdc++.so.6, libgcc_s.so.1
  package.json     # upstream artifact metadata
  UPSTREAM_COMMIT  # pinned upstream commit the binary was built from
```

## Bumping the pinned opencode commit

Edit `UPSTREAM_REF` in `.github/workflows/android.yml`.
