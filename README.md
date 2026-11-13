# opencode-termux

Builds the [opencode](https://github.com/anomalyco/opencode) CLI for **Android aarch64 (Termux)**.

Bun has no Android target, so the binary is compiled as `linux-arm64-musl`: a fully static musl
executable that runs directly on the Android kernel (no proot, no glibc shim).

## How it works

The `android` GitHub Actions workflow:

1. Checks out `anomalyco/opencode` at the pinned commit in `.github/workflows/android.yml` (`UPSTREAM_REF`).
2. Installs with Bun 1.4.4 (dependency cache keyed on `bun.lock` + `patches/**`).
3. Compiles with `OPENCODE_TARGET=linux-arm64-musl` and `BUN_COMPILE_RELEASE=bun-v1.4.4`.
4. Uploads the result as the `opencode-termux-arm64` workflow artifact (tarball with `bin/opencode`).

Trigger it manually with `workflow_dispatch`, optionally overriding `upstream_ref` to build another
branch, tag or commit.

## Install in Termux

```sh
# download the artifact from the latest successful run (Actions tab, or with gh)
gh run download -R berkchy/opencode-termux -n opencode-termux-arm64
tar -xzf opencode-termux-arm64.tar.gz
chmod +x bin/opencode
mv bin/opencode "$PREFIX/bin/opencode"
opencode --version
```

## Bumping the pinned opencode commit

Edit `UPSTREAM_REF` in `.github/workflows/android.yml`.
