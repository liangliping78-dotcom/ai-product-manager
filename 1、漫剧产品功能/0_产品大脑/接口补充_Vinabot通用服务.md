# 接口补充_Vinabot通用服务

## 一、文档边界

- 来源：`/Users/laihuaqianhai/Downloads/api-vinabot接口文档.pdf`
- 补充原则：只记录 `0_产品大脑` 里尚未覆盖的 Vinabot 通用服务接口。
- 不重复展开：`animatic/*`、`nine_grid/*`、`voice/*` 等已在 `接口-字段表.md` 中覆盖的主产品接口。
- 当前口径：以 PDF 可见文字为准，不臆造未给出的响应字段或状态枚举。

## 二、环境与总览

### 2.1 环境

| 环境 | 基础域名 |
| --- | --- |
| 测试 | `http://api.vinabot.net/apicenter/test` |
| 正式 | `http://api.vinabot.net/apicenter/release` |

### 2.2 能力总览

| 能力组 | 接口 | 方法 | 作用 |
| --- | --- | --- | --- |
| 增长归因 | `api/appsflyer/event_track` | POST | 通过 Vinabot 中转上报 AppsFlyer 事件 |
| 文本处理 | `api/translation/polish` | POST | 清洗语音识别文本中的语气词和同音字 |
| 文本处理 | `api/translation/correct` | POST | 用术语映射修正源文和译文 |
| 下载回存 | `api/download/create` | POST | 创建第三方资源下载回存任务 |
| 下载回存 | `api/download/check` | GET | 查询下载回存状态与回存结果 |
| 通用图像生成 | `api/generation/image` | POST | 同步图像生成 |
| 通用图像生成 | `api/generation/image/async` | POST | 异步图像生成任务创建 |
| 通用图像生成 | `api/generation/image/result` | GET | 异步图像生成结果查询 |
| 通用视频生成 | `api/generation/video/create` | POST | 创建通用视频生成任务 |
| 通用视频生成 | `api/generation/video/result` | GET | 查询通用视频生成结果 |
| 安全审核 | `api/moderation/image` | POST | 图片敏感内容审核 |
| 语音辅助 | `api/asr/text2token` | POST | 把关键词转成 ASR 唤醒词 token |

## 三、增长归因接口

### 3.1 AppsFlyer 事件上报

**测试地址**

- `http://api.vinabot.net/apicenter/test/api/appsflyer/event_track`

**正式地址**

- `http://api.vinabot.net/apicenter/release/api/appsflyer/event_track`

**请求方式**

- `POST`

**请求头**

| 参数 | 是否必填 | 说明 |
| --- | --- | --- |
| `Appsflyer-Id` | 是 | AppsFlyer 在应用首次启动时生成的唯一标识 |
| `Content-Type` | 是 | `application/json` |

**请求体公共字段**

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `eventName` | 是 | 事件名 |
| `eventValue` | 是 | 事件参数对象 |

**当前明确支持的事件**

| 业务事件 | `eventName` | `eventValue` 必填字段 |
| --- | --- | --- |
| 注册 | `af_complete_registration` | `af_registration_method`、`platform_type` |
| 支付 | `af_purchase` | `af_price`、`af_currency`、`goods_name`、`af_order_id`、`pay_type`、`if_success`、`platform_type` |
| 首次购买 | `first_purchase` | `af_price`、`af_currency`、`af_order_id`、`platform_type` |

**业务补充**

- `platform_type` 当前文档明确举例为 `vinabot_ios`、`vinabot_Android`。
- PDF 同时给了官方 AppsFlyer S2S 文档入口，说明 Vinabot 这里做的是中转封装，而不是重新定义事件体系。
- PDF 未提供统一 JSON 响应体，因此当前只确认其为“服务端事件回传接口”，不补写虚构返回字段。

## 四、文本处理服务

### 4.1 语音识别文本润色

**地址**

- `http://api.vinabot.net/apicenter/test/api/translation/polish`

**请求方式**

- `POST`

**请求体**

```json
{
  "text": "这个报告呃，嗯，需要在下周五之前提交，啊，特别是财务那块儿的数据，对吧",
  "style": "business"
}
```

**返回示例**

```json
{
  "code": 200,
  "result": "这份报告需在下周五前提交，特别是财务数据。"
}
```

**产品意义**

- 该接口适合承接语音输入后的文本清洗，把 ASR 粗文本变成更适合继续翻译、生成或展示的中间文本。

### 4.2 术语修正翻译

**地址**

- `http://api.vinabot.net/apicenter/test/api/translation/correct`

**请求方式**

- `POST`

