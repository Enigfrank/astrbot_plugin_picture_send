# astrbot_plugin_picture_send

`astrbot_plugin_picture_send` 是一个面向 AstrBot 的企业微信微信客服（wecom）插件，用于在用户发送作业查询指令后自动拉取并发送本地作业图片。插件支持请求统计、企业微信昵称查询、可选图片压缩、请求日志脱敏以及错误转发通知。

## 项目信息

| 项目 | 内容 |
| --- | --- |
| 插件标识 | `astrbot_plugin_homework_send` |
| 注册名称 | `picture-send` |
| 展示名称 | 发送作业 |
| 当前版本 | `v1.5.2` |
| 作者 | `Enigfrank` |
| 支持平台 | 企业微信微信客服（`wecom`） |
| 仓库地址 | <https://github.com/Enigfrank/astrbot_plugin_picture_send> |
| 许可证 | GNU AGPL-3.0 |

## 功能特性

- 通过 `/homework` 指令发送作业图片。
- 仅允许在企业微信微信客服（`wecom`）平台使用，其他平台会直接提示不支持。
- 自动扫描 `/AstrBot/data/homework` 目录下的图片文件。
- 支持 `.png`、`.jpg`、`.jpeg`、`.webp` 图片格式。
- 支持按文件名排序后依次发送多张作业图片。
- 可调用一言接口，在拉取作业图片前返回一条提示文本。
- 可通过企业微信微信客服接口查询用户昵称。
- 支持 access_token 缓存与昵称缓存，降低企业微信 API 调用频率。
- 支持持久化记录用户请求次数和最近请求时间。
- 支持 `/homework_stats` 查看作业请求统计。
- 支持 `/userid` 查询指定企业微信用户 ID 对应的微信昵称。
- 支持可选图片压缩，依赖 Pillow。
- 支持日志输出和用户 ID 脱敏。
- 支持插件异常时主动转发错误信息给管理员。

## 目录结构

```text
picture-send/
├── _conf_schema.json       # AstrBot 插件配置 Schema
├── config.py               # 插件常量与默认配置
├── http_client.py          # HTTP 请求封装
├── image_compressor.py     # 图片压缩工具
├── main.py                 # AstrBot 插件入口与命令处理
├── metadata.yaml           # 插件元数据
├── stats_manager.py        # 请求统计持久化管理
├── utils.py                # 通用工具函数
├── wecom_client.py         # 企业微信 API 客户端
├── LICENSE                 # GNU AGPL-3.0 许可证
└── .gitignore
```

## 运行环境

- Python 3.10 或更高版本。
- AstrBot 运行环境。
- 已接入企业微信微信客服（wecom）平台。
- 可访问以下外部接口：
  - `https://v1.hitokoto.cn/`
  - `https://qyapi.weixin.qq.com/cgi-bin/gettoken`
  - `https://qyapi.weixin.qq.com/cgi-bin/kf/customer/batchget`

### 可选依赖

如果需要启用图片压缩，请在 AstrBot 所使用的 Python 环境中安装 Pillow：

```bash
pip install Pillow
```

未安装 Pillow 时，即使在配置中启用了图片压缩，插件也会跳过压缩并原样发送图片。

## 安装方式

### 方式一：通过 AstrBot 插件管理安装

在 AstrBot 插件管理中使用仓库地址安装：

```text
https://github.com/Enigfrank/astrbot_plugin_picture_send
```

### 方式二：手动安装

将本项目目录放入 AstrBot 的插件目录中，并确保目录内包含 `main.py` 与 `metadata.yaml` 等文件。放置完成后重启 AstrBot 或在 AstrBot 管理面板中重新加载插件。

## 作业图片放置位置

插件默认从以下目录读取作业图片：

```text
/AstrBot/data/homework
```

请将需要发送的作业图片放入该目录，支持的文件后缀为：

```text
.png
.jpg
.jpeg
.webp
```

插件会按文件名排序后依次发送。因此建议使用带序号的文件名，例如：

```text
01.png
02.jpg
03.webp
```

如果目录不存在、目录为空或没有支持格式的图片，用户执行 `/homework` 时会收到未找到图片的提示。

## 插件配置

插件配置由 `_conf_schema.json` 定义，可在 AstrBot 插件配置界面中填写。

### 企业微信配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `wecom_corp_id` | string | 空 | 企业微信 CorpID，用于获取 access_token。 |
| `wecom_kf_secret` | string | 空 | 微信客服 Secret，用于获取 access_token 并调用微信客服接口。 |
| `enable_wecom_name_lookup` | bool | `true` | 是否启用微信客服昵称查询。 |

如果未填写 `wecom_corp_id` 或 `wecom_kf_secret`，插件仍可发送作业图片，但无法通过企业微信 API 查询用户昵称。

### 统计存储配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `stats_storage.enabled` | bool | `true` | 是否启用持久化统计。 |
| `stats_storage.file_name` | string | `homework_stats.json` | 统计数据文件名。 |
| `stats_storage.keep_latest_records` | int | `0` | 每个用户保留的最近请求时间条数，`0` 表示不限制。 |

统计文件默认保存在 AstrBot 数据目录下的插件数据目录中：

```text
<AstrBot data>/plugin_data/<plugin_name>/homework_stats.json
```

### 日志配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `log_settings.enabled` | bool | `true` | 是否输出作业请求日志。 |
| `log_settings.mask_user_id` | bool | `true` | 是否对用户 ID 做脱敏。 |
| `log_settings.template` | string | 内置模板 | 日志模板。 |

日志模板支持以下变量：

