# 部署指南（本地跑通）

> **一份文档，两种部署方式，任选其一：**
> - **方式 A — AI 代理一键部署**：用 Codex / Claude 等，读了就能自主建 venv、装依赖、跑通并推送。
> - **方式 B — 手动部署**：人自己按步骤操作，不依赖 AI 代理。
>
> 两条路径最终效果一样：**本地跑通完整链路并微信推送**。
> 不依赖 GitHub Actions 权限 / Secret（唯一人工步骤是提供两个 API key）。
> 要把系统装进 GitHub Actions 云端自动每天跑，见 README 10.3（需在仓库手动录入 Secret）。

---

## 方式 A — AI 代理一键部署（给 Codex / Claude / 任何 Agent）

把下面这句话丢给 AI 代理：

> 读取 `docs/10_DEPLOY_FOR_AI.md`，按本文件方式 A 在本仓库本地部署：
> 建 venv、装依赖、`--validate` 验证、跑 `charging_cn_weekly` 完整链路并推送。
> 前置检查：Python 3.11+/3.12 是否存在；`.venv` 已存在则复用。key 由部署者通过环境变量提供，
> 严禁写入仓库文件、日志或提交记录。

代理会自主完成下面的第 2、3、4、5 步（唯一需要部署者介入的是「配 key」章节）。`AGENTS.md`
也让代理一进项目就自动知道这些约束与指令。

---

## 方式 B — 手动部署（给人自己操作）

### 前置要求

- **Python 3.11+ 或 3.12**（默认 3.12）。检查：`python --version` 或 `py -3.12 --version`。
- **PowerShell 可用**（Windows，脚本是 `.ps1`）。
- **git**。

---

### 第 1 步 — 建隔离环境 + 装依赖（禁止全局安装）

项目铁律：Python 依赖**只装到项目内 `.venv`**，绝不全局 `pip install`。

```powershell
# 建 .venv（已存在会自动跳过）
powershell -ExecutionPolicy Bypass -File scripts/01_Create-Isolated-Venv.ps1

# 装包 + 开发依赖
.venv\Scripts\pip install -e ".[dev]"
```

> 若 `01_Create-Isolated-Venv.ps1` 失败（提示无 python），先安装 Python 3.12，重试。
> 依赖只有：PyYAML / requests / feedparser / openai / openpyxl（运行时），外加
> pytest / ruff / mypy / types-PyYAML（dev）。本机联网即可装，无需其他系统依赖。

---

### 第 2 步 — 配 key（唯一需要你动手的步骤）

系统跑完整链路需要两个 API key，**都通过环境变量注入，绝不写进任何文件**：

| 环境变量 | 用途 | 去哪申请 |
|---|---|---|
| `DEEPSEEK_API_KEY` | LLM 分析 / 热点发现 / 早报提炼 / 企业发现的 key | https://platform.deepseek.com → API Keys |
| `SERVERCHAN_KEY` | 微信推送（Server酱 SendKey） | https://sct.ftqq.com → 微信扫码 → 拿到 SendKey |

**注意：变量名是 `SERVERCHAN_KEY`**（不是 `.env.example` 里的旧名 `SERVERCHAN_SENDKEY`，用错会静默推送失败）。

**临时设置（当前终端会话）：**
```powershell
$env:DEEPSEEK_API_KEY = "你的DeepSeekKey"
$env:SERVERCHAN_KEY    = "你的Server酱SendKey"
```

> **重要**：真实 key 严禁写入仓库文件、`.env`、`.env.example`、日志、报告、测试数据。
> 只存在**当前终端会话的环境变量**里；跨会话需重新设置。

**不配 key 也能跑（降级）**：没有 `DEEPSEEK_API_KEY` 时，LLM 分析/热点/早报/企业发现退化为空，
采集仍能跑（回退固定三族检索），但报告分析部分为空；没有 `SERVERCHAN_KEY` 时不推送微信。
所以要完整效果必须配全。

---

### 第 3 步 — 验证部署成功（探活）

```powershell
.venv\Scripts\python main.py --version        # 应显示版本号（如 0.7.0a11）
.venv\Scripts\python main.py --validate       # 应显示配置合法（Validation OK）
.venv\Scripts\python -m pytest -q             # 可选：全量单测（应全部通过）
```

- `--version` / `--validate` **不需要 key**，跑通即说明环境、依赖、配置正常。
- 这两个通过 = 部署已基本成功。

---

### 第 4 步 — 跑完整链路（含微信推送）

```powershell
.venv\Scripts\python main.py --topic charging_pile --task charging_cn_weekly --phase2 --phase3 --phase4 --notify true
```

- `--topic charging_pile`：主题（充电与户用储能）。
- `--task charging_cn_weekly`：任务。
- `--phase2 --phase3 --phase4`：采集 → 分析 → 报告。
- `--notify true`：推送微信摘要（需要 `SERVERCHAN_KEY`）。
- 输出会写到 `data/state/industry_intelligence.sqlite` / `data/collection.jsonl` / `output/reports/<run_id>/`。

**成功特征**：日志尾部出现 `[success]`（结果里会写 `notified` 表示推送成功）。
若记录 `[partial]` 但仍生成报告 → 只是某个 LLM 步骤偶发降级（如 review 无效 JSON），不影响主体。

**想换主题/关键词/时间窗口**：
```powershell
.venv\Scripts\python main.py --topic charging_pile --task charging_cn_weekly --focus 换电站,无线充电 --notify true
```

---

## 常见坑

1. **`.venv` 不识别**：目录不对，或没用 `01_` 脚本建（`01_` 脚本会 `Resolve-Path`，须在仓库根跑）。
2. **`SERVERCHAN_SENDKEY` vs `SERVERCHAN_KEY`**：变量名以代码 `config/system.yaml ->
   notification.serverchan_key_env` 为准，是 `SERVERCHAN_KEY`。
3. **跑链路网络问题**：采集会访问 RSS/搜索源/DeepSeek API，需联网；公司内网/代理可能失败，报错见日志 `errors` 字段。
4. **Python 版本**：`requires-python >= 3.11`，用 3.12 最稳。
5. **想要全自动再跑**：`--notify true` + 配好 key 即可；要把「每天自动跑」接到 GitHub Actions，
   属云端部署（README 10.3），需人工在仓库录入 Secret（AI 代理无账号权限不能代做）。
