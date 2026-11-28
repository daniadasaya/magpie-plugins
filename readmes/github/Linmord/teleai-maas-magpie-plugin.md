# magpie-plugin-teleai-maas

把中国电信「天翼星辰MaaS」（天翼 AI 开放平台）接进 magpie / OpenCode 的供应商插件。

- 接口文档：<https://www.teleai.com.cn/doc/cQo5DXMRNoHECdaPasOv/hjQ3zhABy0YZFlT3o42b>
- 端点：`https://api-maas.teleai.com.cn/v1`（OpenAI 兼容 Chat Completions）
- 鉴权：API Key（天翼 AI 控制台 → 应用及API Key）。点一次「浏览器登录控制台」会自动把这个账号已有的 Key 领回来，也可以自己贴

重复执行 `magpie plugin login teleai-maas` 会多出一个 `teleai-maas#xxxx` 账号，不是替换旧 key。

## 模型

| 模型 | 思考 | 工具调用 | 结构化输出 |
| --- | --- | --- | --- |
| Xing4.1-Preview | 默认开，可关 | 支持 | json_object / json_schema |
| Xing4.0-29B-A4B | 无 | 支持 | json_object |
| TeleChat3-438B-A24B | 无 | 支持 | json_object / json_schema |

模型 id 用小写（与平台 `/v1/models` 一致），发请求时自动转成文档写法。上下文与最大输出按 131072 / 32768 估算，文档未写明。

## 安装

```sh
# 从本仓库装（也可以给一个本地目录的绝对路径）
magpie plugin add github:Linmord/teleai-maas-magpie-plugin
magpie plugin login teleai-maas
# 选一种登录方式，然后完全退出 magpie 再启动
```

三种登录方式（供应商设置 → 天翼星辰MaaS）：

| 方式 | 拿到 Key | 显示额度 | 说明 |
| --- | --- | --- | --- |
| API key only | ✅ | ❌ | 最省事，面板里看不到余量 |
| API key + console sign-in | ✅（没填就自动领） | ✅ | 开个专用浏览器登控制台，插件读 Cookie |
| API key + paste cookie | ✅（没填就自动领） | ✅ | 自己贴 `operToken=…`，只要这一段 |

```sh
magpie plugin list          # 列出的是插件目录（…\teleai-maas-magpie-plugin  on）
magpie providers            # teleai-maas  signed in as API key …xxxx
```
## 额度显示

电信控制台有真额度接口，插件直接读它，每个模型一行余量：

```sh
magpie quota teleai-maas      # 最简单的一条
magpie statusline             # 状态栏那一行（网关会缓存，可能慢一拍；要看准数用上一条）
```

不想开界面、也不想经 magpie 的话，还有两条：

```sh
node quota.mjs                # 插件目录里的独立脚本，直接问控制台（--json 给脚本用）
curl http://127.0.0.1:11262/v1/magpie/quotas   # 网关的 HTTP 接口，没有则用 magpie status 里的端口
```

```text
teleai-maas · 免费资源包（控制台） · API key …xxxx  subscription
  Xing4.0-29B-A4B      0% (99.9994 / 100 百万Tokens)
  TeleChat3-438B-A24B  0% (100 / 100 百万Tokens)
  Xing4.0-29B-Intent   0% (100 / 100 百万Tokens)
  Xing4.1-Preview      0% (99.9952 / 100 百万Tokens · 付费 1,000 / 1,000)
```

每个模型的免费包是**各自 100 百万 Tokens**（不是总共十亿）；`Xing4.1-Preview` 还多一份 1000 的付费包，也会一起显示。

上面这两条命令每次都去控制台取真数。magpie **界面里那张卡片显示的是宿主自己的快照**（`~/.config/magpie/quotas.json`，带时间戳），只有它去刷新时才重新问插件——所以发完消息数字没动是正常的，点一下刷新（或跑一次 `magpie quota`）就对上了。插件这边不做本地缓存。

### 三种登录方式

供应商设置 → 天翼星辰MaaS → 登录方式里三选一：

| 方式 | 拿到什么 | 什么时候用 |
| --- | --- | --- |
| **API Key** | 只有对话凭证，额度栏直接说「未登录控制台」 | 只想赶紧聊天 |
| **API Key + 浏览器登录控制台** | Key（那栏可以留空，登录后自动从控制台领回）+ 真实资源包余量 | **推荐**，一次点完 |
| **API Key + 手动填 Cookie** | 同上，Cookie 自己贴进去 | 浏览器起不来、想用自己日常那个浏览器、或者在没图形界面的机器上 |

命令行等价：`magpie plugin login teleai-maas` 然后选 2 或 3；`login.cmd` 等于选 2。

**方式二的流程：**

