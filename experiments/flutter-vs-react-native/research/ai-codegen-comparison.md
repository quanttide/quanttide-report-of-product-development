# AI 生成代码质量对比

> 日期：2026-05-30

## 方法

同一 AI 模型 + 同一功能需求（产品管理页面），对比 Flutter/BLoC 和 RN/Zustand。

## 结果

| 对比项 | Dart (Flutter + BLoC) | TypeScript (Expo + Zustand) |
|--------|:---------------------:|:---------------------------:|
| 首次编译通过 | ✅ BLoC 8.x API 稳定 | ✅ Zustand API 稳定 |
| 修正轮次 | 0 轮 | 0 轮 |
| AI 理解准确度 | ⭐⭐⭐⭐⭐ BLoC 模式训练数据充分 | ⭐⭐⭐⭐⭐ Zustand 极简 |
| 纯逻辑代码 | ✅ 一次通过 | ✅ 一次通过 |
| 类型安全 | `dart analyze` 强检查 | `tsc` 强检查 |

## 结论

| 假设 | 结论 |
|------|------|
| **#5 AI 生成 Dart 质量 >= AI 生成 TS 质量** | **≈ 持平** — 在 API 稳定的框架（BLoC/Zustand）下，AI 对两者的生成质量一致 |
