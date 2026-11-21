---
changelogVersion: 1
plugin: "tripo"
version: "1.0.0"
locale: "en"
translations:
  zh-CN: "CHANGELOG.zh-CN.md"
---
# Changelog

## [1.0.0]

### Added

- Generate and process 3D models with Tripo from the gateway, including text, image, and multiview generation, texture, convert, mesh tools, rigging, and file upload.
- Call Tripo's own paths, such as `POST /v3/generation/text-to-model`, and the same paths under `/tripo`. Poll a task at `GET /v3/tasks/{task_id}` with the gateway task id.
- Pass a gateway task id into a follow-up request such as texture, convert, or rig. The gateway sends Tripo's id upstream.
- `POST /v3/files` forwards the file and lets Tripo accept or reject it by Tripo's own size limits. A model of 64 MB to 150 MB cannot use that path: the gateway stops uploads over `MAX_FILE_DOWNLOAD_MB`, which defaults to 64. Upload those models with `POST /v3/files/presign`.

### Migration

- **Price configuration:** Before enabling Tripo for users, create the model `tripo` and set its Tripo credit unit price. Requests are rejected until that price is saved.