1. 弹出一个**只给插件用**的浏览器窗口（Edge/Chrome，配置目录在数据目录的 `browser\` 下，跟你日常浏览器完全分开，不读你的记录和登录态）；
2. 打开天翼 AI 控制台的模型概览页，你在里面登录（密码、短信都行）；
3. 插件从浏览器里读 Cookie，**真去打一次额度接口**——只有能查到额度才算登录成功；
4. 顺手调控制台的「应用及API Key」列表（`/teleai-api/surfing-api/change/applicationList`），把这个账号已有的 **API Key 领回来**交给 magpie。已经配过 Key 的保留原来那个，不会被覆盖；控制台里一个应用都没有时会提示你去 `console.teleai.com.cn/application` 点一次创建，再登录就自动拿到了；
5. 写进 `config.json`、把每个模型的余量打出来、自己关窗。

超时（默认 10 分钟）没读到有效登录态就退出，重跑一次即可。只读 Cookie，不碰账号密码，也不上传任何东西。

**方式三的流程：**

1. 浏览器打开 <https://console.teleai.com.cn/product/overview/TeleModel/Semantic>（先确认自己是登录状态）
2. F12 → 网络 → 刷新 → 找 `productResource/list` 这条请求 → 请求标头里 `Cookie` 那一行的值，整段复制
3. 回 magpie 选「API Key + 手动填 Cookie」，贴进 Cookie 那栏（Key 栏留空就自动从控制台领）

粘贴后插件会**先真去打一次额度接口**：查得到资源包才算成功，查不到会直接说原因（少了哪一段、是不是掉登录了），不会把一段没用的 Cookie 存下来。

**其实只有 `operToken` 是必需的。** 实测（把控制台那份 14 项 Cookie 拆开逐项试）：

| 给的 Cookie | 结果 |
| --- | --- |
| 全部 14 项 | 200，4 个模型 |
| 去掉 UM_distinctid / CNZZDATA / sensorsdata / CZ_UUID 等统计项 | 200，4 个模型 |
| **只有 `operToken`** | 200，4 个模型；领 API Key 也成功 |
| 其余单项（consoleDataKey / cna / loginSuccessFlag / 各种统计 id…） | 401 code=4001 |

所以从 F12 里只复制 `operToken=…` 那一段、甚至只复制 `operToken` 的值（不带 `operToken=`）都能用，插件会自己补上前缀。

不想进界面的话，也可以直接写进数据目录的 `config.json`：

```text
%USERPROFILE%\.local\share\teleai-maas\config.json
```

```json
{"consoleCookie": "复制出来的那一整串"}
```

注意：

- 这是控制台的**登录态**，等于账号密码，别外传、别提交到仓库。
- 登录态会过期（几周量级）。过期后额度那里会显示「未登录控制台」，重跑一次 `login.cmd` 就好。
- 插件的日志和报错里不会出现这个 Cookie（测试里有专门一条守着），额度缓存文件里也不放它。
- 查到的结果缓存 60 秒，不会每个请求都去打控制台。

### 用完了会怎样

控制台读到的免费余量为 0 时，插件直接回 429（`free_pack_exhausted`），不再转发，免得不知不觉走按量后付费。确认还有额度、或者接受计费时，把 `allowOverage` 设为 `true`。

读不到额度（没登录、控制台挂了、缓存过期）时**不拦**——宁可放行，也不误伤正常请求。

| 选项 | 作用 |
| --- | --- |
| `consoleCookie` | 控制台的 Cookie；`login.cmd` 会自动写，也可以手动填 |
| `allowOverage` | 控制台显示免费包用完时是否继续转发，默认 `false` |

优先级：插件选项 > `config.json` > 环境变量（`TELEAI_MAAS_CONSOLE_COOKIE`）；数据目录可以用 `TELEAI_MAAS_DATA` 覆盖，登录用的端口可以用 `TELEAI_LOGIN_PORT` 指定。

### 那个接口长什么样

留着以后维护用：

```http
POST https://console.teleai.com.cn/teleai-api/console/productResource/list
content-type: application/json
cookie: <控制台登录态>

{"muluRoute":"Semantic","pageNum":1,"pageSize":100,"prodName":""}
```

- `muluRoute` 必填，缺了报「muluRoute和muluRoutes不能同时为空」；
- 取值是 `"Semantic"`（概览页 `/product/overview/TeleModel/Semantic` 的最后一段）。填 `"TeleModel"` 只回空列表，填带斜杠的路径报「目录路由不存在」；
- 除了 Cookie 别的头都不用给，`AUTH_TOKEN_TY` / `operToken` 留空即可；
- 响应里每个模型一份 `yuLiangList`：`balanceInit` 总量、`balanceValue` 剩余、`unitName` 单位、`isFree` 是不是免费包；付费包也在里面（`isFree: false`）。

## 做了哪些适配

- `max_tokens` → `max_completion_tokens`（`max_tokens` 已弃用）
- 删除 `reasoning_effort`（三个模型都不支持）；取值为 none / minimal / off 时改用 `chat_template_kwargs.enable_thinking=false` 关闭思考
- 流式请求照原样转发，不再补 `stream_options`（本机不记账了）
- `messages[].content` 数组只保留文本部分（接口只支持 text）
- 401 响应带 `X-Magpie-Sign-In: expired`，magpie 会提示重新登录

## 开发

```sh
node --test          # 32 个用例
```

目录：`index.mjs` 入口；`login.mjs` 登录控制台取 Cookie（`login.cmd` 双击跑）；`lib/models.mjs` 模型表；`lib/request.mjs` 请求改写；`lib/console.mjs` 控制台额度 + 从控制台领 API Key；`lib/browser-login.mjs` 用 CDP 开浏览器登控制台读 Cookie；`lib/paths.mjs` 数据目录与 `config.json`。