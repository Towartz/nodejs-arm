<div align="center">

# Node.js for Android ARM — **v24.x LTS**

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=v24.x-lts&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Last build](https://img.shields.io/badge/last%20build-%23137%20(Jul%2022)-success)](https://github.com/Towartz/nodejs-arm/actions/runs/29913681810)
[![Latest release](https://img.shields.io/badge/latest%20release-v24.x--run117-blue?logo=github)](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run117)
[![NDK](https://img.shields.io/badge/NDK-r29(Clang%2019)-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v24.x(LTS)-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)
[![Views](https://hits.sh/github.com/Towartz/nodejs-arm.svg?label=views&color=555)](https://hits.sh/github.com/Towartz/nodejs-arm/)

**v24.x LTS** — **✅ Verified** — Cross-compiled standalone static `node` CLI for Android ARM64 / ARM32 via GitHub Actions.

Statically linked against libc++ (zero external dependencies), bundled with `libc++_shared.so` for native N-API addon builds and `*-headers.zip` per ABI for compiling addons against the exact build config.

</div>

---

## Status

| ABI | Balanced (JIT + `-O3`) | Speed (LTO) | Size (V8 lite-mode) |
|-----|------------------------|-------------|---------------------|
| `arm64-v8a` | ✅ Verified | ✅ Verified | ✅ Verified |
| `armeabi-v7a` | ⚠️ Experimental | ⚠️ Experimental | ⚠️ Experimental |

> `armeabi-v7a`: experimental — V8 cross-build bugs ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975))

---

## Latest Builds

| # | Date | Branch | Event | ABI | Release |
|---|------|--------|-------|-----|---------|
| 117 | 2026-07-20 | `v24.x-lts` | `push` | `arm64-v8a` | [`node-android-v24.x--run117`](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run117) |
| 114 | 2026-07-19 | `v24.x-lts` | `workflow_dispatch` | `arm64-v8a` | [`node-android-v24.x--run114`](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run114) |
| 108 | 2026-07-19 | `v24.x-lts` | `workflow_dispatch` | `arm64-v8a` | [`node-android-v24.x--run108`](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run108) |
| 107 | 2026-07-19 | `v24.x-lts` | `workflow_dispatch` | `arm64-v8a` | [`aurora_node-android-v24.x--run107`](https://github.com/Towartz/nodejs-arm/releases/tag/aurora_node-android-v24.x--run107) |

---

## What you get

Each build produces one release per successful run, with **raw files** (not double-packed — GitHub already wraps each artifact in a `.zip`):

| File | Description |
|------|-------------|
| `<lib_name>-android-<abi>` | Standalone, fully static `node` CLI binary (PIE, stripped, `-rdynamic`, `-static-libstdc++`, max-page-size=16K) |
| `<lib_name>-android-<abi>-libc++_shared.so` | NDK's `libc++_shared.so` bundled for native-addon builds (node itself does **not** depend on it) |
| `<lib_name>-android-<abi>-headers.zip` | Node public headers + V8/libuv includes + `config.gypi` — for compiling N-API addons against this build |

> **Note:** No `libnode.so` — the binary is a fully self-contained static ELF. Rename it to `lib*.so` yourself if your Android app needs to `System.loadLibrary()` it with exec permissions.

### Custom library name

Set `lib_name` workflow input to `aurora` → produces `aurora-android-<abi>` with matching headers.

---

## Build profiles

| Profile | Flags | When to use |
|---------|-------|-------------|
| `balanced` | JIT + `-O3` | Best runtime speed (default) |
| `speed` | `+LTO` | ~5–15% faster output, ~2× longer build |
| `size` | `+v8-lite-mode` | Lower peak RAM, 2–3× slower crypto/JS |

---

## Toolchain

| Component | Version | Notes |
|-----------|---------|-------|
| NDK | `r29` | Clang 19 — tracking latest stable |
| Host | `ubuntu-24.04` | Pinned for toolchain ABI stability |
| Target API | `android-24` | Minimum API 24 (Android 7.0+) |

### Key build flags

- `-static-libstdc++` — fully standalone binary, no runtime dep on device's libc++
- `-Wl,-z,max-page-size=16384` — Android 15/16 compatibility
- `-rdynamic` / `--export-dynamic` — all symbols visible for native addons
- `--openssl-no-asm` — disable OpenSSL assembly (avoids arch/OS mismatch in cross-compile)

---

## Patches

This workflow applies community-maintained patches for Android cross-compilation. Tracked upstream at:

- [nodejs/node#57748](https://github.com/nodejs/node/pull/57748) — Termux Android build patches
- [nodejs/node#58505](https://github.com/nodejs/node/issues/58505) — Android test failures
- [nodejs/node#58975](https://github.com/nodejs/node/issues/58975) — 32-bit arm cross-build V8 bugs

Patches cover: V8 stack tracing, trap handler, zlib cpu_features, `std::atomic_ref` polyfill, uv.gyp toolset fix, FICLONE removal, SMI assertion relaxation, turboshaft CRTP `consteval`→`constexpr` downgrade, OpenSSL `asm_target` detection for Android.

---

## Usage

Trigger a build via **Actions** → **Node.js for Android (ARM)** → **Run workflow**.

On `push` to `v24.x-lts`, builds run automatically. Artifacts are promoted to a **GitHub Release** with a `node-android-*` tag.

---

## Branches

| Branch | Node.js | NDK | Ubuntu | Status |
|--------|---------|-----|--------|--------|
| **`main`** | **v26.x** | **r29** | **24.04** | **✅ CI passing** |
| **`v24.x-lts`** | **v24.x LTS** | **r29** | **24.04** | **✅ Verified** |

---

## License

[GPLv3 with Additional Restrictions](LICENSE)

Built by [**21_whiten** (Towartz)](https://github.com/Towartz) — 140+ repos, 32 followers, since 2021.
