---
changelogVersion: 1
plugin: "tripo"
version: "1.0.1"
locale: "en"
translations:
  zh-CN: "CHANGELOG.zh-CN.md"
---
# Changelog

## [1.0.1]

### Changed

- `POST /v3/files` no longer rejects an image over 20 MB. Tripo accepts or rejects the file by its own size limits.
- A model file of 64 MB to 150 MB still cannot use `POST /v3/files`. The gateway stops any upload over `MAX_FILE_DOWNLOAD_MB`, which defaults to 64, before Tripo sees it. Upload those models with `POST /v3/files/presign`.
