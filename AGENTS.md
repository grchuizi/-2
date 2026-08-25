# AGENTS.md — 给 AI 编码代理（Codex / Claude Code / 其它 Agent）

> 你（AI 代理）进入本项目后，先读本文件。它告诉你：这个项目是什么、怎么**在本地部署跑通**、
> 有哪些**绝不能碰的约束**。目标是一句话「帮我部署」就能成功，你自主完成、不需人工盯着。

## 1. 项目是什么
通用产业竞争情报自动化：采集新闻 → LLM 分析 → 生成周报 → 微信推送。
首主题「充电与户用储能」，通过 `config/topics/*.yaml` 换行业，核心代码行业无关。

## 2. 一键本地部署（优先读这个）
按 [`docs/10_DEPLOY_FOR_AI.md`](docs/10_DEPLOY_FOR_AI.md) 第 2~5 步执行。要点：

```bash
# 建隔离 .venv + 装依赖（只装到 .venv，绝不全局 pip install）
powershell -ExecutionPolicy Bypass -File scripts/01_Create-Isolated-Venv.ps1
.venv/Scripts/pip install -e ".[dev]"

# 探活（不需 key）
.venv/Scripts/python main.py --version
.venv/Scripts/python main.py --validate

# 跑完整链路（需两个 key 通过环境变量注入，见 docs/10 第 3 步）
.venv/Scripts/python main.py --topic charging_pile --task charging_cn_weekly --phase2 --phase3 --phase4 --notify true
```

## 3. 硬性约束（违反即失败）

1. **API Key 绝不入库**：`DEEPSEEK_API_KEY` / `SERVERCHAN_KEY` 只许经**环境变量**注入，
   严禁写入仓库文件、`.env`、`.env.example`、日志、报告、测试数据、git 提交记录。
2. **只装到 `.venv`**：禁止全局 `pip install`；禁止改系统/用户环境变量、PATH、注册表。
3. **只改仓库目录内文件**：禁止访问/修改项目外的用户文件、系统配置。
4. **key 变量名是 `SERVERCHAN_KEY`**（不是 `.env.example` 旧名 `SERVERCHAN_SENDKEY`，用错推送静默失败）。
5. **行业数据放 config，核心代码零行业硬编码**：换行业改 `config/topics/*.yaml`，不重写 src/。

## 4. 你要替部署者确认的
- Python 3.11+/3.12 是否存在；`.venv` 是否已存在（存在则复用，别重建）。
- 部署者是否通过环境变量提供了两个 key。若没有：采集仍能跑（降级），但报告分析/推送会空。
  此时应**明确告知部署者**「缺 key，完整效果需提供」，而不是静默当作部署失败。

## 5. 成功标准
`--validate` 输出 `Validation OK` = 环境/配置就绪；完整链路日志出现 `[success]` 且结果含
`notified` = 推送成功。任何一步失败应把**实际报错**报告给部署者，而不是掩盖。
