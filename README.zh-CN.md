[English](README.md) | **中文**

<div align="center">
  <img src="fox.png" alt="camofox-browser" width="200" />
  <h1>camofox-browser</h1>
  <p><strong>面向 AI 代理的反检测浏览器服务，基于 Camoufox 构建</strong></p>
  <p>
    <a href="https://github.com/jo-inc/camofox-browser/actions"><img src="https://img.shields.io/badge/build-passing-brightgreen" alt="Build" /></a>
    <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/license-MIT-blue" alt="License" /></a>
    <a href="https://camoufox.com"><img src="https://img.shields.io/badge/engine-Camoufox-red" alt="Camoufox" /></a>
    <a href="https://hub.docker.com"><img src="https://img.shields.io/badge/docker-ready-blue" alt="Docker" /></a>
  </p>
  <p>
    站在 <a href="https://camoufox.com">Camoufox</a> 这位巨人的肩膀上 —— 一个基于 Firefox 的分支，在 C++ 层面实现指纹伪装。
    <br/><br/>
    <a href="https://askjo.ai?ref=camofox">Jo</a> 背后的同款引擎 —— 一个无需你时刻盯着的 AI 助手。一半运行在你的 Mac 上，一半运行在仅你专用的云端机器。支持 macOS、Telegram 和 WhatsApp。<a href="https://askjo.ai?ref=camofox">免费试用 Beta 版 →</a>
  </p>
</div>

<br/>

```bash
git clone https://github.com/jo-inc/camofox-browser && cd camofox-browser
npm install && npm start
# → http://localhost:9377
```

---

## 为什么

AI 代理需要浏览真实的网页。Playwright 会被拦截。Headless Chrome 会被指纹追踪。Stealth 插件本身也会成为指纹。

Camoufox 在 **C++ 实现层面** 对 Firefox 进行补丁 —— `navigator.hardwareConcurrency`、WebGL 渲染器、AudioContext、屏幕几何信息、WebRTC —— 在 JavaScript 看到它们之前就已经全部伪装完毕。没有 shim，没有 wrapper，没有破绽。

本项目将该引擎封装为专为代理设计的 REST API：使用可访问性快照替代臃肿的 HTML，使用稳定的 element ref 进行点击操作，以及面向常用站点的搜索宏。

## 功能特性

- **C++ 级反检测** - 绕过 Google、Cloudflare 和大多数机器人检测
- **Element Refs** - 稳定的 `e1`、`e2`、`e3` 标识符，确保交互可靠
- **Token 高效** - 可访问性快照比原始 HTML 小约 90%
- **随处可运行** - 懒加载浏览器启动 + 空闲自动关闭，空闲时内存仅约 40MB。设计与你的其他服务共享一台机器 —— Raspberry Pi、$5 VPS、共享 Railway 基础设施。
- **Session 隔离** - 每个用户拥有独立的 cookie/storage
- **Cookie 导入** - 注入 Netscape 格式的 cookie 文件，实现已认证浏览
- **代理 + GeoIP** - 通过住宅代理路由流量，自动设置 locale/timezone
- **结构化日志** - JSON 日志行，带 request ID，便于生产环境可观测性
- **YouTube 字幕提取** - 通过 yt-dlp 从任何 YouTube 视频提取字幕，无需 API key
- **搜索宏** - `@google_search`、`@youtube_search`、`@amazon_search`、`@reddit_subreddit` 等 10 余种
- **Snapshot 截图** - 在可访问性快照旁附带 base64 PNG 截图
- **大页面处理** - 自动截断 snapshot，支持基于 offset 的分页
- **下载捕获** - 捕获浏览器下载并通过 API 获取（可选 inline base64）
- **DOM 图片提取** - 列出 `<img>` 的 src/alt，可选返回 inline data URL
- **随处部署** - Docker、Fly.io、Railway
- **VNC 交互式登录** - 通过 noVNC 可视化登录站点，导出 storage state 供代理复用

## 可选依赖

