# AI Berkshire — 项目指令

## 项目概述

基于 Claude Code 的价值投资研究 Skill 合集。四大师框架：巴菲特、芒格、段永平、李录。
GitHub: xbtlin/ai-berkshire

## 项目结构

```
skills/          — 投研 Skill 定义（.md），复制到 ~/.claude/commands/ 使用
tools/           — 辅助工具（取数、精确计算、选股筛、报告抽检、回测），详见「工具清单」
reports/         — 投资研究报告输出
assets/          — 图片等静态资源
logs/            — 指令日志（log-command.sh 写入，不入库）
```

## 工具清单

`tools/` 下共 11 个脚本。除 `xueqiu_scraper.py`（需 Playwright）外**全部零外部依赖**，仅用 stdlib。

### 取数

| 工具 | 说明 | 用法 |
|------|------|------|
| `ashare_data.py` | A股数据——腾讯行情 + 东方财富搜索/财务 | `quote` / `financials` / `valuation` / `search` |
| `twstock_data.py` | 台股数据——FinMind 开放 API，覆盖上市+上柜。金额单位新台币 | `quote` / `valuation` / `financials` / `revenue`（月营收）/ `dividend` / `search` |
| `morningstar_fair_value.py` | 抓 Morningstar 筛选器全部公允价值估计，算潜在涨幅，输出 Top 100 | 直接运行，输出到 `data/` |
| `xueqiu_scraper.py` | 雪球用户时间线爬虫，按关键词筛本人原发言（滤掉纯转发）。Playwright 登录态复用、断点续爬 | `--user-id` `--keywords` `--output` |

### 校验

| 工具 | 说明 | 用法 |
|------|------|------|
| `financial_rigor.py` | 精确 Decimal 计算，无浮点漂移 | `verify-market-cap` / `verify-valuation` / `cross-validate` / `benford` / `calc` |
| `report_audit.py` | 报告数据抽检：抽 15% 财务数据点与可靠信源比对，出准出/打回判决 | `extract` → Claude 取数填值 → `verdict` |

### 选股与回测

| 工具 | 说明 |
|------|------|
| `stock_screener.py` | 动量发现 + 价值验证选股筛。第一层 60日新高+放量 → 第二层 6维评分 ≥3/6 买入。信号分级：3/6 试探仓3% \| 4/6 标准仓5% \| 5-6/6 确信仓8% |
| `momentum_backtest.py` | 上述框架回测（NVDA/AMD/MU，2022-01~2025-12），Yahoo Chart API 取价 |
| `momentum_backtest_v2.py` | 回测 v2——NVDA 关键节点手工录入（Yahoo API 被限流），AMD/MU 读本地 JSON 日线 |

### 其他

| 工具 | 说明 |
|------|------|
| `star_history_chart.py` | 复刻 star-history.com 的 star 增长曲线 SVG（浅色/深色两版）。GitHub 已限制 stargazers 匿名读取，故自备令牌生成。依赖素材 `xkcd.woff`、`star-history-logo.png` |
| `log-command.sh` | `user_prompt_submit` hook，把用户指令追加到 `logs/command-log.jsonl` |

**凭据一律走环境变量或 `local/`（已 .gitignore），严禁提交到 git**：`FINMIND_TOKEN`（台股，可选）、`XQ_PHONE`/`XQ_PASSWORD`（雪球）、`GITHUB_TOKEN`（star 图表）。

## 报告目录结构

所有报告按**公司名**建文件夹，公司相关的所有报告放在对应文件夹内：

```
reports/
├── AI产业研究/              — AI产业链全景研究（置顶）
│   ├── AI五层蛋糕-产业全景研究-20260605.md
│   └── AI五层蛋糕-公众号-20260605.md
├── 腾讯/                    — 腾讯所有研究报告
│   ├── 腾讯-research-20260408.md
│   ├── 腾讯-earnings-2025Q4.md
│   ├── 腾讯-management-20260409.md
│   └── 腾讯-thesis.md
├── 拼多多/                  — 拼多多所有研究报告
├── 泡泡玛特/                — 泡泡玛特所有研究报告
├── 核电-industry-20260409.md — 行业报告放根目录
├── AI算力-funnel-20260509.md  — 漏斗筛选报告放根目录
├── AI-轮动判断-20260509.md    — 主题级综合判断报告放根目录
├── portfolio-latest.md       — 组合报告放根目录
└── 多公司对比-checklist-20260408.md — 多公司报告放根目录
```

