# VAELOR

**DXE-1.0.0** · **红队全链路自动化攻击平台** — 面向**授权**渗透测试与攻防演练的集成化桌面工具。

> **仅限在取得书面授权的目标上使用。** 未经授权的探测、利用或破坏行为可能违法，相关责任由使用者自行承担。
>
> 产品名与界面文案统一为 **VAELOR**；可执行文件、Go 模块路径（`erraticgopher`）与
> 数据目录（`…/Application Support/ErraticGopher`）沿用旧名 —— 这几处改名会让存量
> 用户的数据目录与升级路径失效，属于刻意保留的技术标识。
>
> 版本号以 `wails.json` 的 `info.productVersion` 为准，
> `cd frontend && node scripts/check-version-consistency.mjs` 会核对所有出现处。

---

## 目录

- [简介](#简介)
- [功能模块](#功能模块)
- [技术栈](#技术栈)
- [架构](#架构)
- [环境要求](#环境要求)
- [编译与运行](#编译与运行)
- [数据目录](#数据目录)
- [目录结构](#目录结构)
- [常见问题](#常见问题)
- [开源说明](#开源说明)

---

## 简介

VAELOR 把**资产发现**、**站点测绘**、**自动化渗透**、**漏洞归档**串成一条链路，
数据按**项目**隔离，各模块结果互相联动。核心是 LLM 驱动的 **ReACT 智能体**：
它自己决定调哪个工具、看什么结果、下一步打哪里，最后把可复现的漏洞结论落进账本。

核心特性：

- **ReACT 智能体引擎**：多 Provider 对接，多步推理，自动调用扫描器 / 浏览器 / 测绘平台
- **Headless 浏览器实测**：SPA 渲染、表单填充、截图取证、敏感信息提取
- **多源资产融合**：端口扫描 + 空间测绘 + Web 爬虫结果统一入库，形成资产全景
- **漏洞账本 + 报告**：全流程留痕，报告按证据门槛生成，缺证据的条目会被降级
- **双通信形态**：桌面（Wails WebView）与 Web（HTTP + WebSocket）同一份后端

---

## 功能模块

### 1. 项目管理与仪表盘

- 多项目创建、归档、统计仪表盘（一屏不滚动，面板内部滚动）
- 目标资产（IP / 域名 / URL）管理与标签分类
- Token 消耗、漏洞总览、探索中会话与活动流

### 2. 智能体（Agent）

- 对接多种大模型 Provider（OpenAI 兼容、Anthropic Claude、DeepSeek 等）
- **ReACT 多步循环**：思考 → 工具调用 → 观察 → 继续，直到任务收敛
- 内置工具面：
  - 端口扫描 / 目录扫描 / 空间测绘查询（FOFA、Hunter、Quake、Shodan）
  - Headless 浏览器（导航、读取、截图、表单、网络日志）
  - HTTP 探测（含会话导入、报文落库、去重与证据链）
  - 子任务派发（dispatch_subtask）与子代理树
  - 工作区文件、持久 shell、后台作业、凭据字典、技能包
- 角色体系（渗透 / 红队 / CTF / 站点侦察）+ 工具 ACL + Skills 覆盖
- 会话记忆、上下文压缩、断点续跑、崩溃恢复
- 报告：模型不写报告，只管出结论；报告由用户在报告面板按账本生成

### 3. 漏洞管理（`/projects/:id/vuln-reports`）

- 项目级漏洞条目（账本）记录、确认、跟踪
- 报告生成与导出（Markdown / PDF），带证据门槛

### 4. 漏洞测绘（`/projects/:id/webcrawl`）

- 全站爬虫：SPA 渲染、API 端点发现、JS 路径静态分析
- 敏感信息提取：JWT、API Key、密码、Token、内部 IP 等
- 资源树浏览（HTML / JS / CSS / 图片 / 字体分类）
- 档位：深度爬取 / 存活探测
- 与智能体联动：高价值站点直接派给智能体深挖

### 5. 集权集控（`/projects/:id/pam`）

- 堡垒机 / 特权系统 / 统一认证面的利用集合

### 6. 云上综合（`/projects/:id/cloud`）

- 公有云开放平台与容器编排的利用集合（云存储、云服务、接口库）

### 7. 空间扫描（`/projects/:id/portscan`）

- TCP / SYN / UDP 多模式端口扫描，常用端口模板 + 全端口
- Nmap 服务指纹（Banner + 协议探测）、Web 标题与 TLS 证书抓取
- 离线 IP 归属（ip2region）、结果分页 / 导出 / 资产详情联动

### 8. 空间测绘（`/projects/:id/spacemap`）

- FOFA / Quake / Hunter / Shodan 语法查询、分页入库、去重
- 与扫描结果交叉关联

### 9. 军团总览（`/agents`）

- 跨项目汇总所有智能体当前在做什么、卡在哪一步

### 10. 系统配置（`/settings`）

- 大模型 Provider（多渠并存，启用/禁用，连接方言实测事实）
- 空间测绘 API 凭据、全局 HTTP 代理与代理池
- 智能体默认参数、角色与 Skills、MCP 外部工具服务器
- 消息通知（飞书 / 企业微信 Webhook）、外观与语言

---

## 技术栈

| 层 | 技术 |
|----|------|
| 桌面框架 | Wails v2（Go + WebView2 / WKWebView） |
| 后端 | Go 1.26+、SQLite（modernc，纯 Go 无 cgo） |
| 前端 | Nuxt 3 + Vue 3 + Naive UI + TypeScript |
| 浏览器引擎 | chromedp（Headless Chrome / 系统 Chrome） |
| 通信 | Wails Bindings（桌面）/ HTTP + WebSocket（Web 形态，默认 `0.0.0.0:8963`） |

---

## 架构

```
┌──────────────────────────────────────────────┐
│                   Frontend                    │
│          Nuxt 3 + Vue 3 + Naive UI           │
│   (app.vue → layouts/default.vue → pages/)   │
├──────────────────────────────────────────────┤
│              Communication Layer              │
│         Wails Bindings | HTTP/WS (webengine) │
├──────────────────────────────────────────────┤
│               Go Backend (main.go)            │
│                                               │
│  ┌──────────┐ ┌──────────┐ ┌─────────────┐   │
│  │ Agent    │ │ PortScan │ │ WebCrawl    │   │
│  │ (ReACT)  │ │ Engine   │ │ Engine      │   │
│  ├──────────┤ ├──────────┤ ├─────────────┤   │
│  │ Agent    │ │ DirScan  │ │ SpaceMap    │   │
│  │ Browser  │ │ Engine   │ │ Engine      │   │
│  ├──────────┤ ├──────────┤ ├─────────────┤   │
│  │ Agent    │ │ GeoIP    │ │ Notify      │   │
│  │ NetTrace │ │ Lookup   │ │ (Wx/Feishu) │   │
│  └──────────┘ └──────────┘ └─────────────┘   │
├──────────────────────────────────────────────┤
│                 Data Layer                    │
│   SQLite (global.db + project.db)             │
│   File System (workspaces, evidence, certs)   │
└──────────────────────────────────────────────┘
```

### 启动流程

```
main()
  └→ wails.Run()
       └→ OnStartup: app.startup(ctx)
            ├→ 放行系统浏览器（未随包 Chromium 时）
            ├→ 创建 agentBrowser（含报文落库 sink）
            ├→ 载入各渠道 LLM 能力档案
            └→ startBoot()（sync.Once）
                 └→ runBootSequence()
                      ├→ 初始化 global.db（含迁移失败计数，失败会写进启动日志）
                      ├→ 应用模型超时 / 代理池 / 全局代理
                      ├→ 初始化 MCP Hub 与外部钩子
                      ├→ 创建扫描 / 目录扫描 / 测绘 / 空间测绘引擎
                      ├→ 组装 Agent Runner 与 Dispatcher
                      ├→ markBootReady() → 前端显示主界面
                      └→ runDeferredBoot()（后台异步：GeoIP、断点续扫等）
```

前端启动门是 `useAppBoot`（轮询 `GetBootStatus`）+ `AppSplashScreen`：启动完成前只显示启动日志。

### 数据流

```
用户操作 (UI)
  → composables/useApi.ts   (Wails call / HTTP fetch)
    → main.go App Method    (Go backend)
      → internal/* engines  (business logic)
        → SQLite / Filesystem
      → Events Emit         (boot:log, portscan:result, agent:message ...)
    → frontend EventsOn listener → UI 更新
```

---

## 环境要求

| 项 | 要求 |
|----|------|
| Go | 1.26+ |
| Node.js | 22+（前端构建） |
| Wails CLI | v2（`go install github.com/wailsapp/wails/v2/cmd/wails@latest`） |
| CGO | 桌面构建需要（macOS: Xcode CLT；Windows: WebView2 运行时；Linux: 见 [docs/BUILD.md](./docs/BUILD.md)） |
| 浏览器 | 智能体浏览器能力需要 Chrome / Chromium（可用 `scripts/fetch-browser.sh` 随包） |

---

## 编译与运行

### 0. 一条命令（推荐）

```bash
bash scripts/build.sh                 # 前端 → 资源包校验 → wails build → 随包浏览器
bash scripts/build.sh --with-browser  # 缺暂存时先下载随包浏览器（约 150–200MB）
bash scripts/build.sh --dev           # 只编二进制（复用已有 frontend/dist，最快）
bash scripts/build.sh --no-wails      # 前端 + go build（交叉编译 / CI）
bash scripts/build.sh --zip           # 再出一个保留目录结构的 zip
```

脚本把下面 1–4 步串起来，每步失败立刻停并告诉你下一步做什么。
排查细节见 [docs/BUILD.md](./docs/BUILD.md)。下面是不用脚本时的分步做法。

### 1. 装依赖

```bash
cd frontend && npm install && cd ..
go install github.com/wailsapp/wails/v2/cmd/wails@latest
```

### 2. 前端产物

```bash
cd frontend
npm run build        # nuxt generate → .output/public → dist/
```

`dist/` 是 Wails 的 `assetdir`，也是默认构建里**直接 embed** 的前端产物。

### 3. 资源包（可选）

仓库里 `internal/egres/data/*.egres` 已经有明文产物，日常不需要重打。
只有改了这些源目录（字典、服务探测规则、技能包、角色提示词等）才需要：

```bash
go run ./cmd/egpack -v           # 全量重打
go run ./cmd/egpack -check       # 只校验清单与现有产物是否一致
go run ./cmd/egpack -only ResRolePrompts -v
```

> 开源版资源包是**明文**（`single-file` 原样、目录型是 `EGA1` 归档），
> 没有加密 / 密钥派生 / `.text` 指纹绑定这条链路。

### 4. 桌面构建

```bash
wails build                 # 产物：build/bin/ErraticGopher
wails build -race           # 调试
wails dev                   # 前端热重载
```

`go build .` 也能出可执行文件（前端产物已在 `frontend/dist` 时会被打进去），
适合快速验证后端链路。

### 5. Web 形态

```bash
./ErraticGopher --web
# EG_WEB_ADDR=127.0.0.1:8963  仅本机
# EG_WEB_TOKEN=<随机串>       开启令牌校验（不设 = 开放模式）
# EG_WEB_ALLOW_REMOTE=1       允许非回环访问
```

### 6. 无头模式（跑一次完整任务后退出）

```bash
./ErraticGopher --headless-agent \
  --target http://192.0.2.10:8080/ \
  --role pentest --task react --timeout 120
```

数据目录用 `ERRATICGOPHER_DATA_DIR` 指定；日志落在 `<数据目录>/logs/`。

### 7. 版本号

`frontend/constants/app.ts` 与 `wails.json` 的 `info.productVersion` 必须一致：

```bash
cd frontend && npm run version:check
```

---

## 数据目录

默认位置随平台（macOS `~/Library/Application Support/ErraticGopher`，Windows `%APPDATA%`，
Linux `~/.config`），可用 `ERRATICGOPHER_DATA_DIR` 覆盖。

```
<数据目录>/
├── global.db              # 全局库：项目列表、供应商、设置、角色、通知
├── projects/<id>/         # 项目库 + 工作区
│   └── project.db         # 资产、扫描、会话、账本、报文
├── logs/                  # 运行日志
├── results/               # 扫描 / 导出产物
├── skills/                # 用户技能包
└── running.marker         # 异常终止取证标记（正常退出会删）
```

同一个数据目录**同时只能被一个实例占用**（`.eg-instance.lock`）。
确实要并行（例如界面开着 + 另外跑 `--headless-agent`）时：
设 `EG_ALLOW_MULTI_INSTANCE=on`，并用 `ERRATICGOPHER_DATA_DIR` 把两者指向**不同**目录。

---

## 目录结构

```
.
├── main.go                    # Wails 绑定层 + 启动序列
├── app_*.go                   # 按域切分的 *App 方法（projects / agent / report / infra …）
├── internal/
│   ├── agent/                 # ReACT 引擎、工具面、提示词装配、守卫
│   ├── project/               # 项目管理、存储门面、账本、攻击图
│   │   └── store/             # 按域拆分的 store（agent / roles / webcrawl …）
│   ├── portscan/  dirscan/  webcrawl/  spacemap/   # 四个引擎
│   ├── webengine/             # 暴露面门禁（哪些方法在服务端形态下不可达）
│   ├── server/                # Web 形态的 HTTP + WebSocket 传输
│   ├── agentbrowser/          # chromedp 封装与证据链
│   ├── llm/                   # Provider 客户端、连接方言、上下文压缩
│   ├── egres/                 # 资源包加载（明文直通）+ 归档格式
│   └── invariants/            # 不变量登记与机械守卫
├── cmd/
│   ├── egpack/                # 资源打包器
│   ├── agentheadless/         # 无头回归场景
│   └── seedprovider/          # 测试用 provider 预置
├── frontend/
│   ├── pages/  components/  composables/  utils/
│   ├── constants/agentTools.ts# 工具标签表（与后端注册表由守卫互锁）
│   └── scripts/               # 前端结构 / i18n / 版本一致性守卫
├── scripts/                   # 打包、依赖、浏览器获取
└── wails.json                 # 产物名、版本号、assetdir
```

---

## 常见问题

**启动后卡在启动日志页**
看 `<数据目录>/logs/erraticgopher.log`。最常见是数据目录被另一个实例占用，
日志里会直接写出占用进程的 pid 与启动时间。

**数据库迁移有失败语句**
启动日志会打印「数据库迁移有 N 条语句失败」，并指向 `[migrate]` 行。
半迁移 schema 会让相关功能以「空列表 / 设置存不下去」的形式坏掉，不要忽略这条。

**智能体不调用工具 / 一直说没模型**
先确认设置里有**启用**的 Provider，再确认模型名在该渠道真实存在。
测试库里的连接方言（不吃 tool_choice、关思考才复用等）是启动时载入的，
换渠道后建议重启一次。

**浏览器工具全废**
没随包 Chromium 时会放行系统 Chrome/Chromium，启动日志里有
`[browser] 未随包浏览器，已放行系统浏览器：<路径>`。没有这一行说明两者都没找到。

**端口扫描 / 测绘没结果**
检查代理设置：分域出口是独立的（agent / llm / portscan / webcrawl … 各一条），
启动日志会打印 `代理路由: …` 一句话自检。

---

## 开源说明

- 本仓库是**去授权化**版本：没有激活、租约、吊销、策略轮询，也没有资源包加密与
  运行期自检。删掉的是发行控制链路，不是功能。
- 与之配套移除的模块：黑泽（`/heize`）、漏洞库（`/vulnmanager`）、指纹管理（`/fingerprint`）、
  北极星（Polaris）联动、小程序反编译与数据处理面板、SSH 远程主机与 Web 服务设置页。
- 资源包、角色提示词、工具标签表现已全部为明文，便于审计与二次开发。

### 免责声明

VAELOR 是一款面向**授权**安全测试与红队演练的集成化工具平台。
使用本工具前，您须确保已获得目标系统或网络所有者的**书面授权**，
并严格遵守所在国家/地区的法律法规及行业规范。

未经授权对任何系统、网络或数据进行探测、扫描、利用或破坏，均可能构成违法行为，
相关法律责任由使用者自行承担。

本软件按「现状」提供，开发者不对因误用、滥用或未授权使用本工具而导致的任何
直接或间接损失、数据泄露、业务中断或法律纠纷承担责任。下载、安装或使用本工具，
即表示您已阅读、理解并同意上述条款。