| 依赖 | 用途 | 安装方式 |
|-----------|---------|---------|
| [yt-dlp](https://github.com/yt-dlp/yt-dlp) | YouTube 字幕提取（快速路径） | `pip install yt-dlp` 或 `brew install yt-dlp` |

Docker 镜像已包含 yt-dlp。本地开发时，请安装以使用 `/youtube/transcript` 端点。未安装时，该端点会回退到较慢的基于浏览器的方式。

## 快速开始

### OpenClaw 插件

```bash
openclaw plugins install @askjo/camofox-browser
```

**Tools:** `camofox_create_tab` · `camofox_snapshot` · `camofox_click` · `camofox_type` · `camofox_navigate` · `camofox_scroll` · `camofox_screenshot` · `camofox_close_tab` · `camofox_list_tabs` · `camofox_import_cookies`

### 独立运行

```bash
git clone https://github.com/jo-inc/camofox-browser
cd camofox-browser
npm install
npm start  # 首次运行下载 Camoufox（约 300MB）
```

默认端口为 `9377`。所有选项见 [环境变量](#环境变量)。

### Docker

附带的 `Makefile` 会自动检测你的 CPU 架构，并在 Docker 构建前预先下载 Camoufox + yt-dlp 二进制文件，因此重建很快（约 30 秒，而非约 3 分钟）。

```bash
# 构建并启动（自动检测架构：M1/M2 为 aarch64，Intel 为 x86_64）
make up

# 停止并移除容器
make down

# 强制干净重建（例如升级 VERSION/RELEASE 后）
make reset

# 仅下载二进制文件（不构建）
make fetch

# 显式覆盖架构或版本
make up ARCH=x86_64
make up VERSION=135.0.1 RELEASE=beta.24
```

> **⚠️ 请勿直接运行 `docker build`。** Dockerfile 使用 bind mount 从 `dist/` 拉取预下载的二进制文件。始终使用 `make up`（或先 `make fetch` 再 `make build`）—— 它会先下载二进制文件。

### Fly.io / Railway

已包含 `railway.toml`。对于 Fly.io 或其他远程 CI，你需要一个在构建时下载二进制文件而非使用 bind mount 的 Dockerfile —— 示例见 [jo-browser](https://github.com/jo-inc/jo-browser)。

## 使用

### Cookie 导入

将浏览器的 cookie 导入 Camoufox，以跳过 LinkedIn、Amazon 等站点的交互式登录。

#### 设置

**1. 生成 secret key：**

```bash
# macOS / Linux
openssl rand -hex 32
```

**2. 在启动 OpenClaw 前设置环境变量：**

```bash
export CAMOFOX_API_KEY="your-generated-key"
openclaw start
```

同一个 key 同时供插件（用于认证请求）和服务端（用于验证请求）使用。两者运行在同一个环境中 —— 设置一次即可。

> **为什么使用环境变量？** 该 key 是 secret。`openclaw.json` 中的插件配置以明文存储，因此 secret 不应放在那里。请在 shell profile、systemd unit、Docker env 或 Fly.io secrets 中设置 `CAMOFOX_API_KEY`。

> **Cookie 导入默认禁用。** 如果未设置 `CAMOFOX_API_KEY`，服务端会拒绝所有 cookie 请求并返回 403。

**3. 从浏览器导出 cookie：**

安装一个可导出 Netscape 格式 cookie 文件的浏览器扩展（例如 Chrome/Firefox 的 "cookies.txt"）。导出你想认证的站点的 cookie。

**4. 放置 cookie 文件：**

```bash
mkdir -p ~/.camofox/cookies
cp ~/Downloads/linkedin_cookies.txt ~/.camofox/cookies/linkedin.txt
```

默认目录为 `~/.camofox/cookies/`。可通过 `CAMOFOX_COOKIES_DIR` 覆盖。

**5. 让你的代理导入它们：**

> 从我的 linkedin.txt 导入 LinkedIn cookie

代理调用 `camofox_import_cookies` → 读取文件 → 使用 Bearer token POST 到服务端 → cookie 被注入浏览器 session。后续对 linkedin.com 的 `camofox_create_tab` 调用将处于已认证状态。

#### 工作原理

```
~/.camofox/cookies/linkedin.txt          (Netscape 格式，存储在磁盘)
        │
        ▼
camofox_import_cookies tool              (解析文件，按域名过滤)
        │
        ▼  POST /sessions/:userId/cookies
        │  Authorization: Bearer <CAMOFOX_API_KEY>
        │  Body: { cookies: [Playwright cookie objects] }
        ▼
camofox server                           (验证、清理、注入)
        │
        ▼  context.addCookies(...)
        │
Camoufox browser session                 (已认证浏览)
```

- `cookiesPath` 相对于 cookies 目录解析 —— 阻止目录遍历到外部
- 每次请求最多 500 个 cookie，文件大小限制 5MB
- Cookie 对象被清理为 Playwright 字段的允许列表

### Session 持久化

默认情况下，camofox 将每个用户的 cookie 和 localStorage 持久化到 `~/.camofox/profiles/`。Session 在浏览器重启后仍然保留 —— 只需登录一次（通过 cookie 或 VNC），后续 session 会自动恢复已认证状态。

```
~/.camofox/
├── cookies/          # 引导 cookie 文件（Netscape 格式）
└── profiles/         # 持久化 session 状态（自动管理）
    └── <hashed-userId>/
        └── storage_state.json
```

通过 `CAMOFOX_PROFILE_DIR` 覆盖目录，或在 persistence 插件配置中设置 `"profileDir"`。要禁用持久化，在 `camofox.config.json` 中设置 `"persistence": { "enabled": false }`。

#### 独立服务端使用

```bash
curl -X POST http://localhost:9377/sessions/agent1/cookies \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_CAMOFOX_API_KEY' \
  -d '{"cookies":[{"name":"foo","value":"bar","domain":"example.com","path":"/","expires":-1,"httpOnly":false,"secure":false}]}'
```

#### Docker / Fly.io

```bash
docker run -p 9377:9377 \
  -e CAMOFOX_API_KEY="your-generated-key" \
  -v ~/.camofox/cookies:/home/node/.camofox/cookies:ro \
  camofox-browser
```

对于 Fly.io：
```bash
fly secrets set CAMOFOX_API_KEY="your-generated-key"
```

### 代理 + GeoIP

通过代理路由所有浏览器流量，并借助 Camoufox 内置的 GeoIP 自动从代理 IP 地址派生 locale、timezone 和 geolocation。

**简单代理（单一端点）：**

```bash
export PROXY_HOST=166.88.179.132
export PROXY_PORT=46040
export PROXY_USERNAME=myuser
export PROXY_PASSWORD=mypass
npm start
```

**Backconnect 代理（轮转粘性 session）：**

对于 Decodo、Bright Data 或 Oxylabs 等提供单一网关端点并支持 session 粘性 IP 的提供商：

```bash
export PROXY_STRATEGY=backconnect
export PROXY_BACKCONNECT_HOST=gate.provider.com
export PROXY_BACKCONNECT_PORT=7000
export PROXY_USERNAME=myuser
export PROXY_PASSWORD=mypass
npm start
```

每个浏览器 context 获得一个唯一的粘性 session，因此不同用户获得不同的 IP 地址。Session 在代理错误或 Google 拦截时自动轮转。

或在 Docker 中：

```bash
docker run -p 9377:9377 \
  -e PROXY_HOST=166.88.179.132 \
  -e PROXY_PORT=46040 \
  -e PROXY_USERNAME=myuser \
  -e PROXY_PASSWORD=mypass \
  camofox-browser
```

配置代理后：
- 所有流量通过代理路由
- Camoufox 的 GeoIP 自动设置 `locale`、`timezone` 和 `geolocation` 以匹配代理的出口 IP
- 浏览器指纹（语言、时区、坐标）与代理位置一致
- 无代理时，默认为 `en-US`、`America/Los_Angeles`、旧金山坐标

### 结构化日志

所有日志输出均为 JSON（每行一个对象），便于日志聚合器解析：

```json
{"ts":"2026-02-11T23:45:01.234Z","level":"info","msg":"req","reqId":"a1b2c3d4","method":"POST","path":"/tabs","userId":"agent1"}
{"ts":"2026-02-11T23:45:01.567Z","level":"info","msg":"res","reqId":"a1b2c3d4","status":200,"ms":333}
```

健康检查请求（`/health`）被排除在请求日志之外，以减少噪音。

### 基本浏览

```bash
# 创建标签页
curl -X POST http://localhost:9377/tabs \
  -H 'Content-Type: application/json' \
  -d '{"userId": "agent1", "sessionKey": "task1", "url": "https://example.com"}'

# 获取带 element refs 的可访问性快照
curl "http://localhost:9377/tabs/TAB_ID/snapshot?userId=agent1"
# → { "snapshot": "[button e1] Submit  [link e2] Learn more", ... }

# 通过 ref 点击
curl -X POST http://localhost:9377/tabs/TAB_ID/click \
  -H 'Content-Type: application/json' \
  -d '{"userId": "agent1", "ref": "e1"}'

# 在元素中输入文本
curl -X POST http://localhost:9377/tabs/TAB_ID/type \
  -H 'Content-Type: application/json' \
  -d '{"userId": "agent1", "ref": "e2", "text": "hello", "pressEnter": true}'

# 使用搜索宏导航
curl -X POST http://localhost:9377/tabs/TAB_ID/navigate \
  -H 'Content-Type: application/json' \
  -d '{"userId": "agent1", "macro": "@google_search", "query": "best coffee beans"}'
```

## API

### 标签页生命周期

| Method | Endpoint | 描述 |
|--------|----------|-------------|
| `POST` | `/tabs` | 创建带初始 URL 的标签页 |
| `GET` | `/tabs?userId=X` | 列出打开的标签页 |
| `GET` | `/tabs/:id/stats` | 标签页统计（tool 调用、访问过的 URL） |
| `DELETE` | `/tabs/:id` | 关闭标签页 |
| `DELETE` | `/tabs/group/:groupId` | 关闭组内所有标签页 |
| `DELETE` | `/sessions/:userId` | 关闭用户的所有标签页 |

### 页面交互

| Method | Endpoint | 描述 |
|--------|----------|-------------|
| `GET` | `/tabs/:id/snapshot` | 带 element refs 的可访问性快照。查询参数：`includeScreenshot=true`（添加 base64 PNG）、`offset=N`（大快照分页） |
| `POST` | `/tabs/:id/click` | 通过 ref 或 CSS selector 点击元素 |
| `POST` | `/tabs/:id/type` | 在元素中输入文本 |
| `POST` | `/tabs/:id/press` | 按下键盘按键 |
| `POST` | `/tabs/:id/scroll` | 滚动页面（上/下/左/右） |
| `POST` | `/tabs/:id/navigate` | 导航到 URL 或搜索宏 |
| `POST` | `/tabs/:id/wait` | 等待 selector 或超时 |
| `GET` | `/tabs/:id/links` | 提取页面所有链接 |
| `GET` | `/tabs/:id/images` | 列出 `<img>` 元素。查询参数：`includeData=true`（返回 inline data URL）、`maxBytes=N`、`limit=N` |
| `GET` | `/tabs/:id/downloads` | 列出已捕获的下载。查询参数：`includeData=true`（base64 文件数据）、`consume=true`（读取后清除）、`maxBytes=N` |
| `GET` | `/tabs/:id/screenshot` | 截图 |
| `POST` | `/tabs/:id/back` | 后退 |
| `POST` | `/tabs/:id/forward` | 前进 |
| `POST` | `/tabs/:id/refresh` | 刷新页面 |

### YouTube 字幕

| Method | Endpoint | 描述 |
|--------|----------|-------------|
| `POST` | `/youtube/transcript` | 从 YouTube 视频提取字幕 |

```bash
curl -X POST http://localhost:9377/youtube/transcript \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ", "languages": ["en"]}'
# → { "status": "ok", "transcript": "[00:18] ♪ We're no strangers to love ♪\n...", "video_title": "...", "total_words": 548 }
```

当 yt-dlp 可用时使用它（快速，无需浏览器）。如果未安装 yt-dlp，则回退到基于浏览器的拦截方式 —— 由于 YouTube 广告前贴片，这种方式更慢且不太可靠。

### 服务端

| Method | Endpoint | 描述 |
|--------|----------|-------------|
| `GET` | `/health` | 健康检查 |
| `POST` | `/start` | 启动浏览器引擎 |
| `POST` | `/stop` | 停止浏览器引擎 |

### Sessions

| Method | Endpoint | 描述 |
|--------|----------|-------------|
| `POST` | `/sessions/:userId/cookies` | 向用户 session 添加 cookie（Playwright cookie 对象） |
| `GET` | `/sessions/:userId/storage_state` | 导出 cookie + localStorage（[VNC 插件](plugins/vnc/)） |

## 搜索宏

`@google_search` · `@youtube_search` · `@amazon_search` · `@reddit_search` · `@reddit_subreddit` · `@wikipedia_search` · `@twitter_search` · `@yelp_search` · `@spotify_search` · `@netflix_search` · `@linkedin_search` · `@instagram_search` · `@tiktok_search` · `@twitch_search`

Reddit 宏直接返回 JSON（无需 HTML 解析）：
- `@reddit_search` - 搜索整个 Reddit，返回包含 25 条结果的 JSON
- `@reddit_subreddit` - 浏览 subreddit（例如 query `"programming"` → `/r/programming.json`）

## 环境变量

| Variable | 描述 | 默认值 |
|----------|-------------|---------|
| `CAMOFOX_PORT` | 服务端端口 | `9377` |
| `PORT` | 服务端端口（后备，用于 Fly.io 等平台） | `9377` |
| `CAMOFOX_API_KEY` | 启用 cookie 导入端点（未设置则禁用） | - |
| `CAMOFOX_ADMIN_KEY` | `POST /stop` 所需 | - |
| `CAMOFOX_COOKIES_DIR` | cookie 文件目录 | `~/.camofox/cookies` |
| `CAMOFOX_PROFILE_DIR` | 持久化 session profile 目录 | `~/.camofox/profiles` |
| `MAX_SESSIONS` | 最大并发浏览器 session 数 | `50` |
| `MAX_TABS_PER_SESSION` | 每个 session 的最大标签页数 | `10` |
| `SESSION_TIMEOUT_MS` | Session 不活动超时 | `1800000` (30min) |
| `BROWSER_IDLE_TIMEOUT_MS` | 空闲时关闭浏览器（0 = 永不） | `300000` (5min) |
| `HANDLER_TIMEOUT_MS` | 任何 handler 的最大执行时间 | `30000` (30s) |
| `MAX_CONCURRENT_PER_USER` | 每个用户的并发请求上限 | `3` |
| `MAX_OLD_SPACE_SIZE` | Node.js V8 堆限制（MB） | `128` |
| `PROXY_STRATEGY` | 代理模式：`backconnect`（轮转粘性 session）或留空（单一端点） | - |
| `PROXY_PROVIDER` | session 格式的提供商名称（例如 `decodo`） | `decodo` |
| `PROXY_HOST` | 代理主机名或 IP（简单模式） | - |
| `PROXY_PORT` | 代理端口（简单模式） | - |
| `PROXY_USERNAME` | 代理认证用户名 | - |
| `PROXY_PASSWORD` | 代理认证密码 | - |
| `PROXY_BACKCONNECT_HOST` | Backconnect 网关主机名 | - |
| `PROXY_BACKCONNECT_PORT` | Backconnect 网关端口 | `7000` |
| `PROXY_COUNTRY` | 代理地理定向的目标国家 | - |
| `PROXY_STATE` | 代理地理定向的目标州/省 | - |
| `TAB_INACTIVITY_MS` | 关闭空闲超过该时长的标签页 | `300000` (5min) |
| `ENABLE_VNC` | 启用 VNC 插件以进行交互式浏览器访问（`1`） | - |
| `VNC_PASSWORD` | VNC 访问密码（生产环境建议设置） | - |
| `NOVNC_PORT` | noVNC Web UI 端口 | `6080` |

## 架构

```
Browser Instance (Camoufox)
└── User Session (BrowserContext) - 隔离的 cookies/storage
    ├── Tab Group (sessionKey: "conv1")
    │   ├── Tab (google.com)
    │   └── Tab (github.com)
    └── Tab Group (sessionKey: "conv2")
        └── Tab (amazon.com)
```

Session 在不活动 30 分钟后自动过期。浏览器本身在无活动 session 5 分钟后关闭，并在下次请求时重新启动。

当 session 的标签页上限达到时，最旧/最少使用的标签页会被自动回收，而不会返回错误 —— 因此长时间运行的代理 session 不会陷入死胡同。

## 测试

```bash
npm test              # 所有测试
npm run test:e2e      # 仅 e2e 测试
npm run test:live     # 实时站点测试（Google、宏）
npm run test:debug    # 带服务端输出
```

## npm

```bash
npm install @askjo/camofox-browser
```

## 鸣谢

- [Camoufox](https://camoufox.com) - 基于 Firefox 的 C++ 级反检测浏览器
- [Donate to Camoufox's original creator daijro](https://camoufox.com/about/)
- [OpenClaw](https://openclaw.ai) - 开源 AI 代理框架

## 加密货币诈骗警告

一些可疑人士正在利用名为 "Camofox" 的加密货币代币做可疑的事情，因为该项目正在获得关注。**Camoufox 不是加密货币项目，也永远不会是。** 任何使用 Camoufox 名称的代币、硬币或 NFT 都与我们无关。

## License

MIT