## 报告命名规范

| Skill | 文件命名格式 | 示例 |
|------|---------|------|
| /investment-team | `{公司名}/` 目录内含4个视角+最终报告 | `reports/拼多多/最终报告.md` |
| /investment-research | `{公司名}-research-{YYYYMMDD}.md` | `reports/腾讯/腾讯-research-20260408.md` |
| /investment-checklist | `{公司名}-checklist-{YYYYMMDD}.md` | `reports/腾讯/腾讯-checklist-20260408.md` |
| /industry-research | `{行业名}-industry-{YYYYMMDD}.md`（根目录） | `reports/核电-industry-20260409.md` |
| /industry-funnel | `{行业名}-funnel-{YYYYMMDD}.md`（根目录） | `reports/AI算力-funnel-20260509.md` |
| /private-company-research | `{公司名}-private-{YYYYMMDD}.md` | `reports/字节跳动/字节跳动-private-20260408.md` |
| /earnings-review | `{公司名}-earnings-{期间}.md` | `reports/腾讯/腾讯-earnings-2025Q4.md` |
| /earnings-team | `{公司名}/` 目录内含4个大师视角+研究底稿+公众号文章+读者评审 | `reports/腾讯/腾讯-earnings-2025Q4.md`（公众号定稿） |
| /thesis-tracker | `{公司名}-thesis.md`（长期维护） | `reports/腾讯/腾讯-thesis.md` |
| /portfolio-review | `portfolio-latest.md`（根目录，持续更新） | `reports/portfolio-latest.md` |
| /management-deep-dive | `{公司名}-management-{YYYYMMDD}.md` | `reports/腾讯/腾讯-management-20260409.md` |

## /investment-team 文件结构

```
reports/{公司名}/
├── README.md                         — 研究框架概览+核心结论
├── 01-商业模式分析-段永平视角.md
├── 02-财务估值分析-巴菲特视角.md
├── 03-行业竞争分析-芒格视角.md
├── 04-风险管理层评估-李录视角.md
└── 最终报告.md                       — Team Lead 综合报告
```

## 投研分析核心原则（最高优先级）

- **客观、客观、客观**——所有投研分析必须基于事实和数据，严禁主观臆断
- 严格区分"事实"与"观点"：事实用数据支撑，观点必须明确标注为"观点"或"推测"
- **不预设立场**：不预设看多或看空，先摆数据、再推逻辑、最后得结论。结论必须从数据中自然推出
- 禁止使用"我认为"、"我觉得"、"显然"等主观表述，改用"数据显示"、"证据表明"、"根据XX来源"
- **呈现正反两面**：每个核心判断都必须附带反面论据（"但另一方面..."），让读者自己权衡
- 对不确定的事情诚实说"不确定"或"数据不足"，不要用推测填充确定性
- 所有skill（investment-team、investment-research、earnings-review等）在执行时都必须遵守以上原则

## 报告语言与风格

- 所有报告使用**中文**
- 风格：直接、犀利、不说废话
- 数据必须标注来源，关键数据至少2个来源交叉验证
- 估计值必须注明"估计"
- 评分使用★符号（★1-5），不含半星
- 穿插巴菲特/芒格/段永平/李录的语录点评

## GitHub 操作

- 本地克隆路径：`~/ai-berkshire/`
- 远程仓库：`https://github.com/xbtlin/ai-berkshire.git`
- 推送前先 `git pull --rebase origin main`（远程经常有新提交）
- commit message 用中文，描述清楚改了什么
- 不要推送中间过程文件（如 data_collection.md），只推最终报告

## 常用命令

```bash
# 推送报告到GitHub
cd ~/ai-berkshire
git add reports/xxx.md
git commit -m "添加xxx报告"
git pull --rebase origin main
git push origin main
```

## 注意事项

- 市值必须手算校验：股价 × 总股本，与报告市值对比
- 货币单位要明确（港币/人民币/美元/新台币），防止混淆
- PE/ROE等指标用 tools/financial_rigor.py 精确计算
- A股数据用 tools/ashare_data.py 获取
- 台股数据用 tools/twstock_data.py（FinMind）获取，并按 skills/financial-data.md 台股章节交叉验证
- 报告定稿前用 tools/report_audit.py 抽检，打回则按原因修正后重抽
- 报告写完后主动询问是否推送到GitHub
