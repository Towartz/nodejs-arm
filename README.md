<div align="center">

# Node.js for Android (ARM) — v24.x LTS 分支

[![CI](https://img.shields.io/github/actions/workflow/status/Towartz/nodejs-arm/node-android.yml?branch=v24.x-lts&label=CI&logo=github)](https://github.com/Towartz/nodejs-arm/actions)
[![Latest release](https://img.shields.io/badge/latest%20release-v24.x--run117-blue?logo=github)](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run117)
[![NDK](https://img.shields.io/badge/NDK-r29-blue?logo=android)](https://developer.android.com/ndk)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen?logo=android)](https://developer.android.com/guide/topics/manifest/uses-sdk-element)
[![Node](https://img.shields.io/badge/Node-v24.x%20LTS-339933?logo=nodedotjs)](https://nodejs.org)
[![License](https://img.shields.io/github/license/Towartz/nodejs-arm)](LICENSE)

通过 GitHub Actions 交叉编译的 Android ARM 独立 node 命令行程序，本分支对应 Node.js v24.x LTS 版本。

二进制文件完全静态链接 libc++，不依赖任何外部库。每个发布包都包含二进制文件、用于编译原生插件的 libc++_shared.so，以及完整的 N-API 编译头文件。

</div>

---

## 构建状态

| ABI | 均衡模式 (JIT + -O3) | 速度模式 (LTO) | 体积模式 (V8 精简模式) |
|---|---|---|---|
| arm64-v8a | 已验证 | 已验证 | 已验证 |
| armeabi-v7a | 实验性 | 实验性 | 实验性 |

armeabi-v7a 仍属实验性质，原因是上游 V8 存在交叉编译缺陷。详情见 [nodejs/node#58975](https://github.com/nodejs/node/issues/58975)。

---

## 最新发布

| 构建编号 | 日期 | 分支 | 触发方式 | ABI | 发布链接 |
|---|---|---|---|---|---|
| 117 | 2026-07-20 | v24.x-lts | push | arm64-v8a | [node-android-v24.x--run117](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run117) |
| 114 | 2026-07-19 | v24.x-lts | workflow_dispatch | arm64-v8a | [node-android-v24.x--run114](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run114) |
| 108 | 2026-07-19 | v24.x-lts | workflow_dispatch | arm64-v8a | [node-android-v24.x--run108](https://github.com/Towartz/nodejs-arm/releases/tag/node-android-v24.x--run108) |
| 107 | 2026-07-19 | v24.x-lts | workflow_dispatch | arm64-v8a | [aurora_node-android-v24.x--run107](https://github.com/Towartz/nodejs-arm/releases/tag/aurora_node-android-v24.x--run107) |

---

## 发布包内容

每个 ABI 对应一个压缩包，内容如下。

| 压缩包内文件 | 作用 |
|---|---|
| node | 命令行二进制文件，完全静态、已剥离符号、PIE，可直接运行 |
| libc++_shared.so | NDK 自带的 libc++ 副本，用于编译原生插件时使用 |
| include/node/*.h | Node、V8、libuv 的头文件，加上 config.gypi，用于编译 N-API 插件 |

此发布包不包含 libnode.so，node 二进制文件本身已是完全静态。如果你的 Android 应用需要通过 System.loadLibrary() 加载，请自行将文件重命名为 lib*.so 格式。

### 自定义库名称

在工作流输入中设置 lib_name，例如 aurora，生成结果会变成 aurora-android-<abi>.zip，对应头文件也会同步调整。

---

## 构建模式

| 模式 | 附加编译选项 | 适用场景 |
|---|---|---|
| balanced | JIT + -O3 | 默认模式，运行速度最佳 |
| speed | 追加 LTO | 速度提升 5% 到 15%，构建时间延长约一倍 |
| size | 追加 v8-lite-mode | 更省内存，加密和 JS 运算速度降低 2 到 3 倍 |

---

## 工具链

| 组件 | 版本 | 说明 |
|---|---|---|
| NDK | r29 | 跟随最新稳定版 |
| 构建主机 | ubuntu-24.04 | 固定版本，保证工具链行为一致 |
| 最低目标 API | android-24 | 对应 Android 7.0 及以上 |

### 关键编译参数

- `-static-libstdc++`：二进制文件独立运行，设备端无需 libc++_shared.so
- `-Wl,-z,max-page-size=16384`：兼容 Android 15/16 的 16KB 页面大小
- `-rdynamic` 与 `--export-dynamic`：保留所有符号，供原生插件调用
- `--openssl-no-asm`：交叉编译时关闭 OpenSSL 汇编优化，避免架构不匹配问题

---

## 已应用的补丁

Android 并非 Node.js 官方支持的目标平台。以下补丁用于确保交叉编译成功。

补丁涵盖内容：V8 堆栈跟踪、trap handler、zlib cpu_features 检测、std::atomic_ref 兼容实现、uv.gyp 工具集修复、FICLONE 移除、SMI 断言放宽、Turboshaft CRTP 的 consteval 降级为 constexpr、OpenSSL 汇编目标检测适配 Android。

上游相关链接：

- [nodejs/node#57748](https://github.com/nodejs/node/pull/57748)，来自 Termux 项目的 Android 补丁
- [nodejs/node#58505](https://github.com/nodejs/node/issues/58505)，Android 测试失败记录
- [nodejs/node#58975](https://github.com/nodejs/node/issues/58975)，32 位 arm 交叉编译中的 V8 缺陷

---

## 使用方法

打开 Actions 标签页，选择 Node.js for Android (ARM) 工作流，点击 Run workflow。

推送到 v24.x-lts 分支会自动触发构建，构建完成后会自动发布到 GitHub Release，标签格式为 node-android-*。

---

## 可用分支

| 分支 | Node 版本 | NDK | Ubuntu | 状态 |
|---|---|---|---|---|
| main | v26.x | r29 | 24.04 | CI 正常运行 |
| v24.x-lts | v24.x LTS | r29 | 24.04 | 已验证 |

---

## 许可证

GPLv3 附加限制条款，完整内容见 LICENSE 文件。

由 [21_whiten (Towartz)](https://github.com/Towartz) 构建。