**请求体**

```json
{
  "src_text": "我们是跑在 inno 平台上的服务",
  "src_lang": "zh",
  "tgt_text": "Our service runs on inno.",
  "tgt_lang": "en",
  "term_map": {
    "音诺": "InnAIO"
  }
}
```

**返回示例**

```json
{
  "code": 200,
  "src_text": "我们是跑在音诺平台上的服务",
  "tgt_text": "Our service runs on InnAIO."
}
```

**业务补充**

- 若 `tgt_text` 不传，则仅按 `term_map` 修复原文。
- 该接口更像“术语对齐服务”，适合解决品牌名、专有名词和翻译一致性问题。

## 五、下载回存服务

### 5.1 创建下载回存任务

**地址**

- `http://api.vinabot.net/apicenter/test/api/download/create`

**请求方式**

- `POST`

**请求体关键字段**

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `url` | 是 | 原始第三方下载地址 |
| `keyname` | 否 | 回存资源 key |
| `direct` | 否 | `true` 时直接返回 `obs_url` |
| `return_first_frame` | 否 | 是否返回首帧 |
| `return_last_frame` | 否 | 是否返回尾帧 |
| `overseas` | 否 | 是否上传到海外资源域 |

**返回示例**

```json
{
  "code": 200,
  "task_id": "c8abedcd-5f26-4be4-83f7-45bec4520354"
}
```

### 5.2 查询下载回存状态

**地址**

- `http://api.vinabot.net/apicenter/release/api/download/check`

**请求方式**

- `GET`

**Query**

- `task_id`

**返回字段重点**

| 字段 | 说明 |
| --- | --- |
| `status` | 任务状态，文档明确有 `pending`、`downloading`、`uploading`、`done`、`error` |
| `url` | 原始第三方结果地址 |
| `obs_url` | 回存到平台资源域后的地址 |
| `first_frame_url` | 首帧图地址 |
| `last_frame_url` | 尾帧图地址 |
| `keyname` | 回存资源 key |
| `overseas` | 是否走海外资源域 |

**产品意义**

- 该能力不是普通“下载按钮”，而是把第三方结果统一回存到来画资源域，并可顺带产出首尾帧，适合作为通用资产入库与结果托管层。

## 六、通用图像生成服务

### 6.1 同步图像生成

**地址**

- `http://api.vinabot.net/apicenter/test/api/generation/image`

**请求方式**

- `POST`

**请求体关键字段**

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `model` | 否 | 默认 `doubao-seedream-5.0-lite` |
| `prompt` | 是 | 提示词 |
| `ratio` | 否 | 如 `1:1`、`16:9`、`9:16`、`21:9` |
| `resolution` | 否 | 按模型支持范围选择 |
| `image_urls` | 否 | 参考图列表，最多 10 张，单张不超过 10MB |
| `n` | 否 | 组图数量，仅部分模型支持 |
| `overseas` | 否 | 是否上传到海外 |

**返回示例**

```json
{
  "code": 200,
  "error": null,
  "urls": [
    "https://resources.laihua.com/2026-03-04/6e9fc690-2d58-401f-975d-6f15b8aad03d.jpeg"
  ]
}
```

**状态码**

| `code` | 含义 |
| --- | --- |
| `200` | 成功 |
| `400` | 失败 |
| `401` | 敏感内容 |
| `402` | 图片太小 |
| `403` | 图片格式有问题 |

### 6.2 异步图像生成任务创建

**地址**

- `http://api.vinabot.net/apicenter/test/api/generation/image/async`

**请求方式**

- `POST`

**返回示例**

```json
{
  "task_id": "",
  "code": 200,
  "description": "生成中"
}
```

### 6.3 异步图像生成结果查询

**地址**

- `http://api.vinabot.net/apicenter/test/api/generation/image/result`

**请求方式**

- `GET`

**Query**

- `task_id`

**返回示例**

```json
{
  "task_id": "",
  "code": 200,
  "description": "已完成",
  "urls": [
    ""
  ]
}
```

**查询状态码**

| `code` | 含义 |
| --- | --- |
| `200` | 已完成 |
| `201` | 生成中 |
| `400` | 查询失败 |
| `404` | 任务不存在 |

**当前可确认的模型口径**

| 模型 | 当前可确认分辨率 | 当前可确认说明 |
| --- | --- | --- |
| `doubao-seedream-5.0-lite` | `2K`、`3K` | 支持多参考图输入 |
| `doubao-seedream-4.5` | `2K`、`4K` | 支持多参考图输入 |
| `doubao-seedream-4.0` | `1K`、`2K`、`4K` | 支持多参考图输入 |
| `gemini-3.1-flash-image-preview` | `1K`、`2K`、`4K` | 最多 6 张参考图 |
| `gemini-3-pro-image-preview` | `1K`、`2K`、`4K` | 最多 6 张参考图 |

