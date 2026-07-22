<div align="center">

# Node.js for Android ARM — **v26.x**

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=main&label=CI&logo=github&color=success)](https://github.com/Towartz/nodejs-arm/actions)
[![Last build](https://img.shields.io/badge/last%20build-%23136%20(Jul%2022)-success)](https://github.com/Towartz/nodejs-arm/actions/runs/29906238656)
[![Latest release](https://img.shields.io/badge/latest%20release-v26.x--run132-blue?logo=github)](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run132)
[![NDK](https://img.shields.io/badge/NDK-r29(Clang%2019)-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v26.x-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)
[![Views](https://hits.sh/github.com/Towartz/nodejs-arm.svg?label=views&color=555)](https://hits.sh/github.com/Towartz/nodejs-arm/)

**v26.x** — **✅ CI passing** — Cross-compiled standalone static `node` CLI for Android ARM64 / ARM32 via GitHub Actions.

Statically linked against libc++ (zero external dependencies), bundled with `libc++_shared.so` for native N-API addon builds and `*-headers.zip` per ABI for compiling addons against the exact build config.

</div>

---

## Status

| ABI | Balanced (JIT + `-O3`) | Speed (LTO) | Size (V8 lite-mode) |
|-----|------------------------|-------------|---------------------|
| `arm64-v8a` | ✅ Verified | ✅ Verified | ✅ Verified |
| `armeabi-v7a` | ✅ Included* | ⚠️ Experimental | ⚠️ Experimental |

> *`armeabi-v7a` balanced builds succeed but are still considered **experimental** upstream ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975)).

> `armeabi-v7a` speed/size profiles: experimental — V8 cross-build bugs ([nodejs/node#58975](https://github.com/nodejs/node/issues/58975))

---

## Latest Builds

| # | Date | Branch | Event | ABI | Release |
|---|------|--------|-------|-----|---------|
| 132 | 2026-07-22 | `main` | `push` | `arm64-v8a` + `armeabi-v7a` | [`node-android-v26.x--run132`](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run132) |
| 130 | 2026-07-22 | `main` | `push` | `arm64-v8a` | [`node-android-v26.x--run130`](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run130) |
| 128 | 2026-07-20 | `main` | `push` | `arm64-v8a` | [Actions run #29729197643](https://github.com/Towartz/nodejs-arm/actions/runs/29729197643) |

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

## Recent activity

| Date | Commit | Message |
|------|--------|---------|
| 2026-07-22 | [`9f784a5`](https://github.com/Towartz/nodejs-arm/commit/9f784a5) | Fix OpenSSL asm target selection for Android builds |
| 2026-07-22 | [`e334d87`](https://github.com/Towartz/nodejs-arm/commit/e334d87) | Refactor OpenSSL ASM flag handling in workflow |
| 2026-07-22 | [`fbc53e0`](https://github.com/Towartz/nodejs-arm/commit/fbc53e0) | Configure OpenSSL asm optimizations for arm64 |
| 2026-07-22 | [`646dc2f`](https://github.com/Towartz/nodejs-arm/commit/646dc2f) | Refactor Node.js headers packaging in workflow |
| 2026-07-22 | [`bf1a1d5`](https://github.com/Towartz/nodejs-arm/commit/bf1a1d5) | Clarify bundled OpenSSL and zlib headers usage |
| 2026-07-22 | [`e99d2c4`](https://github.com/Towartz/nodejs-arm/commit/e99d2c4) | Refactor Node.js build workflow by removing patchelf |
| 2026-07-22 | [`0a9e876`](https://github.com/Towartz/nodejs-arm/commit/0a9e876) | Upgrade Android NDK to r29 and update patches |
| 2026-07-22 | [`5c5e671`](https://github.com/Towartz/nodejs-arm/commit/5c5e671) | Update Android build workflow for Ubuntu 24.04 |
| 2026-07-21 | [`8b41e86`](https://github.com/Towartz/nodejs-arm/commit/8b41e86) | Refactor Node.js packaging for static binary |

---

## Toolchain

| Component | Version | Notes |
|-----------|---------|-------|
| NDK | `r29` | Clang 19 — last known-good NDK for v26.x CRTP patterns |
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

On `push` to `main` or `v24.x-lts`, builds run automatically. Artifacts are promoted to a **GitHub Release** with a `node-android-*` tag.

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
