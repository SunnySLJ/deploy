# 🦐 OpenClaw 一键部署 SOP（标准操作流程）

> **版本**: v2.1 | **更新日期**: 2026-04-02
>
> 本文档是 OpenClaw 智能视频发布系统的完整部署标准操作流程。
> 适用于**全新部署**和**跨机器迁移**两种场景。

---

## 📋 目录

1. [部署前准备](#1-部署前准备)
2. [获取部署包](#2-获取部署包)
3. [执行部署脚本](#3-执行部署脚本)
4. [部署过程详解（14 步）](#4-部署过程详解14-步)
5. [部署后操作](#5-部署后操作)
6. [验证清单](#6-验证清单)
7. [常见问题排查](#7-常见问题排查)
8. [迁移部署指南](#8-迁移部署指南)
9. [部署包文件清单](#9-部署包文件清单)

---

## 1. 部署前准备

### 1.1 硬件/系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | macOS 12+ 或 Windows 10/11 |
| 内存 | 8GB+（推荐 16GB） |
| 磁盘空间 | 至少 2GB 可用 |
| 网络 | 需要访问 GitHub、npm、API 服务 |

### 1.2 软件前置条件

| 软件 | 版本要求 | 检查命令 | 安装方式 |
|------|---------|---------|---------|
| **Node.js** | v18+ | `node -v` | [nodejs.org](https://nodejs.org/) 或 `brew install node` |
| **npm** | 随 Node.js 安装 | `npm -v` | 随 Node.js |
| **Git** | 任意版本 | `git --version` | [git-scm.com](https://git-scm.com/) 或 `brew install git` |
| **Python 3.12** | 3.12.x | `python3.12 --version` | `brew install python@3.12` 或 [python.org](https://www.python.org/) |

### 1.3 需要准备的信息

> ⚠️ 以下信息在部署过程中交互式输入，**不会硬编码到任何文件中**。

| 信息 | 说明 | 示例 |
|------|------|------|
| **LLM API Key** | 百炼 或 n1n.ai 的密钥 | `sk-sp-xxx`（百炼）或 `sk-xxx`（n1n） |
| **视频生成 Token** | 帧龙虾服务的认证 Token | 从帧龙虾平台获取 |
| **微信 Target ID**（可选） | 接收通知的微信 ID | `xxx@im.wechat`（绑定微信后获取） |
| **飞书 App ID/Secret**（可选） | 如果使用飞书通道 | 从飞书开放平台获取 |

### 1.4 需要确认的偏好

| 偏好 | 说明 | 举例 |
|------|------|------|
| **你的称呼** | AI 怎么叫你 | 千千、小明、老板 |
| **你的行业** | 用于生成匹配的标题/文案/标签 | 美妆、科技、美食、教育、宠物 |
| **视频风格** | 内容创作风格 | 可爱风、科技感、文艺、搞笑、治愈 |
| **AI 助手名字** | 你想叫 AI 什么名字 | 虾王、小助、阿福 |
| **AI 性格** | AI 的回复风格 | 轻松幽默 / 严谨专业 / 活泼可爱 |
| **发布确认** | 发布视频前是否需要你手动确认 | 是 / 否 |
| **登录检查时间** | 每天几点检查平台登录状态 | 10:10 |

---

## 2. 获取部署包

### 方式 A：直接复制（推荐）

将 `deploy-openclaw/` 整个目录复制到目标机器的任意位置：

```bash
# 例如通过 scp
scp -r deploy-openclaw/ user@newmachine:~/deploy-openclaw/

# 或者通过 U 盘、AirDrop、云盘等
```

### 方式 B：从 Git 仓库拉取

```bash
git clone https://github.com/SunnySLJ/deploy-openclaw.git
```

---

## 3. 执行部署脚本

### macOS

```bash
# 进入部署包目录
cd deploy-openclaw/

# 赋予执行权限
chmod +x deploy-openclaw.sh

# 运行一键部署
./deploy-openclaw.sh
```

### Windows

```powershell
# 以管理员身份打开 PowerShell

# 允许脚本执行（如果需要）
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

# 进入部署包目录
cd deploy-openclaw\

# 运行一键部署
.\deploy-openclaw.ps1
```

### 选择部署模式

脚本启动后会询问：

```
部署模式：
  1) 全新部署 — 从零安装所有组件
  2) 迁移部署 — 仅复制配置文件和技能（OpenClaw 已安装）

请选择 (1/2, 默认 1):
```

- **全新部署（1）**：适用于全新机器，执行全部 14 步
- **迁移部署（2）**：适用于 OpenClaw 已安装的机器，跳过安装步骤

---

## 4. 部署过程详解（14 步）

### 步骤 1️⃣：检查系统环境

**自动执行** — 检查 Node.js / npm / npx / Git / Homebrew

```
[步骤 1/14] 检查系统环境
  ✅ 操作系统: macOS 15.3
  ✅ Node.js: v22.14.0
  ✅ npm: 10.9.2
  ✅ npx: 可用
  ✅ Git: 2.48.1
  ✅ Homebrew: 已安装
```

> 如果缺少必要软件，脚本会提示安装方式并退出。

---

### 步骤 2️⃣：检查 / 安装 Python 3.12

**自动检测** — 按优先级查找 Python 3.12

| 平台 | 查找顺序 |
|------|---------|
| macOS | `/opt/homebrew/bin/python3.12` → `/usr/local/bin/python3.12` → `python3.12` |
| Windows | `py -3.12` → `python3.12` → `python` |

> 如果未找到，macOS 会提示使用 Homebrew 安装。

---

### 步骤 3️⃣：安装 OpenClaw（固定版本 2026.3.28）

**自动执行** — 安装稳定版 OpenClaw

```bash
npm install -g @anthropics/openclaw@2026.3.28
```

同时创建目录结构：
```
~/.openclaw/
├── workspace/
│   ├── inbound_images/     # 接收的图片
│   ├── inbound_videos/     # 接收的视频
│   ├── logs/auth_qr/       # 登录二维码
│   └── memory/             # 每日记忆
└── skills/                 # 技能目录
```

---

### 步骤 4️⃣：安装微信插件

**自动执行** — 安装微信通道

```bash
npx -y @tencent-weixin/openclaw-weixin-cli@latest install
```

> ⚠️ 安装完成后需要在步骤 5（部署后）手动扫码绑定微信。

---

### 步骤 5️⃣：安装飞书插件（可选）

**交互式** — 询问是否安装飞书

```
是否安装飞书插件？ [y/N]: y
请输入飞书 App ID: cli_xxx
请输入飞书 App Secret: xxx
```

> 选择 N 则跳过，后续可手动安装。

---

### 步骤 6️⃣：用户个性化初始化

**交互式** — 收集用户偏好和 AI 身份设定

```
👤 用户信息
  你希望 AI 怎么称呼你？(例: 千千、小明): 千千
  你所在的行业？(例: 美妆、科技、美食): 美妆
  你的视频风格？(例: 可爱风、科技感、文艺): 可爱风

🤖 AI 助手身份设定
  AI 助手名字？(默认: 虾王): 虾王
  AI 代表表情？(默认: 🦐): 🦐
  AI 性格风格？(默认: 轻松幽默): 轻松幽默

🎬 视频发布设置
  发起视频发布前是否需要人工确认？(推荐 Yes) [Y/n]: Y

✍️ AI 灵魂 (SOUL) 设定
  1) 默认模板 — 注重实用、有个性
  2) 严谨专业 — 正式、稳重
  3) 活泼互动 — 可爱、主动聊天
  选择灵魂风格 (1/2/3, 默认 1): 1
```

**这些信息会自动写入**：
- `IDENTITY.md` → AI 名字/表情/性格
- `SOUL.md` → 行为准则（3 种风格模板）
- `USER.md` → 用户称呼/行业
- `config.yaml` → 行业/风格/发布确认开关

---

### 步骤 7️⃣：配置 LLM 大模型

**交互式** — 选择 LLM 服务商

```
🧠 选择 LLM 服务商：
  1) 百炼 Coding Plan — 通义千问系列 (qwen3-coder-plus 等)
     API: https://coding.dashscope.aliyuncs.com

  2) n1n.ai — GPT-4.1 + Claude Opus 4.1
     API: https://api.n1n.ai
     文档: https://docs.n1n.ai/

请选择 (1/2, 默认 2): 2
请输入 n1n.ai API Key: sk-xxx
```

| 选项 | 主模型 | Embedding | memory LLM | lossless-claw |
|------|--------|-----------|------------|---------------|
| 百炼 | qwen3-coder-plus | text-embedding-v4 | qwen3-coder-plus | bailian/qwen3-coder-plus |
| n1n | **GPT-4.1** | text-embedding-3-small | gpt-4.1 | openai/gpt-4.1 |

> memory-lancedb-pro 和 lossless-claw 的 LLM 配置会自动匹配所选服务商。

---

### 步骤 8️⃣：安装 xiaolong-upload（图生视频）

**自动执行** — 克隆项目 + 安装依赖

```bash
git clone https://github.com/SunnySLJ/xiaolong-upload.git
# + Python venv + pip install
```

---

### 步骤 9️⃣：安装 openclaw_upload（视频发布）

**自动执行** — 克隆项目 + 安装依赖 + 生成 config.yaml

```bash
git clone https://github.com/SunnySLJ/openclaw_upload.git
# + Python venv + pip install
# + 生成 flash_longxia/config.yaml
```

**config.yaml 自动填入**：
- `content.industry` → 你的行业
- `content.video_style` → 你的视频风格
- `auto_generate_title: true` → 发布前自动生成标题
- `auto_generate_description: true` → 发布前自动生成文案
- `confirm_before_generate` → 是否需要人工确认
- `wechat_target: ""` → 留空，绑定微信后自动填写
- 飞书通知配置（如果步骤 5 填写了）

---

### 步骤 🔟：初始化 Workspace 配置文件

**自动执行** — 生成 7 个核心 md 文件

| 文件 | 内容 | 来源 |
|------|------|------|
| `IDENTITY.md` | AI 名字/表情/性格 | 步骤 6 用户输入 |
| `SOUL.md` | AI 行为准则 | 步骤 6 选择的风格模板 |
| `USER.md` | 用户偏好/行业 | 步骤 6 用户输入 |
| `MEMORY.md` | 9 条红线规则 + 工作流 + 内容生成规则 | 模板 |
| `TOOLS.md` | 路径/端口/命令 | 模板（路径自动替换） |
| `AGENTS.md` | 会话启动流程 | 模板 |
| `HEARTBEAT.md` | 定时任务文档 | 模板 |

> 已存在的文件不会被覆盖。

---

### 步骤 1️⃣1️⃣：安装 Skills（4 个技能）

**自动执行** — 复制技能到 `~/.openclaw/skills/`

| 技能 | 功能 |
|------|------|
| `flash-longxia` | 图片生成视频 |
| `auth` | 平台登录管理 |
| `longxia-upload` | 视频发布 |
| `longxia-bootstrap` | 项目引导 |

---

### 步骤 1️⃣2️⃣：配置 Memory 插件

**自动执行** — 创建目录结构

```
~/.openclaw/memory/        # 向量数据库
~/.openclaw/memory-md/     # Markdown 镜像
~/.openclaw/workspace/memory/  # 每日记忆
```

> 插件源码不随包分发，OpenClaw 首次启动时自动下载安装。

---

### 步骤 1️⃣3️⃣：创建定时任务

**交互式** — 配置 2 个定时任务

```
定时任务 1: 每日登录状态检查
  每天几点检查登录状态？(默认 10:10): 10:10

定时任务 2: 每周视频文件清理
  每周几清理视频文件？(默认 2=周二): 2
  几点执行清理？(默认 01:00): 01:00
```

| 任务 | 默认时间 | 功能 |
|------|---------|------|
| 登录检查 | 每天 10:10 | 检查视频号登录状态，失效则发送二维码 |
| 视频清理 | 每周二 01:00 | 清理 output 目录，保留 7 天内视频 |

---

### 步骤 1️⃣4️⃣：配置 Token 和微信推送

**交互式** — 输入视频生成 Token 和微信 Target

```
视频生成 API Token (留空跳过): [输入 Token]
微信 Target ID (留空跳过): [输入或留空]
```

> - Token 存储在 `flash_longxia/token.txt`
> - 微信 Target 自动写入 `config.yaml` 的 `wechat_target`
> - 留空则部署后手动配置

---

## 5. 部署后操作

### 5.1 启动 OpenClaw

```bash
openclaw
```

> 首次启动会自动初始化向量数据库，可能需要几分钟。

### 5.2 绑定微信（必须）

```bash
openclaw channel connect openclaw-weixin
```

出现二维码后，用微信扫码完成绑定。

> 绑定成功后，获取到的 `wechat_target` 会自动写入 `config.yaml`。

### 5.3 初始化对话

启动后，对 AI 说：

```
你好，帮我初始化环境
```

AI 会自动读取 MEMORY.md、SOUL.md 等文件，完成自我初始化。

### 5.4 安装技能

告诉 AI：

```
帮我安装 xiaolong-upload 和 openclaw_upload
```

### 5.5 验证视频流程

发一张图片给 AI，然后说：

```
帮我生成视频
```

验证完整流程：图片 → 生成视频 → 发送通知 → 确认发布 → **自动生成标题/文案/标签** → 上传视频号

---

## 6. 验证清单

部署完成后，逐项确认：

### 文件检查

- [ ] `~/.openclaw/openclaw.json` 存在，API Key 已填写
- [ ] `~/.openclaw/workspace/MEMORY.md` 存在，包含 9 条红线规则
- [ ] `~/.openclaw/workspace/SOUL.md` 存在
- [ ] `~/.openclaw/workspace/IDENTITY.md` 存在，AI 名字正确
- [ ] `~/.openclaw/workspace/USER.md` 存在，称呼和行业正确
- [ ] `~/.openclaw/workspace/TOOLS.md` 存在，路径正确
- [ ] `~/.openclaw/cron/jobs.json` 存在，定时任务配置正确

### 项目检查

- [ ] `~/.openclaw/workspace/xiaolong-upload/` 存在
- [ ] `~/.openclaw/workspace/openclaw_upload/` 存在
- [ ] `openclaw_upload/flash_longxia/config.yaml` 存在，行业/风格已填写
- [ ] `openclaw_upload/flash_longxia/token.txt` 存在（视频 Token）

### 技能检查

- [ ] `~/.openclaw/skills/flash-longxia/SKILL.md` 存在
- [ ] `~/.openclaw/skills/auth/SKILL.md` 存在
- [ ] `~/.openclaw/skills/longxia-upload/SKILL.md` 存在
- [ ] `~/.openclaw/skills/longxia-bootstrap/SKILL.md` 存在

### 功能验证

- [ ] `openclaw` 命令可以正常启动
- [ ] 微信已扫码绑定
- [ ] 发送一张图片，AI 能识别并提示生成视频
- [ ] 发布时 AI 能自动生成匹配风格的标题/文案/标签
- [ ] 定时登录检查在设定时间触发

---

## 7. 常见问题排查

### Q1: node/npm 未找到

```bash
# macOS
brew install node

# Windows - 从 nodejs.org 下载安装
```

### Q2: Python 3.12 未找到

```bash
# macOS
brew install python@3.12

# Windows - 从 python.org 下载安装，勾选「Add to PATH」和「py launcher」
```

### Q3: OpenClaw 启动后没有反应

检查 `openclaw.json` 中的 API Key 是否正确：
```bash
cat ~/.openclaw/openclaw.json | grep apiKey
```

### Q4: 微信绑定失败

```bash
# 重新尝试绑定
openclaw channel connect openclaw-weixin
```

确保微信客户端已登录，扫码时在同一网络环境。

### Q5: 视频生成 Token 过期

更新 Token 文件：
```bash
echo "新的Token" > ~/.openclaw/workspace/openclaw_upload/flash_longxia/token.txt
```

### Q6: 定时任务没有执行

检查 cron 配置：
```bash
cat ~/.openclaw/cron/jobs.json
```
确认 `enabled: true` 且时间格式正确。

### Q7: memory-lancedb-pro 启动报错

```bash
# 删除旧数据库重新初始化
rm -rf ~/.openclaw/memory/lancedb-pro*
# 重启 openclaw
```

### Q8: PowerShell 脚本无法运行

```powershell
# 允许执行脚本
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

# 如果仍有问题，右键 ps1 文件 → 属性 → 解除锁定
```

---

## 8. 迁移部署指南

从旧机器迁移到新机器时，选择部署模式 2（迁移部署）。

### 需要从旧机器备份的文件

| 文件 | 路径 | 说明 |
|------|------|------|
| 配置 | `~/.openclaw/openclaw.json` | LLM/插件配置 |
| 记忆 | `~/.openclaw/workspace/MEMORY.md` | 红线规则 + 经验教训 |
| 定时任务 | `~/.openclaw/cron/jobs.json` | cron 配置 |
| Token | `openclaw_upload/flash_longxia/token.txt` | 视频生成 Token |
| Cookies | `openclaw_upload/cookies/` | 平台登录状态 |
| 向量数据库 | `~/.openclaw/memory/` | 整个目录（可选） |

### 迁移步骤

```bash
# 1. 在旧机器打包需要的文件
tar -czf openclaw-backup.tar.gz \
  ~/.openclaw/openclaw.json \
  ~/.openclaw/workspace/MEMORY.md \
  ~/.openclaw/cron/jobs.json \
  ~/.openclaw/workspace/openclaw_upload/flash_longxia/token.txt

# 2. 将 backup + deploy-openclaw 目录传到新机器

# 3. 在新机器运行部署脚本（选择模式 2）
./deploy-openclaw.sh

# 4. 部署完成后，覆盖生成的 MEMORY.md（保留经验教训）
cp backup/MEMORY.md ~/.openclaw/workspace/MEMORY.md

# 5. 恢复 cookies（如果有）
cp -r backup/cookies/ ~/.openclaw/workspace/openclaw_upload/cookies/
```

---

## 9. 部署包文件清单

```
deploy-openclaw/                        # 部署包根目录
│
├── deploy-openclaw.sh                  # macOS 部署脚本 (Bash)
├── deploy-openclaw.ps1                 # Windows 部署脚本 (PowerShell)
├── README.md                           # 快速入门指南
├── SOP.md                              # 本文件（标准操作流程）
│
├── config/                             # 配置模板
│   ├── openclaw-bailian.json.template  # 百炼方案模板
│   ├── openclaw-n1n.json.template      # n1n.ai 方案模板 (GPT-4.1)
│   └── login_check_config.json         # 登录检查配置
│
├── workspace/                          # Workspace 文件模板
│   ├── AGENTS.md                       # 会话启动流程
│   ├── IDENTITY.md                     # AI 身份（部署时根据用户输入生成）
│   ├── SOUL.md                         # AI 灵魂（3 种风格模板）
│   ├── USER.md                         # 用户偏好（部署时根据用户输入生成）
│   ├── MEMORY.md                       # 红线规则 + 工作流 + 内容生成规则
│   ├── HEARTBEAT.md                    # 心跳/定时任务文档
│   └── TOOLS.md                        # 工具配置（路径自动替换）
│
└── skills/                             # 技能文件
    ├── flash-longxia/                  # 图片生成视频
    │   ├── SKILL.md
    │   ├── agents/openai.yaml
    │   └── scripts/
    │       ├── common.py
    │       ├── download_video.py
    │       ├── generate_video.py
    │       └── preflight.py
    │
    ├── auth/                           # 平台登录管理
    │   ├── SKILL.md
    │   ├── login_check_config.json
    │   └── scripts/
    │       ├── platform_login.py
    │       └── scheduled_login_check.py
    │
    ├── longxia-upload/                 # 视频发布
    │   └── SKILL.md
    │
    └── longxia-bootstrap/              # 项目引导
        ├── SKILL.md
        ├── project_config.json
        └── scripts/bootstrap.py
```

---

## ⚡ 快速参考卡片

```
┌──────────────────────────────────────────────────────────┐
│  OpenClaw 一键部署 — 快速参考                              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  macOS:    chmod +x deploy-openclaw.sh                   │
│            ./deploy-openclaw.sh                          │
│                                                          │
│  Windows:  .\deploy-openclaw.ps1                         │
│                                                          │
│  部署后:   openclaw                          # 启动       │
│            openclaw channel connect          # 绑微信     │
│              openclaw-weixin                             │
│                                                          │
│  版本:     OpenClaw 2026.3.28 (稳定版)                    │
│  LLM:      百炼 Coding Plan / n1n.ai (GPT-4.1)           │
│  Python:   3.12 (必须)                                    │
│                                                          │
│  更新技能: ~/.openclaw/workspace/update-skills.sh         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

> 📝 **维护说明**：本 SOP 随部署脚本版本一起更新。如果修改了部署脚本，请同步更新本文档。
