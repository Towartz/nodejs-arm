
<div align="center">

# Node.js for Android (ARM)

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=main&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Latest release](https://img.shields.io/badge/latest%20release-v26.x--run132-blue?logo=github)](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run132)
[![NDK](https://img.shields.io/badge/NDK-r29-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v26.x-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)

Standalone `node` CLI cross-compiled for Android ARM, built with GitHub Actions.

Binary statis, terhubung langsung ke libc++. Tidak butuh dependensi eksternal. Setiap rilis berisi binary, `libc++_shared.so` untuk build native addon, dan header lengkap untuk kompilasi N-API.

</div>

---

## Status build

| ABI | Balanced (JIT + `-O3`) | Speed (LTO) | Size (V8 lite mode) |
|---|---|---|---|
| `arm64-v8a` | Terverifikasi | Terverifikasi | Terverifikasi |
| `armeabi-v7a` | Berhasil, eksperimental | Eksperimental | Eksperimental |

`armeabi-v7a` masih eksperimental karena bug cross-build V8 di upstream. Detail ada di [nodejs/node#58975](https://github.com/nodejs/node/issues/58975). Build ini tidak pernah memblokir hasil `arm64-v8a`.

---

## Rilis terbaru

| Run | Tanggal | Branch | Trigger | ABI | Rilis |
|---|---|---|---|---|---|
| 132 | 2026-07-22 | `main` | push | arm64-v8a, armeabi-v7a | [node-android-v26.x--run132](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run132) |
| 130 | 2026-07-22 | `main` | push | arm64-v8a | [node-android-v26.x--run130](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v26.x--run130) |
| 128 | 2026-07-20 | `main` | push | arm64-v8a | [Run #29729197643](https://github.com/Towartz/nodejs-arm/actions/runs/29729197643) |

---

## Isi setiap rilis

Setiap ABI mendapat satu file zip. Isinya:

| File dalam zip | Fungsi |
|---|---|
| `node` | Binary CLI, statis penuh, di-strip, PIE, siap jalan langsung |
| `libc++_shared.so` | Salinan libc++ dari NDK, dipakai saat kamu build native addon |
| `include/node/*.h` | Header Node, V8, dan libuv, plus `config.gypi`, untuk kompilasi addon N-API |

Tidak ada `libnode.so` di paket ini. Binary `node` sudah statis penuh. Kalau aplikasi Android kamu perlu load lewat `System.loadLibrary()`, ganti nama file itu sendiri ke format `lib*.so`.

### Nama library kustom

Set input `lib_name` di workflow, misalnya `aurora`. Hasilnya jadi `aurora-android-<abi>.zip` dengan header yang sudah menyesuaikan.

---

## Profil build

| Profil | Flag tambahan | Kapan pakai |
|---|---|---|
| `balanced` | JIT + `-O3` | Default. Kecepatan runtime terbaik. |
| `speed` | + LTO | 5-15% lebih cepat, waktu build 2x lebih lama. |
| `size` | + v8-lite-mode | RAM lebih hemat, crypto dan JS 2-3x lebih lambat. |

---

## Toolchain

| Komponen | Versi | Catatan |
|---|---|---|
| NDK | r29 | Clang 21. Lompat dari r27d (Clang 18) per 2026-07-22. |
| Host runner | ubuntu-24.04 | Wajib untuk GCC 13.2+, syarat V8 versi ini. |
| API target minimum | android-24 | Android 7.0 ke atas. Jangan turunkan di bawah ini untuk Node v24+. |

### Flag build penting

- `-static-libstdc++`: binary jalan sendiri, tidak butuh `libc++_shared.so` di device.
- `-Wl,-z,max-page-size=16384`: kompatibel dengan page size 16 KB di Android 15/16.
- `-rdynamic` dan `--export-dynamic`: semua simbol tetap terlihat untuk native addon.
- `--openssl-no-asm`: mematikan optimisasi assembly OpenSSL untuk cross-compile. Lihat bagian Patch di bawah untuk alasannya.

---

## Patch yang diterapkan

Android bukan target resmi yang didukung Node.js. Patch berikut dipakai untuk cross-compile berhasil, dan perlu ditinjau ulang tiap kali Node naik versi major:

- Stack trace, trap handler, dan `uv.gyp` untuk libuv di Android.
- Polyfill `std::atomic_ref` karena libc++ NDK belum mengimplementasikannya.
- Perbaikan `zlib` cpu_features, ganti `android_getCpuFeatures()` dengan `getauxval()`.
- Downgrade `consteval` ke `constexpr` di regexp bytecode, workaround bug Clang lama.
- Perbaikan CRTP di Turboshaft assembler untuk lookup dua fase C++20.
- Perbaikan `decltype` di wrappers-inl.h untuk kompatibilitas Clang NDK.
- Nonaktifkan jalur simdutf atomic, karena tidak tersedia di libc++ Android.
- Perbaikan target asm OpenSSL, supaya build ARM tidak salah pakai asm x86_64.

Referensi upstream:

- [nodejs/node#57748](https://github.com/nodejs/node/pull/57748), patch Android dari Termux.
- [nodejs/node#58505](https://github.com/nodejs/node/issues/58505), kegagalan test Android.
- [nodejs/node#58975](https://github.com/nodejs/node/issues/58975), bug V8 cross-build 32-bit arm.

---

## Cara pakai

Jalankan lewat tab Actions, pilih workflow Node.js for Android (ARM), lalu klik Run workflow. Kamu bisa atur versi Node, nama library, dan profil build.

Push ke branch `main` juga memicu build otomatis. Hasilnya langsung naik ke GitHub Release dengan tag `node-android-*`.

---

## Branch yang tersedia

| Branch | Versi Node | NDK | Ubuntu | Status |
|---|---|---|---|---|
| `main` | v26.x | r29 | 24.04 | CI aktif |
| `v24.x-lts` | v24.x LTS | r29 | 24.04 | Terverifikasi |

---

## Lisensi

GPLv3 dengan tambahan pembatasan. Detail lengkap ada di file LICENSE.

Dibangun oleh [21_whiten (Towartz)](https://github.com/Towartz).
