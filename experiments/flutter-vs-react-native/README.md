# 实验记录：Flutter vs React Native 技术栈选型

> 状态：✅ 已完成（2026-05-30）
> 决策产出：ADR-0001（客户端框架选型——Flutter Studio + Taro Slice）
> 本记录用于**恢复实验**：原实验代码位于 `examples/default/examples/flutter-vs-react-native`（已删除），按本记录可完整重建。

## 实验目的

在"AI 是主力开发者"的前提下选型客户端框架，回答两个问题：
1. Flutter 与 React Native 哪个更适合做主应用（Studio）？
2. 是否需要预留小程序能力？主框架选型对小程序有什么影响？

关键前置判断：**AI 不擅长把握数据模型**，所以数据模型层要单独拆出来作为独立库，与 UI 框架解耦——这个判断本身就是实验的起点。

## 假设列表

| # | 假设 | 结论 |
|---|------|------|
| 1 | 数据模型层可独立为纯逻辑库，与 Flutter / RN 均解耦 | ✅ 成立（OpenAPI 3.x + OpenAPI Generator） |
| 2 | 团队 Dart + BLoC 积累使 Flutter 初始开发效率显著高于 RN | ✅ 成立（BLoC API 稳定，AI 生成质量高） |
| 3 | Flutter 原生工具链效率优势 > RN 生态库复用效率优势 | ✅ 成立（`dart analyze` 强于默认 `tsc`） |
| 4 | 小程序代码复用（UI 层不要求复用，仅数据模型） | 双方等价（数据模型复用 M1 解决，UI 均独立开发） |
| 5 | AI 生成 Dart 代码质量 >= AI 生成 TS/JS | ≈ 持平（框架 API 稳定性比语言本身更影响 AI 生成质量） |

## 方法（分 PoC 验证）

### M1（PoC #1）：数据模型层解耦验证 —— 2.5h

- 选定 IDL：**OpenAPI 3.x + OpenAPI Generator**（见 `research/idl-selection.md`，五方案评分 22/25 最高）
- 定义 User/Product/Order 等 11 个 Schema（`spec/openapi.yaml`，单一事实来源）
- 生成 Dart client（1618 LOC，`json_serializable` 输出）与 TS client（完整类型映射）
- 验证点：required/pattern/enum/nested 正确生成、双向编译零错误
- **结论：数据模型层可以纯 IDL 形式独立存在，无痛双端集成**

### M2（PoC #2）：开发效率对比 —— 2h（双端各约 1h）

- 统一验收页面规范：产品管理（列表 + 创建表单 + 状态处理），见 `spec/acceptance-page.md`
- Flutter 侧：BLoC 8.x + Material 3，首次编译零问题
- RN 侧：Expo + Zustand，首次编译零问题
- **发现：AI 辅助下双端效率持平（约 1h），重度依赖 AI 缩小了 Dart 积累的差距；纯手动编码场景 Flutter 预期优势更明显**
- 状态管理选型依据见 `research/state-mgmt-selection.md`（Flutter 选 BLoC，RN 选 Zustand）

### M3（PoC #3）：小程序复用可行性 —— 4.5h

- 核心策略：**UI 层不要求复用**，仅数据模型 + 业务逻辑层复用
- 双端调研（2h）+ Taro 项目搭建 + Zustand store 复用尝试（2h）+ MPFlutter 验证（0.5h），见 `research/miniprogram-comparison.md`
- **发现：MPFlutter 不在 pub.dev 需商业授权不可用；flutter_mp 2019 年停更；2026 年无方案能"将现有 Flutter 代码自动编译到小程序"**
- **结论：UI 不要求复用后双端等价；业务逻辑复用强弱取决于是否选 React 生态（Zustand store 可直用）**

### M4（PoC #4）：AI 生成代码质量对比 —— 0.5h

- 同一 AI + 同一功能需求（产品管理页面），双端分别生成
- **结果：双端均首次编译通过、0 修正轮次**（见 `research/ai-codegen-comparison.md`）
- **结论：框架 API 稳定性（BLoC / Zustand）比语言本身更影响 AI 生成质量**

### M5：综合决策 —— 0.5h

- 决策矩阵：每条假设给 Flutter/RN 打 +1/0/-1 → **Flutter +2 vs RN 0**
- 产出 ADR-0001

总故事点 58（AI 辅助实际用时约 30 分钟）。

## 决策

**Studio（主应用）= Flutter + BLoC；Slice（小程序）= Taro（TS + Zustand）；数据模型 = OpenAPI 单一事实来源双端生成。**

存储层策略：双端存储以 OpenAPI 模型为准，端上只做序列化/反序列化适配，不自行重定义模型结构，避免两端走样。

## 恢复步骤（重建本实验）

### 环境

- Flutter SDK（含 Dart）、`flutter_bloc` 8.x
- Node.js + npm；Expo（RN 侧）、Taro CLI 4.x（小程序侧）、Zustand
- OpenAPI Generator CLI 7.22.0（配置见 `openapitools.json`，npm 包 `@openapitools/openapi-generator-cli`）

### 重建步骤

1. **数据模型**：取 `spec/openapi.yaml`，执行双端生成：
   ```
   openapi-generator generate -i spec/openapi.yaml -g dart -o generated/flutter_client
   openapi-generator generate -i spec/openapi.yaml -g typescript -o generated/ts_client
   ```
   验证 11 个 Schema 的 required/pattern/enum/nested 生成正确、双端编译零错误（M1）
2. **验收页面**：按 `spec/acceptance-page.md` 的产品管理页面规范，分别实现 Flutter（BLoC + Material 3）与 RN（Expo + Zustand）版本，记录从零到跑通耗时与编译问题数（M2）
3. **小程序**：搭建 Taro 项目尝试复用 Zustand store；尝试 `dart pub global activate mpflutter` 验证 MPFlutter 不可用性（M3）
4. **AI 对比**：同一 AI、同一 prompt（产品管理页面），双端生成对比首次编译通过率与修正轮次（M4）
5. **决策**：按假设表逐条打分（+1/0/-1），汇总矩阵，对照 ADR-0001 验证决策一致性（M5）

### 与 ADR-0001 的对应

| 决策要素 | ADR-0001 状态 | 本记录位置 |
|---------|--------------|-----------|
| 数据模型解耦（OpenAPI） | 已接受 | `spec/openapi.yaml` + `research/idl-selection.md` |
| Studio = Flutter + BLoC | 已接受 | `research/state-mgmt-selection.md`（Flutter 侧） |
| Slice = Taro | 已接受 | `research/miniprogram-comparison.md` |
| 存储层不重定义模型 | 已接受 | 本记录"决策"节 |

## 待落地事项

- 数据模型独立库（Dart + npm 双端发布）尚未创建，需在 Studio 和 Slice 正式开发前完成
