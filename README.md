# yongge-restaurant-skill

一个面向中文餐饮创业决策的 Agent Skill。基于公开整理的「勇哥餐饮创业说」（梁朝勇）连麦内容，沉淀出可复用的诊断 SOP、选址方法论、加盟欺诈识别框架、案例库与对话风格指引，可被 Claude Skills、Cursor、Codex、Cline、Coze、Dify 等 Agent 框架直接加载。

## 能力

| 能力 | 说明 |
|---|---|
| 已开门店诊断 | 收集营业额、毛利率、房租、人工等数据，计算保本线与达成率，给出三档结论（劝退 / 整改 / 优化）。 |
| 开店决策 | 走五步法（选品、预算、选址、算账、预案），输出可执行的判断与备选方案。 |
| 加盟风险评分 | 基于五项指标（信息来源、总部地理、投入金额、直营规模、承诺话术）给出量化得分。 |
| 选址打分 | 基于 12 项观察维度（门头、台阶、阴阳街、转让密度、同业浓度、竞品环境等）输出加权总分。 |
| 案例匹配 | 在 27 个真实案例中按金额、品类、地理、关键词四维度返回最相近样本。 |
| 风格化输出 | 按统一的语气控制约束生成回复，确保结论直接、可溯源、不依赖主观判断。 |

## 仓库结构

```
.
├── SKILL.md                  Skill 加载入口，含 YAML frontmatter
├── README.md                 本文档
├── LICENSE                   MIT
├── CONTRIBUTING.md           贡献指南
├── CHANGELOG.md              版本记录
│
├── corpus/                   知识库，可直接用于 RAG
│   ├── 01-人物档案.md
│   ├── 02-诊断SOP.md
│   ├── 03-选址方法论.md
│   ├── 04-快招识别指南.md
│   ├── 05-品类与赛道.md
│   ├── 06-金句与话术.md
│   ├── 07-行业常识速查.md
│   ├── 08-时代背景与文化.md
│   └── 99-信源索引.md
│
├── cases/                    案例库
│   ├── README.md             索引（按金额、品类、原因）
│   ├── 失败案例/             22 个完整案例
│   └── 成功案例/             5 个完整案例
│
├── skill/                    工程化资源
│   ├── 提问树.md             对话状态机
│   ├── 街景观察清单.md       12 项打分模型
│   ├── 保本线计算器.md       公式与各品类参数
│   └── 风格指引.md           语气控制
│
├── tools/                    可执行工具（Python 标准库，无依赖）
│   ├── breakeven.py
│   ├── quack_score.py
│   ├── match_case.py
│   └── README.md
│
└── docs/                     设计文档
    ├── skill-架构.md
    └── 路线图.md
```

## 接入方式

### Claude Skills

将仓库放入 Skills 目录，Claude 会读取根目录 `SKILL.md` 的 frontmatter 完成注册：

```bash
git clone https://github.com/Astro-wen/yongge-restaurant-skill
```

### Cursor / Codex / Cline

将 `SKILL.md` 写入项目规则文件：

```bash
cp SKILL.md .cursorrules     # Cursor
cp SKILL.md AGENTS.md        # Codex
cp SKILL.md .clinerules      # Cline
```

### Coze / Dify / 扣子

- 人设：复制 `SKILL.md` 全文为 Bot Prompt
- 知识库：上传 `corpus/` 与 `cases/` 作为 RAG 数据
- 工具：将 `tools/*.py` 注册为自定义插件

### 通用 OpenAI 兼容协议

```python
import requests

with open("SKILL.md", encoding="utf-8") as f:
    system_prompt = f.read()

resp = requests.post("/v1/chat/completions", json={
    "model": "claude-3-5-sonnet",
    "messages": [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "我开了家奶茶店在县城综合体三楼，每天卖 800"},
    ],
})
```

## 工具

三个零依赖 Python 脚本，输入参数后以 JSON 输出，便于程序化拼接到回复中。

### 保本线计算

```bash
python3 tools/breakeven.py \
  --daily-revenue 800 \
  --daily-food-cost 250 \
  --rent 8000 \
  --labor 9000 \
  --investment 350000 \
  --category 奶茶
```

输出字段：`gross_margin`、`monthly_fixed`、`daily_breakeven`、`monthly_breakeven`、`achievement_rate`、`status`、`monthly_profit_estimate`、`payback_months`、`risk_flags`、`yongge_verdict`。

### 加盟风险评分

```bash
python3 tools/quack_score.py \
  --source "抖音广告" \
  --hq-city "济南" \
  --total-fee 580000 \
  --direct-stores 2 \
  --years 1 \
  --promises "零加盟费,6个月回本,总部全包"
```

输出字段：`score`、`level`、`reasons`、`yongge_verdict`。

### 案例匹配

```bash
python3 tools/match_case.py \
  --amount 900000 \
  --category 奶茶 \
  --location 县城 \
  --top-k 3
```

输出按 `match_score` 降序的案例数组，含案例 id、名称、金额、品类、地理、标签与文件路径。

无 Python 工具调用能力的运行环境可直接按 [`skill/保本线计算器.md`](./skill/保本线计算器.md) 与 [`corpus/04-快招识别指南.md`](./corpus/04-快招识别指南.md) 中的公式手算。

## 数据规模

| 类别 | 数量 |
|---|---|
| 核心语料文档 | 9 篇 |
| 失败案例 | 22 个 |
| 成功案例 | 5 个 |
| 工程化资源 | 4 篇 |
| 可执行工具 | 3 个 |
| 信源 | 13 个权威源（虎嗅、36 氪、腾讯新闻、网易、新浪、人人都是产品经理、百度百科、明日歌笔记等） |

## 设计原则

1. 数据优先于直觉，所有结论必须建立在可验证的数字之上。
2. 倾向劝退而非鼓励，避免对高风险决策提供过于乐观的信号。
3. 类比真实案例优先于抽象说理。
4. 输出短句，避免长段陈述。
5. 始终预设最坏情况，因为咨询场景下用户通常已临近风险阈值。

完整风格控制规则见 [`skill/风格指引.md`](./skill/风格指引.md)。

## 贡献

欢迎以下类型的 PR：

- 新案例（按 [`cases/失败案例/_模板.md`](./cases/失败案例/_模板.md) 提交，必须含金额、品类、时间、出处）
- 新金句（必须含可溯源链接或具体场景）
- 街景观察清单的新维度（必须含阈值与至少一个反例案例）
- skill 接入示例与运行报告

详见 [`CONTRIBUTING.md`](./CONTRIBUTING.md)。

## 引用与边界

- 内容整理自公开网络资料，金句、案例、数据均标注来源，著作权归原作者所有。
- 「勇哥」「勇哥餐饮创业说」「梁朝勇」等名称权益归原主体所有，本仓库与原主体及四川三更教育科技有限公司无任何官方关联。
- 方法论与工具仅供参考，最终决策责任由使用者本人承担。

## License

MIT，详见 [`LICENSE`](./LICENSE)。