## 七、通用视频生成服务

### 7.1 创建视频生成任务

**地址**

- `http://api.vinabot.net/apicenter/test/api/generation/video/create`

**请求方式**

- `POST`

**请求体关键字段**

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `model` | 否 | 默认 `doubao-seedance-1-5-pro` |
| `prompt` | 是 | 提示词 |
| `ref_image` | 否 | 参考图，PDF 标注部分模型最多支持 4 张 |
| `ref_video` | 否 | 为 2.0 模型预留 |
| `ref_audio` | 否 | 为 2.0 模型预留 |
| `keyframe` | 否 | 首尾帧，与 `ref_image` 二选一 |
| `resolution` | 否 | 如 `480p`、`720p`、`1080p` |
| `ratio` | 否 | 如 `16:9`、`9:16`、`21:9`、`adaptive` |
| `duration` | 否 | 文档给出 `2~12` 秒或 `4~15` 秒等模型差异范围 |
| `return_first_frame` | 否 | 是否返回首帧 |
| `return_last_frame` | 否 | 是否返回尾帧 |
| `overseas` | 否 | 是否上传到海外 |

**返回示例**

```json
{
  "code": 200,
  "task_id": "****",
  "error": ""
}
```

**当前可确认的模型能力收敛**

| 模型族 | 当前可确认能力 |
| --- | --- |
| `doubao-seedance-1-5-pro` / `1-0-pro` | 更偏文生视频 / 关键帧视频，支持 `480p`、`720p`、`1080p` |
| `doubao-seedance-1-0-lite-i2v` | 偏图生视频，PDF 标注最多 4 张参考图 |
| `doubao-seedance-2-0` | 支持更多参考图与更长时长范围 |
| `veo-3.1-*` | 更偏标准化比例和固定时长档位，PDF 中同时出现预览版与正式版命名 |

### 7.2 查询视频生成结果

**地址**

- `http://api.vinabot.net/apicenter/test/api/generation/video/result`

**请求方式**

- `GET`

**Query**

- `task_id`

**返回字段重点**

| 字段 | 说明 |
| --- | --- |
| `status` | 任务状态 |
| `video_url` | 最终视频地址 |
| `first_frame_url` | 首帧图地址 |
| `last_frame_url` | 尾帧图地址 |
| `error` | 错误信息 |

**产品意义**

- 这组接口说明平台除主产品 `animatic/*` 之外，还保留了一层更通用的图像/视频生成服务，可被不同前台产品复用。

## 八、安全与语音辅助

### 8.1 图片审核

**地址**

- `http://api.vinabot.net/apicenter/test/api/moderation/image`

**请求方式**

- `POST`

**请求体**

```json
{
  "image_url": "https://resources.laihua.com/2026-3-30/266d27db-be40-4df5-9014-f8bf3426752f.jpg"
}
```

**返回示例**

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "is_sensitive": true,
    "reason": "Contains sexually suggestive revealing content"
  }
}
```

### 8.2 唤醒词 token 生成

**地址**

- `http://api.vinabot.net/apicenter/test/api/asr/text2token`

**请求方式**

- `POST`

**请求体**

```json
{
  "keywords": [
    "LIGHT UP",
    "法国"
  ]
}
```

**返回示例**

```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {
      "keyword": "LIGHT UP",
      "tag": "@LIGHT_UP",
      "tokens": "L AY1 T AH1 P @LIGHT_UP"
    },
    {
      "keyword": "法国",
      "tag": "@法国",
      "tokens": "f ǎ g uó @法国"
    }
  ]
}
```

**错误码**

| `code` | `msg` |
| --- | --- |
| `400` | `keywords 不能为空` |
| `503` | `模型文件缺失 或 sherpa-onnx-cli 未找到` |
| `500` | `输出行数错误` |

## 九、对产品脑图的新增结论

- `Vinabot` 已经不只是主产品接口转发层，而是开始承接增长归因、语言处理、下载回存、通用生成、安全审核与语音辅助等共享服务。
- 对 AI 漫剧平台而言，这意味着前台工作流之外已经存在可复用的服务底座，后续新功能不一定都要从 `animatic/*` 主链路重新搭一遍。
- 这些接口是“平台通用能力补充”，不等于它们都已前台化为 AI 漫剧页面入口；当前能确认的是底层服务已存在并可被不同产品复用。
