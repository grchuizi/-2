# Industry Intelligence Agent

通用产业竞争情报自动化 Agent。

> 🧭 快速上手：
> - **让 AI 代理（Codex/Claude）一键本地部署？** → [`docs/10_DEPLOY_FOR_AI.md`](docs/10_DEPLOY_FOR_AI.md)（AI 读了就能自主建 venv、装依赖、跑通并推送）
> - **手动本地部署 / 换电脑？** → [`换电脑安装指南.md`](换电脑安装指南.md)
> - **换关键词 / 换行业？** → [`如何换关键词.md`](如何换关键词.md)
> - **配置你自己的 API Key？** → [`换电脑安装指南.md`](换电脑安装指南.md)「配 API Key」（仓库不含任何作者密钥，用自己的）
> - **Fork 后用不了 / 看不到 Run workflow？** → [`换电脑安装指南.md`](换电脑安装指南.md)「场景二：fork 一份」（fork 后要先启用 Actions，再配你自己的 Secret）

## 1. 项目名称

Industry Intelligence Agent（industry-intelligence-agent）。

## 2. 项目目标

构建一套无网站、低人工干预、可配置、可追溯、可扩展的产业竞争情报自动化系统。

通过 `Topic Profile + Task Config` 双层配置切换行业、企业、地区、时间范围与关注指标，核心代码不绑定具体行业。首个验证主题为中国充电基础设施，第二验证主题为户用储能。

## 3. 当前阶段

- **阶段：** Phase 0 — 项目骨架与规范（本阶段完成工程骨架、环境隔离、基础测试与 CI）。
- **业务代码：** 尚未开始。
- 更多规划见 `docs/00_MASTER_PLAN.md` 与 `ROADMAP.md`。

## 4. 核心设计原则

- 核心代码行业无关：行业知识通过 `config/topics/*.yaml` 配置。
- 数据源可插拔：新数据源通过 Source Adapter 扩展。
- LLM 通过 Provider 抽象调用，不绑定单一厂商或模型。
- 通知通过 Notifier 抽象，渠道可替换。
- 事实与推断分离：`FACT` / `INFERENCE` / `FORECAST` 必须区分。
- 证据优先：关键结论必须回链到可追溯来源。

## 5. 目录结构

```text
├── .github/workflows/       GitHub Actions（当前仅 CI）
├── config/                  配置（system/topics/tasks/sources/prompts）
├── src/industry_intelligence/  核心 Python 包
├── tests/                   unit / integration / fixtures
├── data/                    raw / normalized / intelligence / state / cache
├── reports/                 报告输出
├── logs/                    日志
├── scripts/                 PowerShell 环境脚本
├── docs/                    架构文档与 Phase 指令
├── pyproject.toml
├── main.py                  基础入口
└── README.md
```

## 6. Python 环境要求

- Python >= 3.11（优先 3.12；本机如无 3.12，可用 3.11）。

## 7. `.venv` 使用方法

项目内创建隔离虚拟环境（禁止全局安装）：

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

或运行：

```powershell
.\scripts\01_Create-Isolated-Venv.ps1
```

## 8. 开发依赖安装方式

只安装到 `.venv`：

```powershell
.\.venv\Scripts\pip install -e ".[dev]"
```

开发依赖仅包括：`pytest`、`pytest-cov`、`ruff`、`mypy`。

## 9. 测试命令

```powershell
.\.venv\Scripts\python main.py
.\.venv\Scripts\pytest
.\.venv\Scripts\ruff check .
.\.venv\Scripts\mypy src
```

## 10. GitHub 全自动运行（Phase 5）

### 10.1 本地验证调度器

```powershell
.\.venv\Scripts\python scheduler.py --validate-schedules   # 校验 config/schedules.yaml
.\.venv\Scripts\python scheduler.py --dry-run              # 列出今天到期任务
.\.venv\Scripts\python scheduler.py --run-due --dry-run    # 同上
```

### 10.2 首次接入步骤（一次性）

