---
title: CHATSAM - ChatGPT2API with Data Persistence
emoji: 🎨
colorFrom: blue
colorTo: purple
sdk: docker
app_port: 7860
pinned: false
---

<h1 align="center">CHATSAM - ChatGPT2API</h1>

<p align="center">一键部署到 HuggingFace Spaces，支持数据持久化 + 防止休眠</p>

<p align="center">
  <a href="#-项目说明">📖 项目说明</a> •
  <a href="#-一键部署">🚀 一键部署</a> •
  <a href="#-数据持久化">📦 数据持久化</a> •
  <a href="#-防止休眠">⚡ 防止休眠</a> •
  <a href="#-需要修改的地方">🔧 配置修改</a>
</p>

---

## 📖 项目说明

### 项目来源

本项目基于 **[basketikun/chatgpt2api](https://github.com/basketikun/chatgpt2api)** 进行二次开发和优化。

**原项目功能**：
- ChatGPT 官网图片生成、图片编辑能力的逆向封装
- OpenAI 兼容图片 API / 代理
- 在线画图、号池管理、多种账号导入方式
- Docker 自托管部署能力

**本项目新增功能**：
- ✅ **数据持久化** - 支持 Git 仓库存储，防止数据丢失
- ✅ **防止休眠** - GitHub Actions 自动保活，24小时在线
- ✅ **一键部署** - 完整部署文档，快速部署到 HuggingFace
- ✅ **普通用户额度显示** - 修复普通用户不显示剩余额度的问题

### 致谢

感谢以下项目和开发者：

| 项目/开发者 | 贡献 |
|-------------|------|
| **[basketikun](https://github.com/basketikun)** | 原项目作者，核心功能开发 |
| **[chatgpt2api Contributors](https://github.com/basketikun/chatgpt2api/graphs/contributors)** | 所有贡献者 |
| **[LinuxDO](https://linux.do)** | 社区支持 |

> 如果这个项目对你有帮助，请给原项目 [basketikun/chatgpt2api](https://github.com/basketikun/chatgpt2api) 一个 ⭐ Star！

---

## 🚀 一键部署到 HuggingFace

### 步骤 1：创建 HuggingFace Space

1. 登录 [HuggingFace](https://huggingface.co)
2. 点击右上角 **"Create Space"**
3. 填写信息：
   - **Space name**: 你的项目名称（如 `my-chatsam`）
   - **SDK**: 选择 **Docker**
   - **Hardware**: 选择 **CPU basic**（免费）
4. 点击 **"Create Space"**

### 步骤 2：克隆此项目并推送

```bash
# 克隆此项目
git clone https://github.com/chanzsam/CHATSAM-Public.git
cd CHATSAM-Public

# 添加 HuggingFace 远程仓库
git remote add hf https://huggingface.co/spaces/<你的HF用户名>/<你的Space名称>

# 推送到 HuggingFace
git push hf main
```

### 步骤 3：配置管理员密码

在 Space 的 **Settings → Variables and secrets** 中添加：

| Secret 名称 | Secret 值 |
|------------|-----------|
| `CHATGPT2API_AUTH_KEY` | `你的管理员密码`（建议使用强密码） |

---

## 📦 数据持久化（防止数据丢失）

### 为什么需要数据持久化？

HuggingFace Spaces 使用 **临时存储 (Ephemeral Storage)**：
- ⚠️ 48 小时无访问 → 容器休眠 → 数据丢失
- ⚠️ 推送新代码 → 容器重建 → 数据丢失
- ⚠️ 资源限制重启 → 数据丢失

**解决方案**：使用 Git 仓库存储后端，数据永久保存！

### 配置步骤

#### 1️⃣ 创建 GitHub 数据存储仓库

1. 在 GitHub 创建一个新的**私有仓库**（如 `my-chatsam-data`）
2. 在仓库中创建以下文件：

**accounts.json**
```json
{
  "items": []
}
```

**auth_keys.json**
```json
{
  "items": []
}
```

3. 推送到 GitHub

#### 2️⃣ 创建 GitHub Personal Access Token

1. 访问 [GitHub Token 设置](https://github.com/settings/tokens)
2. 点击 **"Generate new token (classic)"**
3. 填写信息：
   - **Note**: `CHATSAM Data Storage`
   - **Expiration**: `No expiration`（或选择较长时间）
   - **Select scopes**: 选择 **`repo`**（完整仓库访问）
4. 点击 **"Generate token"**
5. ⚠️ **复制 Token**（只显示一次，请妥善保存）

#### 3️⃣ 配置 HuggingFace Secrets

在 Space 的 **Settings → Variables and secrets** 中添加：

| Secret 名称 | Secret 值 | 说明 |
|------------|-----------|------|
| `STORAGE_BACKEND` | `git` | 使用 Git 存储 |
| `GIT_REPO_URL` | `https://github.com/<你的用户名>/<数据仓库名>.git` | 数据仓库地址 |
| `GIT_TOKEN` | `ghp_xxxxxxxxxxxx` | GitHub Token |

#### 4️⃣ 重启 Space

添加 Secrets 后：
1. 回到 Space 主页
2. 点击右上角 **"⋯"** → **"Factory reboot"**
3. 等待约 5-10 分钟重新构建

#### 5️⃣ 验证数据持久化

1. 登录你的 Space
2. 添加一个测试用户
3. 检查 GitHub 数据仓库是否有新提交
4. 重启 Space，验证数据是否保留

---

## ⚡ 防止休眠（24小时在线）

### 为什么需要防止休眠？

HuggingFace Spaces 免费 48 小时无访问会自动休眠：
- 休眠后冷启动需要 **1-3 分钟**
- 用户体验不佳

**解决方案**：使用 GitHub Actions 每 12 小时自动访问！

### 配置步骤

#### 1️⃣ Fork 此项目到你的 GitHub

访问 https://github.com/chanzsam/CHATSAM-Public 并点击 **"Fork"**

#### 2️⃣ 启用 GitHub Actions

1. 进入你的 Fork 仓库
2. 点击 **Settings → Actions → General**
3. 选择 **"Allow all actions and reusable workflows"**
4. 点击 **"Save"**

#### 3️⃣ 修改保活 Workflow

编辑 `.github/workflows/keep-alive.yml`：

```yaml
name: Keep HuggingFace Space Alive

on:
  schedule:
    - cron: '0 */12 * * *'  # 每12小时执行一次
  workflow_dispatch:         # 支持手动触发

jobs:
  keep-alive:
    runs-on: ubuntu-latest
    steps:
      - name: Ping HuggingFace Space
        run: |
          # ⚠️ 请修改为你的 Space 地址
          curl -s https://huggingface.co/spaces/<你的HF用户名>/<你的Space名称>
          curl -s https://<你的HF用户名>-<你的Space名称>.hf.space
      - name: Log status
        run: echo "Keep-alive ping sent at $(date)"
```

**需要修改的地方**：
- `<你的HF用户名>` → 你的 HuggingFace 用户名
- `<你的Space名称>` → 你的 Space 名称

#### 4️⃣ 推送修改

```bash
git add .github/workflows/keep-alive.yml
git commit -m "update: keep-alive workflow URL"
git push
```

#### 5️⃣ 手动测试

1. 进入仓库 **Actions** 页面
2. 点击 **"Keep HuggingFace Space Alive"**
3. 点击 **"Run workflow"** → **"Run workflow"**
4. 查看运行结果

---

## 🔧 需要修改的地方

### 📋 修改清单

部署前需要修改以下文件：

| 文件 | 修改内容 | 必需 |
|------|---------|------|
| `.github/workflows/keep-alive.yml` | 修改 HF Space 地址 | ✅ 必需 |
| `config.json` | 修改管理员密码（或使用 Secrets） | ✅ 必需 |
| HF Secrets | 配置存储后端 | 推荐 |

---

### 1️⃣ `.github/workflows/keep-alive.yml`

**位置**: `.github/workflows/keep-alive.yml`

**需要修改**:

```yaml
# 第 15-16 行，修改为你的 Space 地址
curl -s https://huggingface.co/spaces/<YOUR_HF_USERNAME>/<YOUR_SPACE_NAME>
curl -s https://<YOUR_HF_USERNAME>-<YOUR_SPACE_NAME>.hf.space
```

**示例**:

如果你的 HF 用户名是 `myuser`，Space 名称是 `my-chatsam`：

```yaml
curl -s https://huggingface.co/spaces/myuser/my-chatsam
curl -s https://myuser-my-chatsam.hf.space
```

---

### 2️⃣ `config.json`

**位置**: `config.json`

**需要修改**:

```json
{
  "auth-key": "YOUR_SECRET_KEY_HERE"  // ← 修改为你的管理员密码
}
```

**或者使用 Secrets（推荐）**:

在 HF Space Settings 中添加 `CHATGPT2API_AUTH_KEY`，会覆盖 `config.json` 中的设置。

---

### 3️⃣ HuggingFace Secrets

**位置**: Space Settings → Variables and secrets

**需要添加**:

| Secret 名称 | 值示例 | 说明 |
|------------|-------|------|
| `CHATGPT2API_AUTH_KEY` | `MyStrongPassword123!` | 管理员密码 |
| `STORAGE_BACKEND` | `git` | 存储后端类型 |
| `GIT_REPO_URL` | `https://github.com/myuser/my-data.git` | 数据仓库地址 |
| `GIT_TOKEN` | `ghp_xxxxxxxxxxxx` | GitHub Token |

---

## ✨ 功能特性

### API 兼容能力

- 兼容 `POST /v1/images/generations` 图片生成接口
- 兼容 `POST /v1/images/edits` 图片编辑接口
- 兼容面向图片场景的 `POST /v1/chat/completions`
- 兼容面向图片场景的 `POST /v1/responses`
- `GET /v1/models` 返回可用模型列表

### 在线画图功能

- 内置在线画图工作台
- 支持 `gpt-image-2`、`codex-gpt-image-2`、`auto` 等模型
- 编辑模式支持参考图上传
- 本地保存图片会话历史

### 号池管理功能

- 自动刷新账号邮箱、类型、额度
- 轮询可用账号执行图片生成
- 自动剔除无效 Token
- 支持多种导入方式

### 数据持久化

- 支持 Git 仓库存储
- 支持 PostgreSQL 存储
- 支持 SQLite 存储
- 配置、账号、用户数据永久保存

---

## 📋 部署清单

| 配置项 | 说明 | 必需 | 配置方式 |
|--------|------|------|---------|
| `CHATGPT2API_AUTH_KEY` | 管理员密码 | ✅ 必需 | HF Secrets |
| `STORAGE_BACKEND` | 存储后端类型 | 推荐 | HF Secrets |
| `GIT_REPO_URL` | Git 数据仓库地址 | 推荐 | HF Secrets |
| `GIT_TOKEN` | GitHub Token | 推荐 | HF Secrets |
| `keep-alive.yml` | 保活 URL | ✅ 必需 | 修改文件 |

---

## 🔧 本地开发

### Docker 运行

```bash
docker compose up -d
```

访问：`http://localhost:7860`

### 本地开发

```bash
# 后端
uv sync
uv run main.py

# 前端
cd web
bun install
bun run dev
```

---

## 📸 项目截图

> ⚠️ 截图待补充，你可以访问 [原项目](https://github.com/basketikun/chatgpt2api) 查看更多截图。

### 界面预览

| 功能 | 说明 |
|------|------|
| **文生图界面** | 输入提示词生成图片 |
| **编辑图界面** | 上传图片进行编辑 |
| **号池管理** | 管理账号和额度 |
| **用户管理** | 创建普通用户账号 |

---

## ⚠️ 免责声明

> 本项目涉及对 ChatGPT 官网相关能力的逆向研究，仅供个人学习、技术研究与非商业性技术交流使用。
>
> - 严禁用于任何商业用途
> - 严禁用于违反 OpenAI 服务条款的行为
> - 严禁用于生成违法内容
> - 使用者应自行承担全部风险

---

## 📄 License

MIT License

---

## 🙏 致谢

### 原项目

本项目基于 [basketikun/chatgpt2api](https://github.com/basketikun/chatgpt2api) 开发。

### 贡献者

<a href="https://github.com/basketikun/chatgpt2api/graphs/contributors">
  <img alt="Contributors" src="https://contrib.rocks/image?repo=basketikun/chatgpt2api" />
</a>

### 社区

学 AI，上 L 站：[LinuxDO](https://linux.do)

---

## 📌 Star History

如果这个项目对你有帮助，请给原项目一个 Star！

[![Star History Chart](https://api.star-history.com/chart?repos=basketikun/chatgpt2api&type=date)](https://star-history.com/)