# ADR-0001: 客户端框架选型——Flutter（Studio）+ Taro（Slice）

> 状态：✅ 已接受（2026-05-30，基于量潮软件工程创新实验室实验验证）
> 来源：实验"Flutter vs React Native 技术栈选型"（实验室 `examples/default/examples/flutter-vs-react-native`，已删除）

## 背景

- 主力是 AI，但 **AI 不擅长把握数据模型**——数据模型层必须单独拆出来作为独立库，与 UI 框架解耦
- 需要预留小程序对接能力（RN 在小程序生态有 Taro / uni-app 优势）
- 团队积累：主力是 Dart（Flutter + BLoC），几乎没有 JS 积累（实验证明 AI 辅助大幅降低语言门槛）
- 候选方案：**Flutter** vs **React Native**（小程序场景：Taro / MPFlutter）

## 实验验证（5 条假设）

| # | 假设 | 结论 | 对选型影响 |
|---|------|------|-----------|
| 1 | 数据模型层可独立为纯逻辑库，与 Flutter / RN 均解耦 | ✅ 成立（OpenAPI 3.x + OpenAPI Generator） | 持平 |
| 2 | 团队 Dart + BLoC 积累使 Flutter 初始效率显著更高 | ✅ 成立（BLoC API 稳定，AI 生成质量高） | Flutter +1 |
| 3 | Flutter 工具链优势 > RN 生态复用优势 | ✅ 成立（`dart analyze` 静态分析强于默认 `tsc`） | Flutter +1 |
| 4 | 小程序代码复用（UI 层不要求复用，仅数据模型） | 双方等价（数据模型复用 M1 已解决，UI 均独立开发） | 持平 |
| 5 | AI 生成 Dart 质量 >= AI 生成 TS/JS | ≈ 持平（**框架 API 稳定性比语言本身更影响 AI 生成质量**） | 持平 |

**决策矩阵：Flutter +2 vs RN 0。**

关键发现：
- 双端简单页面开发效率持平（AI 辅助下均约 1h 零编译问题），纯手动编码场景 Flutter 预期优势更明显
- MPFlutter **不在 pub.dev、需商业授权**，不可直接使用；Taro 开源可用但 UI 也需重写
- 若业务逻辑层需复用（如 Zustand store），RN + Taro 有优势，但团队无 RN 积累

## 决策

1. **Studio（主应用）**：Flutter + BLoC（Material 3）
2. **Slice（小程序）**：Taro（TS + Zustand）
3. **数据模型**：`spec/openapi.yaml` 为单一事实来源，OpenAPI Generator 双端生成（Dart / TS）
4. **存储层策略**：双端存储（离线缓存、本地数据库）以 OpenAPI 模型为准，端上只做序列化/反序列化适配，**不自行重定义模型结构**，避免两端走样

```
spec/openapi.yaml          ← 数据模型单一事实来源
       │
       ├── OpenAPI Gen (dart) → Studio (Flutter + BLoC)
       └── OpenAPI Gen (ts)   → Slice (Taro)
```

## 后果

### 正面

- 复用团队 Dart + BLoC 积累，无语言/框架学习成本
- Flutter 静态分析（`dart analyze`）在开发期提前发现问题（废弃 API、类型安全、未使用变量）
- 数据模型单源双端生成，两端结构一致，不会走样
- 小程序能力不受主框架选型影响（UI 独立开发、数据模型共享等价）

### 负面 / 代价

- RN / React 生态不可用：放弃"业务逻辑层复用 + Taro"的 React 生态优势
- MPFlutter 不可用（商业授权 + 不在 pub.dev）
- 团队需维护 Dart / TS 双语言产物（数据模型独立库尚未创建，**待落地**：Dart + npm 双端发布需在 Studio 和 Slice 正式开发前完成）

## 关联

- 决策矩阵与 PoC 结论均记录于本 ADR（实验详情随实验代码一并删除）