| 变量 | 说明 |
| --- | --- |
| `{user_id}` | 用户 ID，可能已脱敏。 |
| `{user_name}` | 用户昵称或事件中的发送者名称。 |
| `{api}` | 昵称查询是否成功，值为 `ok` 或 `fail`。 |
| `{platform}` | 平台名称。 |
| `{count}` | 当前用户累计请求次数。 |
| `{time}` | 请求时间。 |

### 图片压缩配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `image_compression.enabled` | bool | `false` | 是否启用图片压缩。 |
| `image_compression.quality` | int | `85` | 压缩质量，范围为 `1` 到 `100`。 |
| `image_compression.max_width` | int | `1920` | 最大宽度，超过后等比缩放，`0` 表示不限制。 |
| `image_compression.max_height` | int | `1080` | 最大高度，超过后等比缩放，`0` 表示不限制。 |

压缩后的图片会临时写入 AstrBot 临时目录下的 `homework_compress` 目录。发送完成后，插件会尝试删除临时文件。

### 错误转发配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `error_forward.enabled` | bool | `false` | 是否启用错误转发。 |
| `error_forward.target_uid` | string | 空 | 接收错误信息的目标用户 UID。 |
| `error_forward.include_traceback` | bool | `true` | 是否包含完整 traceback。 |
| `error_forward.notify_user` | bool | `true` | 命令处理报错时是否提示当前用户已通知管理员。 |
| `error_forward.max_length` | int | `600` | 错误消息最大长度，超出后截断，`0` 表示不限制。 |

`target_uid` 可以填写纯用户 UID，插件会自动拼接为：

```text
wecom:friend:{uid}
```

如果已填写完整会话标识，例如包含多个冒号的 session 字符串，插件会直接使用该值。

## 配置示例

```json
{
  "wecom_corp_id": "your_corpid",
  "wecom_kf_secret": "your_kf_secret",
  "enable_wecom_name_lookup": true,
  "stats_storage": {
    "enabled": true,
    "file_name": "homework_stats.json",
    "keep_latest_records": 0
  },
  "log_settings": {
    "enabled": true,
    "mask_user_id": true,
    "template": "作业请求 | 微信ID={user_id} | 微信昵称={user_name} [api={api}] | 平台={platform} | 计数={count} | 时间={time}"
  },
  "image_compression": {
    "enabled": false,
    "quality": 85,
    "max_width": 1920,
    "max_height": 1080
  },
  "error_forward": {
    "enabled": false,
    "target_uid": "",
    "include_traceback": true,
    "notify_user": true,
    "max_length": 600
  }
}
```

## 使用指令

### 发送作业图片

```text
/homework
```

执行流程：

1. 检查当前平台是否为 `wecom`。
2. 尝试获取一言提示文本。
3. 扫描 `/AstrBot/data/homework` 目录中的图片。
4. 获取用户 ID 与用户名称。
5. 根据配置查询企业微信昵称。
6. 记录请求统计并输出日志。
7. 按文件名顺序发送所有作业图片。

### 查询用户昵称

```text
/userid {用户微信ID}
```

示例：

```text
/userid wm_xxxxxxxxxxxxx
```

该指令会调用企业微信微信客服接口查询指定用户 ID 对应的微信昵称。仅支持 `wecom` 平台。

### 查看请求统计

```text
/homework_stats
```

该指令会返回：

- 总请求次数。
- 用户数量。
- 各用户累计请求次数。

## 数据与缓存说明

### access_token 缓存

企业微信 access_token 会在内存中缓存，并在过期前提前刷新，避免临界时间请求失败。

### 昵称缓存

用户昵称会在内存中缓存，默认有效期为 1 小时。缓存仅在插件运行期间有效，插件重启后会重新查询。

### 统计数据

统计数据会持久化保存为 JSON 文件。每次执行 `/homework` 且统计功能启用时，插件会记录：

- 用户 ID。
- 用户名称。
- 平台名称。
- 用户累计请求次数。
- 请求时间。
- 最近请求时间列表。

## 常见问题

### 执行 `/homework` 提示未找到图片

请检查：

1. `/AstrBot/data/homework` 目录是否存在。
2. 目录中是否有 `.png`、`.jpg`、`.jpeg`、`.webp` 文件。
3. AstrBot 进程是否有权限读取该目录。

### 企业微信昵称查询失败

请检查：

1. `wecom_corp_id` 是否正确。
2. `wecom_kf_secret` 是否正确。
3. 当前用户 ID 是否属于微信客服可查询的外部联系人。
4. AstrBot 所在环境是否能访问企业微信 API。

### 图片压缩没有生效

请检查：

1. 是否安装了 Pillow。
2. `image_compression.enabled` 是否为 `true`。
3. 图片尺寸是否超过 `max_width` 或 `max_height`。
4. `quality` 是否小于 `100`。

### 非企业微信平台无法使用

该插件在代码中限制仅支持 `wecom` 平台。如果在其他平台执行指令，会返回“本插件仅支持企业微信(wecom)”。

## 开发说明

项目采用按职责拆分的轻量模块结构：

- `main.py` 负责 AstrBot 生命周期、命令注册和业务编排。
- `wecom_client.py` 负责企业微信 access_token、昵称查询与缓存。
- `http_client.py` 负责 HTTP GET/POST JSON 请求。
- `stats_manager.py` 负责统计数据加载、保存和汇总。
- `image_compressor.py` 负责图片压缩与临时文件生成。
- `utils.py` 负责文本清理、用户 ID 脱敏和时间格式化。
- `config.py` 负责集中维护常量和默认值。

## 许可证

本项目基于 GNU Affero General Public License v3.0 发布。详情请查看 [LICENSE](./LICENSE)。
