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
- `POST /v3/files` 会把文件转给 Tripo，由 Tripo 按自己的大小限制接受或拒绝。64 MB 到 150 MB 的模型不能走这条路径：网关会拦住超过 `MAX_FILE_DOWNLOAD_MB` 的上传，默认是 64。这些模型请用 `POST /v3/files/presign`。

### Migration

- **价格配置：**给用户启用 Tripo 之前，先建模型 `tripo` 并设置 Tripo 积分单价。价格未保存时，请求会被拒绝。
