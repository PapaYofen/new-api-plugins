---
changelogVersion: 1
plugin: "tripo"
version: "1.0.0"
locale: "zh-CN"
---
# Changelog

## [1.0.0]

### Added

- 可以在网关上用 Tripo 生成和加工三维模型，包括文生、图生、多视图生成、贴图、转换、网格工具、绑骨和文件上传。
- 可以调用 Tripo 自己的路径，例如 `POST /v3/generation/text-to-model`，也可以用 `/tripo` 前缀下的同一路径。用网关任务号在 `GET /v3/tasks/{task_id}` 查询。
- 贴图、转换或绑骨等后续请求传入网关任务号，网关会换成 Tripo 的任务号再发出去。

### Migration

- **价格配置：**给用户启用 Tripo 之前，先建模型 `tripo` 并设置 Tripo 积分单价。价格未保存时，请求会被拒绝。
