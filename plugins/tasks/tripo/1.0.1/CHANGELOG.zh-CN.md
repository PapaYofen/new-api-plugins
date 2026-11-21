---
changelogVersion: 1
plugin: "tripo"
version: "1.0.1"
locale: "zh-CN"
---
# Changelog

## [1.0.1]

### Changed

- `POST /v3/files` 不再拒绝超过 20 MB 的图片。文件大小由 Tripo 按自己的限制接受或拒绝。
- 64 MB 到 150 MB 的模型文件仍然不能走 `POST /v3/files`。网关会在 Tripo 看到文件之前拦住超过 `MAX_FILE_DOWNLOAD_MB` 的上传，默认是 64。这些模型请改用 `POST /v3/files/presign`。
