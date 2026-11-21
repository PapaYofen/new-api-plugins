---
translations:
  zh-CN: README.zh-CN.md
---
# Tripo

Generates and processes 3D models with Tripo, and uploads source files.

## Channel setup

- Choose the Task Plugin (61) channel type and bind Tripo.
- Put one Tripo API key on the channel. To spread requests across keys, create one channel per key.
- Leave the API address empty to use `https://openapi.tripo3d.ai/v3`. For the `.com` endpoint, set `https://openapi.tripo3d.com/v3`. Keep the `/v3` suffix and do not add a trailing slash.
- A channel has one upstream address. To use `.ai` and `.com` together, create two Tripo channels and set one address on each.
- To use another New API gateway as the upstream, choose New API (60) and enter that gateway's address and one of its tokens. That gateway needs the Tripo plugin too.

## Models and endpoints

The gateway model is `tripo`. Create that one model and attach the price to it. H-series and P-series names stay in the request body and are forwarded to Tripo. Logs and bills show `tripo`.

Clients call Tripo's own paths. The same paths also work under `/tripo`, such as `/tripo/v3/generation/text-to-model`.

| Endpoint | What it does |
| --- | --- |
| `POST /v3/generation/text-to-model` | Text to 3D |
| `POST /v3/generation/image-to-model` | Image to 3D |
| `POST /v3/generation/multiview-to-model` | Multiview images to 3D |
| `POST /v3/generation/image-to-splat` | Image to a Gaussian splat |
| `POST /v3/generation/text-to-image` | Text to image |
| `POST /v3/generation/image-to-image` | Image to image |
| `POST /v3/generation/image-to-multiview` | Image to multiview images |
| `POST /v3/generation/edit-multiview` | Edit multiview images |
| `POST /v3/generation/image-to-depth` | Image to a depth map |
| `POST /v3/models/texture` | Texture a model |
| `POST /v3/models/convert` | Convert a model |
| `POST /v3/models/import` | Import a model |
| `POST /v3/mesh/segment` | Segment a mesh |
| `POST /v3/mesh/smartsegment` | Smart-segment a mesh |
| `POST /v3/mesh/complete` | Complete a mesh |
| `POST /v3/mesh/decimate` | Decimate a mesh |
| `POST /v3/mesh/uv` | Generate UVs |
| `POST /v3/animations/rig-check` | Check whether a model can be rigged |
| `POST /v3/animations/rig` | Rig a model |
| `POST /v3/animations/retarget` | Retarget an animation |
| `GET /v3/tasks/{task_id}` | Read one task |
| `POST /v3/tasks/list` | Read up to 100 tasks |
| `POST /v3/files` | Upload a file |
| `POST /v3/files/presign` | Request a direct upload URL |

Replies keep Tripo's `{code, data}` shape. The `task_id` the client sees is the gateway id, `task_` plus 32 characters. Follow-up calls such as texture, convert, and rig pass that gateway id back. The gateway replaces it with Tripo's id before the request leaves. A raw Tripo UUID is not treated as a gateway task.

Poll `GET /v3/tasks/{task_id}`. A finished model or image is downloaded from the temporary URL in `output`, not through the gateway.

## Billing

"Tripo credit unit price" is charged per Tripo credit on the model `tripo`.

Before the call, the plugin estimates the credits from the request and reserves that many. Options it cannot price reserve 105 credits. When the task finishes, settlement uses the `credits_consumed` Tripo reports. A failed, cancelled, banned, or expired task settles at 0. File upload and presign reserve 0.

A request with no saved price for `tripo` is rejected before it reaches Tripo.

## Example

```bash
curl https://your-gateway.example.com/v3/generation/text-to-model \
  -H "Authorization: Bearer sk-..." \
  -H "Content-Type: application/json" \
  -d '{"model": "v3.1-20260211", "prompt": "a wooden chair"}'
```

Poll with the `task_id` from the reply:

```bash
curl https://your-gateway.example.com/v3/tasks/{task_id} \
  -H "Authorization: Bearer sk-..."
```

## Limits

- `GET /v3/account/balance`, `GET /v3/account/usage`, and `GET /v3/account/usage/history` are not served.
- An image sent through `POST /v3/files` must be JPEG or PNG and at most 20 MB.
- A file uploaded through the gateway is also limited by `MAX_FILE_DOWNLOAD_MB`, which defaults to 64. Larger model files should use `POST /v3/files/presign` and upload the bytes to the returned URL, which does not pass through the gateway.
- `POST /v3/tasks/list` accepts 1 to 100 gateway task ids. A Tripo UUID, an empty list, or an unknown id is rejected.
- Tripo webhooks are not used. Poll the gateway task id.
