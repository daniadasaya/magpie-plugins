# Magpie 自动上下文压缩

Magpie 的 Provider 插件。对话历史超过设定条数时，自动压缩上下文，让长对话继续进行。

支持 OpenAI Responses、Chat Completions 和 Anthropic Messages。

## 安装

在插件目录运行：

```powershell
magpie plugin add "$PWD"
```

## 使用

1. 编辑 [`examples/options.jsonc`](examples/options.jsonc)，填入接口地址、API 密钥和模型 ID。
2. 在 PowerShell 7 或更高版本中，运行下面的命令导入配置。
3. 在 Magpie 中选择模型，例如 `responses-main/coder`。

自用配置见 [`examples/selfuse.jsonc`](examples/selfuse.jsonc)。使用时，将命令中的配置路径换成该文件。

```powershell
$options = Get-Content -Raw -Encoding utf8 examples/options.jsonc |
  ConvertFrom-Json -ErrorAction Stop |
  ConvertTo-Json -Depth 100 -Compress
magpie plugin options magpie-autocompact $options
magpie models
```

修改配置后，重新运行导入命令。

## 配置

示例默认启用 `responses-main`，其他供应商按需启用。所有供应商都放在 `providers` 中。

常用选项：

- `protocol`：接口类型，可选 `responses`、`chat`、`anthropic`。
- `models`：模型列表和别名。省略时尝试从接口获取。
- `maxMessages`：历史超过多少条时压缩。省略或设为 `0` 时不压缩。
- `compactionTargetRatio`：压缩后希望保留的消息数占 `maxMessages` 的比例，大于 `0` 且不超过 `1`，默认 `0.5`。安全边界或必须保留的内容可能使实际结果高于目标。
- `maxRetainedRatio`：可选，大于 `0` 且不超过 `1`。安全切分无法将保留项数降到 `maxMessages` 的此比例时，允许强制压缩、摘要或打包到最后一项，包括当前输入和未完成的工具调用。强制摘要与打包附加继续执行的提示；打包仍保留加密项，无法保证压到此比例。省略此项则不启用强制压缩。
- `compactionNotice`：设为 `true` 时按实际使用的压缩方式及保留消息数显示提示；默认关闭。
- `diagnostics`：设为 `true` 时记录请求结构和压缩诊断，默认关闭。日志写入该供应商的 `stateDir/diagnostics.jsonl`；默认 `stateDir` 为 `~/.local/state/magpie-autocompact/<供应商 ID>`，也可由 `XDG_STATE_HOME` 或 `stateDir` 指定。
- `timeoutMs`：单事件静默超时，默认 `120000` 毫秒（2 分钟）；限制请求到首个上游事件及流式事件之间的等待时间，不限制整个请求总时长。压缩、摘要和推理分别应用其对应 timeout；`0` 表示不限。

流式请求在压缩或摘要上游产生进度事件时会收到空 delta 保活；空 delta 不延长上游事件超时。连续的空文本消息会在计数和转发前移除，不占用 `maxMessages`。

更多选项见[配置示例中的注释](examples/options.jsonc)。

压缩依次尝试原生 trigger、`/responses/compact`、`summary` 和 `pack`；非 Responses 协议只尝试后两者。

- `summary`：未指定 `summary.provider` 或引用自身时，加密历史（包括已有压缩前缀）以原生输入发送给同一供应商，不把密文塞进 JSON 文本。摘要模型仍须支持这些输入；其他供应商不能解读绑定在原供应商上的加密内容。跨供应商摘要命中加密缓存时，可使用客户端重放的原始明文历史。
- `pack`：只合并历史中相邻的明文内容；加密、签名及其他不透明项原样保留，位置不变，不跨越它们合并。跨越不透明项的工具调用批次也保持原样。支持的媒体仍作为原生内容块传递；仅靠打包不能保证减少 token，也可能无法降到消息数上限，此时明确返回压缩失败。

两者均支持复用已压缩的历史前缀；同供应商摘要和打包无需为了加密前缀重新处理原始历史。前缀缓存持久化于 `stateDir`，并按供应商、凭据、模型及相关配置隔离。

诊断日志按 JSON Lines 记录请求 ID、会话标识的哈希、输入项类型、角色、文本长度和是否为空、缓存命中的前缀长度、压缩前后项数及推理结果状态；不记录消息正文、工具参数、媒体内容、请求头或密钥。每个输入列表最多记录前 2000 项，其余数量写在 `omitted`。日志会持续追加，排查后可关闭 `diagnostics` 并按需删除该日志文件。

配置文件带有注释，请用上面的命令解析后导入。不要将真实密钥提交到版本库。

## 开发

需要 Node.js 22.13 或更高版本。

```sh
npm test
```

安装 Magpie 后，可运行集成测试：

```sh
npm run test:integration
```

## LICENSE

本项目使用 MIT License，详见 [`LICENSE`](LICENSE)。

## 致谢

感谢 Magpie 项目及所有为本项目提供反馈和贡献的使用者与维护者。
