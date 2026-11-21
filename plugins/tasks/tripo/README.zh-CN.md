---
locale: zh-CN
---
# Tripo

用 Tripo 生成和加工三维模型，并上传素材文件。

## 渠道设置

- 渠道类型选任务插件 (61)，任务插件选 Tripo。
- 一个渠道放一把 Tripo API Key。要分散请求，就每个渠道各放一把。
- API 地址留空时使用 `https://openapi.tripo3d.ai/v3`。走 `.com` 时填写 `https://openapi.tripo3d.com/v3`。地址要带 `/v3`，末尾不要再加斜杠。
- 一个渠道只有一个上游地址。`.ai` 和 `.com` 要同时用，就建两条 Tripo 渠道，各自填对应地址。
- 如果上游是另一台 New API 网关，渠道类型选 New API (60)，填写该网关的地址和它的令牌。上游网关也需要安装 Tripo 插件。

## 模型与接口

网关模型名是 `tripo`。后台只建这一个模型，价格也配在它上面。H 系列、P 系列的名字写在请求正文里，原样转给 Tripo。日志和账单里看到的是 `tripo`。

客户端调用 Tripo 自己的路径。这些路径加上 `/tripo` 前缀也能调用，例如 `/tripo/v3/generation/text-to-model`。

| 接口 | 用途 |
| --- | --- |
| `POST /v3/generation/text-to-model` | 文生三维 |
| `POST /v3/generation/image-to-model` | 图生三维 |
| `POST /v3/generation/multiview-to-model` | 多视图生三维 |
| `POST /v3/generation/image-to-splat` | 图生高斯溅射 |
| `POST /v3/generation/text-to-image` | 文生图 |
| `POST /v3/generation/image-to-image` | 图生图 |
| `POST /v3/generation/image-to-multiview` | 图生多视图 |
| `POST /v3/generation/edit-multiview` | 编辑多视图 |
| `POST /v3/generation/image-to-depth` | 图生深度 |
| `POST /v3/models/texture` | 给模型贴图 |
| `POST /v3/models/convert` | 转换模型 |
| `POST /v3/models/import` | 导入模型 |
| `POST /v3/mesh/segment` | 分割网格 |
| `POST /v3/mesh/smartsegment` | 智能分割网格 |
| `POST /v3/mesh/complete` | 补全网格 |
| `POST /v3/mesh/decimate` | 减面 |
| `POST /v3/mesh/uv` | 生成 UV |
| `POST /v3/animations/rig-check` | 检查模型能否绑骨 |
| `POST /v3/animations/rig` | 绑骨 |
| `POST /v3/animations/retarget` | 重定向动画 |
| `GET /v3/tasks/{task_id}` | 查询一个任务 |
| `POST /v3/tasks/list` | 一次查询最多 100 个任务 |
| `POST /v3/files` | 上传文件 |
| `POST /v3/files/presign` | 申请直传地址 |

响应保持 Tripo 的 `{code, data}` 外形。用户看到的 `task_id` 是网关号，`task_` 后面 32 位。贴图、转格式、绑骨这类后续请求把这个网关号传回来，网关在发出去之前换成 Tripo 的号。直接填 Tripo 的 UUID，网关不会把它当成自己的任务。

用 `GET /v3/tasks/{task_id}` 轮询。完成后，模型或图片从 `output` 里的临时地址下载，不经过网关。

## 计费

「Tripo 积分单价」按 Tripo 积分计费，配在模型 `tripo` 上。

请求进来时，插件按请求内容估计要用的积分并先占住。看不清选项时先占 105 个积分。任务完成后，按 Tripo 返回的 `credits_consumed` 结算。失败、取消、封禁或过期的任务按 0 结算。上传文件和申请直传地址占 0。

模型 `tripo` 没有保存价格时，请求在到达 Tripo 之前被拒绝。

## 示例

```bash
curl https://your-gateway.example.com/v3/generation/text-to-model \
  -H "Authorization: Bearer sk-..." \
  -H "Content-Type: application/json" \
  -d '{"model": "v3.1-20260211", "prompt": "a wooden chair"}'
```

用返回的 `task_id` 轮询：

```bash
curl https://your-gateway.example.com/v3/tasks/{task_id} \
  -H "Authorization: Bearer sk-..."
```

## 限制

- `GET /v3/account/balance`、`GET /v3/account/usage`、`GET /v3/account/usage/history` 没有接入。
- 经 `POST /v3/files` 上传的图片须为 JPEG 或 PNG，且不超过 20 MB。
- 经网关上传的文件还受 `MAX_FILE_DOWNLOAD_MB` 限制，默认 64 MB。更大的模型文件改走 `POST /v3/files/presign`，把文件直接传到返回的地址，这一步不经过网关。
- `POST /v3/tasks/list` 接受 1 到 100 个网关任务号。Tripo 的 UUID、空列表或不认识的号会被拒绝。
- 不使用 Tripo 的 Webhook。请轮询网关任务号。
