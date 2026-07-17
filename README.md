# Node.js for Android ARM

Cross-compiles [Node.js](https://nodejs.org/) for Android ARM64 (arm64-v8a) and ARM32 (armeabi-v7a, experimental) via GitHub Actions.

Produces per ABI: `libnode.so` (shared library) + `node` CLI binary.

## Usage

Artifacts are available as tarballs from workflow runs or GitHub Releases.

### Build profiles

| Profile | Flags | Use case |
|---------|-------|----------|
| `balanced` | JIT + `-O3` | Best runtime speed (default) |
| `speed` | `+LTO` | Faster, longer build |
| `size` | `+v8-lite-mode` | Lower RAM, slower JS/crypto |

### Custom library name

Set `lib_name` input to `aurora` → outputs `libaurora.so` with `set-soname` and `patchelf` dependency rewrite.

### API level

Minimum Android API level: **24** (Android 7.0). Required by Node.js v24+.

## Architectures

| ABI | Status |
|-----|--------|
| `arm64-v8a` | Verified working |
| `armeabi-v7a` | Experimental — upstream V8 cross-build bug ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975)) |

## Patches applied

This workflow carries community-maintained patches for Android cross-compilation issues tracked upstream:

- [nodejs/node#57748](https://github.com/nodejs/node/pull/57748) — Termux Android build patches
- [nodejs/node#58505](https://github.com/nodejs/node/issues/58505) — Android test failures
- [nodejs/node#58975](https://github.com/nodejs/node/issues/58975) — 32-bit ARM cross-build V8 bugs

See the workflow file for the full list of applied patches.

## License

[MIT](LICENSE)