1. **推送代码到 GitHub** 并绑定远端：`git remote add origin <你的仓库 URL>` → `git push -u origin main`。
2. **配置 Secrets**（Settings → Secrets and variables → Actions）：
   - `DEEPSEEK_API_KEY` — DeepSeek API Key（配置 `llm.api_key_env`）
   - `SERVERCHAN_KEY` — Server酱 SendKey（配置 `notification.serverchan_key_env`）
   - 密钥只在 GitHub 端保存，仓库内 `.env.example` 只有变量名，日志/报告不输出密钥。
3. **手动触发一次**：Actions → `Manual Run` → Run workflow，输入任务 id（默认 `charging_cn_weekly`）。
4. **验收标准**：无本地电脑参与完成一次全流程（采集 → 分析 → 审查 → 报告），并在微信收到摘要推送。

### 10.3 Fork 后启用 Actions（别人 Clone 用不了？先看这里）

**如果你是从别人的仓库 fork 过来的**：fork 后 GitHub 会**默认禁用 Actions**（安全策略，fork 视为不信任代码）。所以你会看到：

- Actions 标签页里所有 workflow 是**灰色/禁用**状态，或者干脆看不到 `Manual Run`、`Run workflow` 按钮不出现。
- **这不是仓库/代码有问题，是 fork 默认冻结了 Actions**，需要手动解冻。

**两步启用（在你 fork 的那个仓库里操作，不是原作者仓库）：**

1. **Allow Actions**：`Settings → Actions → General → Actions permissions` → 选 **"Allow all actions and reusable workflows"**（默认常常是 Disable / Allow select）。
2. **给读写权限**：同一页的 **Workflow permissions** → 选 **"Read and write permissions"**（本仓库 workflow 要 `contents: write` 来自动提交数据）。
3. **启用 workflow**：回到 `Actions` 页签 → 左列找到 `manual_run.yml` → 点进去 → 右上角 **"Enable workflow"** 按钮 → 点一下。

完成后 `Actions → Manual Run → Run workflow` 的按钮就会出现。

> ⚠️ **Secret 配在哪？** 配在**你 fork 的那个仓库**（Settings → Secrets and variables → Actions）。GitHub **故意不复制**原仓库的 Secret 到 fork，所以原作者的两个 key 不会、也不该出现在你的 fork 里——**你要用自己的**：
> - `DEEPSEEK_API_KEY` — 你自己的 DeepSeek API Key
> - `SERVERCHAN_KEY` — 你自己的 Server酱 SendKey
> - 注意变量名是 **`SERVERCHAN_KEY`**（不是 `.env.example` 里旧的 `SERVERCHAN_SENDKEY`），配错会导致推送静默失败。

### 10.4 工作流

| 工作流 | 触发 | 作用 |
|--------|------|------|
| `scheduled_dispatcher.yml` | 每日北京 08:17（cron） | 按 `config/schedules.yaml` 运行到期任务，提交数据，上传报告 |
| `manual_run.yml` | `workflow_dispatch` | 手动跑指定任务，可覆盖地区/企业/核心词/时间窗口 |
| `validation.yml` | 每日 | `main.py --validate` + `scheduler.py --validate-schedules`，失败微信告警 |
| `maintenance.yml` | 每周六 | SQLite 完整性检查 + 清理 90 天前报告 |

所有运行共享同一并发组，避免同一天多次写同一 SQLite；失败任务自动重试一次并推送微信告警。

### 10.5 持久化数据

- `data/state/industry_intelligence.sqlite` — 跨运行积累的查询层（历史比较的事实源），随运行提交回仓库。
- `data/collection.jsonl` — 规范化文档审计层（追加式）。
- `data/state/scheduler_state.json` — 调度幂等状态（当天已运行不重复触发）。
- `output/reports/<run_id>/` — 每轮 Markdown / Excel / 微信摘要报告。

## 11. 当前禁止事项

Phase 0 阶段禁止实现：真实爬虫、搜索/新闻/RSS 抓取、SQLite 业务模型、Topic/Task/Source/Evidence Schema、LLM API、Server酱/企业微信、Markdown/Excel 周报、GitHub cron、历史比较、竞争指数、事件聚类、实体识别。

禁止操作：修改项目目录外文件、全局 pip 安装、修改 Windows PATH/注册表/系统环境变量、创建计划任务/系统服务、复用日常浏览器 Profile、在仓库中写入任何 API Key/Token。

