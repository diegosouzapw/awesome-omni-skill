---
name: a2ui-v0-8
description: 使用 A2UI v0.8（以 renderers/lit/src/0.8 与 specification/0.8 为准）生成服务端 JSONL UI 消息，并实现客户端事件与 A2A 扩展回传。适用于把用户需求转成 A2UI 组件树+数据模型+渲染序列，以及处理 action/userAction 与后端接口调用的场景。
---

# A2UI v0.8 统一技能

## 概览
提供 A2UI v0.8 的端到端规范与实操流程：UI 生成（surfaceUpdate/dataModelUpdate/beginRendering/deleteSurface）与客户端事件回传（action → userAction → A2A）。

## 工作流（必须遵循）
1. 确认 Catalog 与 surfaceId：默认标准 Catalog，所有组件 id 唯一。
2. 设计组件图：扁平组件列表 + children/child 组装树。
3. 写入数据模型：dataModelUpdate 填充 ValueMap，并用 path 绑定。
4. 触发渲染：发送 beginRendering 指定 root。
5. 交互回传：组件 action.context 绑定数据，客户端解析为 userAction。
6. 增量更新：结构变更 surfaceUpdate，数据变更 dataModelUpdate。

## 生成侧规则
- 每条消息只能有一个顶级键：beginRendering / surfaceUpdate / dataModelUpdate / deleteSurface。
- surfaceUpdate.components 中 component 只能有一个组件类型键。
- children 使用 explicitList 或 template（template 必须含 componentId 与 dataBinding）。
- dataModelUpdate.contents 只使用 ValueMap（valueString/valueNumber/valueBoolean/valueMap）。
- BoundValue 的 literal* 与 path 二选一；相对 path 以 dataContextPath 解析。

## 客户端事件规则
- userAction 必填：name、surfaceId、sourceComponentId、timestamp。
- action.context 必须解析成最终值（不可保留 path）。
- 发送 DataPart 时 mimeType 必须与服务端一致，规范为 application/json+a2ui。
- 每次 A2A 消息都携带 a2uiClientCapabilities（supportedCatalogIds）。

## 兼容性提示
- TextField 字段命名在 schema 与 renderer 存在差异：schema 常用 textFieldType，Lit v0.8 类型用 type。接入时必须做字段对齐或映射。
- Modal 的 entryPointChild 会被渲染器赋 slot="entry"，不要复用 slot 名称。

## 参考资料（按需加载）
- 协议与数据模型：`references/protocol-v0-8.md`
- 标准组件清单：`references/standard-catalog-v0-8.md`
- Schema 定位：`references/schemas-v0-8.md`
- JSONL 模板示例：`references/examples-jsonl-v0-8.md`
- 片段库：`references/snippets-v0-8.md`
- 客户端事件解析：`references/client-events-v0-8.md`
- A2A 扩展与能力协商：`references/a2a-extension-v0-8.md`
- 端到端流程：`references/sample-client-flow.md`
- 端到端请求模板：`usage.md`
