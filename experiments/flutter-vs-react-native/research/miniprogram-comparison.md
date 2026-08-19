# 小程序双端对比调研（修正版）

> 日期：2026-05-30
> 对应 TODO：M3-1, M3-2
> **关键假设修正：UI 层不需要复用，核心是数据模型复用**

## 核心策略

实验目标不是"把 Flutter/RN 代码编译到小程序"，而是：
1. ✅ 数据模型跨平台共享（M1 已验证：OpenAPI → Dart/TS）
2. ✅ 业务逻辑层可提取为共享包
3. ❌ UI 不需要复用 — 小程序 UI 独立设计

在这个策略下，主框架选型（Flutter vs RN）对小程序开发的影响很小。

---

## RN → 小程序（参考）

| 方案 | 特点 |
|------|------|
| **Taro 4.x** | 开源(MIT)、npm 可安装、React 生态、成熟 |
| **uni-app** | Vue 生态，团队非 React 不适用 |
| **共享 Zustand store** | 纯 TS，可在 RN 和 Taro 间共享业务逻辑 |

## Flutter → 小程序（2026 年方案全景）

| 方案 | 类型 | 状态 | 生产可用？ | 免费？ |
|------|------|:----:|:---------:|:------:|
| **MPFlutter** | Flutter 框架（非编译器） | ✅ 活跃 | ✅ | ❌ 需商业授权 |
| **flutter_mp** (areslabs) | 编译器（实验性） | ❌ 2019 年停更 | ❌ | ✅ MIT |
| **zhaomenghuan/flutter-mini-program** | Flutter 内小程序引擎 | ❌ 2019 年停更 | ❌ | ✅ |
| **FinClip/MoP** | 小程序容器（嵌入 Flutter） | ✅ 活跃 | ✅ | ❌ 商业 |
| **NUI Mini Program** | 小程序容器 | ⚠️ 2025-11 更新 | 部分 | 待确认 |
| **mini_program_platform** (mehedi) | 便携小程序平台 | 🆕 2026-04 起活跃 | v0.3.x 早期 | 待确认 |
| **unimp** | uni-app 小程序运行时 | ❌ 2023 停更 | ❌ | 待确认 |
| **Flutter Web + WebView** | 嵌入方案 | N/A | 有限（无法调原生 API） | ✅ |

### 实际验证

| 方案 | 验证结果 |
|------|---------|
| **MPFlutter** | 不在 pub.dev，`dart pub global activate` 失败；官方声明"并非完全开源"，商业使用需购买授权 |
| **flutter_mp** | 仓库仍存在（`areslabs/flutter_mp`，742 stars），但 2019 年后未更新，不可用 |
| **mini_program_platform** | 最新（2026-05 更新 v0.3.x），但尚未发布到主 pub.dev 或未稳定 |

## 修正后的判断

| 维度 | Flutter → 小程序 | RN → 小程序 |
|------|:---------------:|:-----------:|
| 自动转换工具 | ❌ 无不需 — UI 本就不打算复用 | ❌ 无不需 — UI 本就不打算复用 |
| 数据模型复用 | ✅ OpenAPI → TS 输出（M1 已验证） | ✅ OpenAPI → TS 输出（M1 已验证） |
| 业务逻辑复用 | 需 Dart→JS 转换工具或手动重写 | Zustand store 可直用 |
| 小程序框架选择 | 不受限（微信原生/Taro 均可） | Taro 更自然（同一 React 生态） |

**结论：** 小程序场景不偏袒 Flutter 或 RN。数据模型复用双方等价（M1 已验证），UI 均独立开发。业务逻辑复用的强弱取决于是否选择同一前端生态（React）。

### 关于 Flutter 小程序方案的补充说明

2026 年存在多个 Flutter→小程序方案，但**没有一个能实现"将现有 Flutter 代码自动编译到微信小程序"**：

- **编译器类**（flutter_mp）：2019 年就死了，从未成熟
- **框架类**（MPFlutter）：需用其 API 重写代码，非编译器，且需商业授权
- **容器类**（FinClip/NUI）：在 Flutter 内运行小程序，不是"Flutter 代码转小程序"
- **新兴方案**（mini_program_platform）：2026 年 4 月刚刚出现，v0.3.x 早期阶段

对于"UI 不要求复用，仅需数据模型共享"的策略，有无这些方案不影响结论。
