# tools/ · 可执行工具

> 这些是 SKILL.md 在工具调用模式下使用的脚本。**纯 Python 标准库实现，无需任何依赖**。

## 快速测试

```bash
# 算账
python tools/breakeven.py \
--daily-revenue 800 --daily-food-cost 250 \
--rent 8000 --labor 9000 --investment 350000 --category 奶茶

# 快招评分
python tools/quack_score.py \
--source "抖音广告" --hq-city 济南 \
--total-fee 580000 --direct-stores 2 --years 1 \
--promises "零加盟费,6个月回本,总部全包"

# 案例匹配
python tools/match_case.py --amount 900000 --category 奶茶 --location 县城
```

## 在 Claude Skills / Cursor / Codex 中使用

- **Claude (Skills)**：放在 skill 包内，模型按 SKILL.md 指引自动调用
- **Cursor / Codex / Cline**：通过 shell 工具调用
- **Coze / Dify**：可包装成 Python 插件

## 输出格式

所有工具均输出标准 JSON，便于程序化拼接到回复中。