## 13. 数据源与网页搜索（v0.7.0a1 起；v0.7.0a5 起热点优先；v0.7.0a6 起推送反映热点；v0.7.0a7 起企业节质量；v0.7.0a8 起推送完整优先）

采集采用组合适配器（RSS 基线 + 网页搜索并行，websearch 禁用/失败时自动回退 RSS-only）。**v0.7.0a5 起，关键词只是大方向锚点**：检索优先用 LLM 动态发现的当前行业热点，固定三族查询降级为兜底。

- **RSS**：`config/sources/search.yaml -> rss_feeds`。
- **网页搜索（无 API Key、无账号）**：`WebSearchAdapter` 直接抓取 Bing 搜索结果页（`config/sources/search.yaml -> websearch.engines`），解析结果块生成采集条目；查询间礼貌限速（`delay_seconds`，缺省用 `collection.polite_delay_seconds`），失败退避重试（`collection.retries`）。
- **动态热点发现（v0.7.0a5）**：`HotTopicGenerator` 用 DeepSeek 基于大方向词（core，或 `--focus` 覆盖）生成当前行业热点话题短语；`SearchPlanner` 热点可用时优先生成 `family="hot"` 查询（热点短语 × 地区），热点为空/失败时回退固定三族。开关与上限：`config/system.yaml -> collection.hot_topics_enabled` / `hot_topics_max`。
- **固定三族查询（兜底）**：
  - **企业族**：核心词 × 跟踪企业 × 地区；
  - **事件族**：核心词 × 事件词 × 地区；
  - **权威官方站点检索**：每个行业在 `config/topics/<topic>.yaml` 的 `official_domains` 声明其权威官网域名（如 `gov.cn` / `ndrc.gov.cn` / `miit.gov.cn` / `nea.gov.cn`），Planner 自动生成 `"核心词 site:域名"` 查询族；命中官方站点的文档在 `extra.official_domain` 标记，供 JSONL/SQLite 追溯。检索哪些官网由该行业关键词决定，核心代码零行业硬编码。
- **热点与门控联动**：当次热点短语自动并入相关性门控信号词（`build_relevance_terms(extra=…)`），保证按热点检索到的文档不被误杀；热点为空时回退行为与 v0.7.0a3 完全一致。
- **推送反映热点（v0.7.0a6）**：热点短语贯通到报告层——微信摘要顶部新增"本期热点关注"行（最多 3 条），`最重要的 5 件事` 里命中热点的条目自动排到前面；企业竞争变化改为**内容优先**——事件/Claim 中出现的企业（含未跟踪企业）先展示真实动态，无内容企业不再占用位置，仅在有动态企业不足 3 行时用"本期暂无动态"补足。
- **企业节质量（v0.7.0a7）**：分析 prompt 放宽 `entity_id` 规则，允许标注采集数据中实际出现的未跟踪企业（如"蔚来/比亚迪"）；`ReportDataBuilder` 对 entity_id 做别名归一（映射来自 TopicProfile，零硬编码）；微信摘要分节预算——一句话判断/5 件事标题/企业节动态分别限长，保证企业节在 600 字内不被前两节挤掉。
- **推送完整优先（v0.7.0a8）**：不再"话说一半"——`DigestFormatter` 语义截断（超长优先截到句子结束符，完整句子不加省略号）；整体超 600 字时**先减条数**（5 件事 5→4→3、企业节 5→4→3，均不低于 3 条），条目控制在 3~5 之间而非截断单条；报告链接改为相对仓库根路径（`output/reports/<run_id>/report.md`），省掉 GitHub runner 绝对路径约 60 字预算。
- **边界**：只访问公开 HTML 页面，礼貌限速，不绕过登录/验证码/付费墙。后续版本（V0.8+）按需接入个别官网站内自建搜索。

## 12. Roadmap

- 阶段总览：`ROADMAP.md`
- 系统总设计：`docs/00_MASTER_PLAN.md`
- 当前阶段指令：`docs/phase_instructions/PHASE_0_BOOTSTRAP.md`
- 项目宪法与安全边界：`CLAUDE.md`、`SECURITY_BOUNDARY.md`
