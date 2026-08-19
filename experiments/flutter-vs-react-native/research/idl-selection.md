# IDL 方案调研

> 日期：2026-05-30
> 对应 TODO：M1-1, M1-2

## 需求

- 单一数据模型定义源 → 生成 Dart + TypeScript 代码
- 支持序列化（JSON）、校验
- AI 可良好理解与生成

## 候选方案评分

| 方案 | Dart 代码生成 | TS 代码生成 | 校验支持 | AI 兼容性 | 维护成本 | 总分 |
|------|:------------:|:----------:|:--------:|:---------:|:--------:|:----:|
| A: Protobuf | 5 | 4 | 2 | 5 | 2 | 18 |
| B: JSON Schema + Quicktype | 4 | 4 | 5 | 5 | 3 | 21 |
| C: Dart-first | 5 | 1 | 3 | 3 | 1 | 13 |
| D: TypeSpec (Microsoft) | 1 | 4 | 5 | 4 | 2 | 16 |
| **E: OpenAPI 3.x + OpenAPI Generator** | **5** | **5** | **4** | **5** | **3** | **22** |

## 选定方案

**OpenAPI 3.x + OpenAPI Generator**

理由：
1. Dart 和 TypeScript 代码生成均为 STABLE 级别，生态最成熟
2. AI 训练数据覆盖最广，Prompt 生成质量最高
3. Schema 校验规则（required / pattern / minLength 等）可直接映射到表单校验
4. 业务应用最终往往需要 API 层，OpenAPI 可自然扩展，无需后续迁移
5. Dart 侧使用 `json_serializable` 输出，符合 Flutter 标准实践

## 集成路线

```
OpenAPI Spec (spec.yaml)
  components:
    schemas:
      User: ...
  │
  ├── openapi-generator generate -g dart     → dart_client/lib/model/
  └── openapi-generator generate -g typescript → ts_client/src/model/
```

## 工具链

- **OpenAPI Generator CLI**：npm 包 `@openapitools/openapi-generator-cli`
- **Spec 编写**：YAML，AI 辅助生成
- **运行时校验**：Dart 侧 `json_schema`，TS 侧 `ajv`
