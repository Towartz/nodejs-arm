<div align="center">
  <h1>Node.js for Android ARM</h1>
  <p>Cross-compiled <code>libnode.so</code> + <code>node</code> CLI for Android ARM64/ARM32</p>
</div>

## Builds

| Profile | Flags | Use Case | Status |
|---------|-------|----------|--------|
| `balanced` | JIT + `-O3` | Best runtime speed (default) | ✅ |
| `speed` | `+LTO` | Faster execution, longer build | ✅ |
| `size` | `+v8-lite-mode` | Lower RAM, slower JS/crypto | ✅ |

| ABI | Status |
|-----|--------|
| `arm64-v8a` | ✅ Verified |
| `armeabi-v7a` | ⚠️ Experimental ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975)) |

## Usage

Trigger a build via **Actions** → **Node.js for Android (ARM)** → **Run workflow**.

Artifacts are downloaded as `lib<name>-android-<abi>.tar.gz` containing:
- `lib<name>.so` — shared library (stripped, `set-soname` applied)
- `node` — CLI binary (stripped)

### Custom library name

Set `lib_name` to `aurora` → produces `libaurora.so` with automatic `patchelf` dependency rewrite on the CLI binary.

### Build profiles

Select `profile` in `workflow_dispatch`:

- **balanced** — `-O3`, JIT enabled. Best for Baileys/server workloads.
- **speed** — adds LTO. ~5-15% faster output, ~2x longer build.
- **size** — enables V8 lite-mode. Lower peak RAM but 2-3x slower crypto/JS.

> Minimum Android API: **24** (Android 7.0+). Required by Node.js v24+.

## Toolchain

| Component | Version | Notes |
|-----------|---------|-------|
| NDK | `r26d` | Clang 17 — last NDK without the turboshaft CRTP regression |
| Host | `ubuntu-22.04` | Pinned for NDK toolchain ABI stability |
| Target API | `android-24` | MinSdkVersion 24 |

> NDK r27+ ships Clang 18+, which fails to compile V8's turboshaft `AssemblerOpInterface<Next> : public Next` CRTP pattern on v26.x. See `deps/v8/src/compiler/turboshaft/assembler.h` for the affected code.

## Patches

This workflow applies community-maintained patches for Android cross-compilation. These are tracked upstream at:

- [nodejs/node#57748](https://github.com/nodejs/node/pull/57748) — Termux Android build patches
- [nodejs/node#58505](https://github.com/nodejs/node/issues/58505) — Android test failures
- [nodejs/node#58975](https://github.com/nodejs/node/issues/58975) — 32-bit arm cross-build V8 bugs

Patches cover: V8 stack tracing, trap handler, zlib cpu_features, std::atomic_ref polyfill, uv.gyp toolset fix, FICLONE removal, SMI assertion relaxation, and turboshaft CRTP workaround.

## Branches

| Branch | Node.js version | NDK | Status |
|--------|----------------|-----|--------|
| `main` | v26.x | r26d | Active development |
| `build_ndkr27` | v24.x LTS | r27d | Maintained (v24.x turboshaft unaffected) |

## License

[MIT](LICENSE)
