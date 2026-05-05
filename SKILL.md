---
name: industry-news-daily
description: 通用的行业资讯日报/周报/月报生成器。按用户配置的行业领域、关键词和资讯来源，搜集最新动态，交叉验证后生成中文报道，推送到飞书群，并保存本地Markdown文件。支持日报（默认）、周报（周一或手动触发）、月报（每月25日或手动触发）。当用户提及"今日动态""行业日报""查一下最近XX新闻""XX行业有什么新消息""帮我搜集XX资讯"及类似变体时触发。
---

# 行业资讯日报/周报/月报

## 概述

按 `config.json` 中配置的行业领域，搜集最新资讯，交叉验证后生成中文报道：
1. **分类整理**：按配置的 categories 分别生成独立板块
2. **飞书推送**：结构化卡片 + 详细报道推送到飞书群
3. **本地存档**：日报/周报/月报存为 Markdown 文件
4. **深度长文**：每条新闻撰写公众号风格的深度报道
5. **跨日去重**：维护 `.dedup.json` 避免同一事件重复报道

## 前置条件

首次使用前，将 `config.example.json` 复制为 `config.json`，填入自己的行业配置。`.gitignore` 已排除 `config.json`，飞书 webhook 等敏感信息不会被提交。

## 第一步：读取配置

从项目根目录的 `config.json` 读取所有配置项：
- `categories`：资讯分类及各自的关键词、厂商、站点关键词、**信源列表**
- `shared_sources`：跨板块共享的通用行业媒体（每个板块搜索时会合并板块专属信源 + 共享信源）
- `feishu_webhook`：飞书机器人 Webhook URL
- `output`：输出目录设置
- `reports`：日报/周报/月报的时间窗口和触发条件
- `search`：搜索轮数和公众号搜索策略

## 第二步：确定运行模式和时间窗口

根据当前北京时间和用户措辞自动选择模式：

| 模式 | 触发条件 | 时间窗口 |
|------|---------|---------|
| **日报**（默认） | 任意触发 | 昨天 00:01 ~ 当前（config 中 timezone 时区） |
| **周报** | 周一触发 / 用户说"周报"或"本周" | 往前 7×24 小时 |
| **月报** | 每月 25 日 / 用户说"月报"或"本月" | 往前 30×24 小时 |

文件命名规则：
- 日报：`{output.base_dir}/YYYY-MM-DD-{category.name}.md`
- 周报：`{output.base_dir}/weekly-YYYY-MM-DD-{category.name}.md`
- 月报：`{output.base_dir}/monthly-YYYY-MM-DD-{category.name}.md`

## 第三步：搜索资讯

对每个 category，分中英文各搜至少一轮（轮数见 `search.rounds_per_category`）：

1. **WebSearch 全网搜索**：用 category 的 `search_keywords_cn` 和 `search_keywords_en` 各搜若干轮，中英文交替。重要新闻用不同措辞额外验证
2. **微信公众号搜索**（双通道）：
   - 通道一：WebSearch + 关键词 + `site:mp.weixin.qq.com`
   - 通道二：WebFetch `https://weixin.sogou.com/weixin?type=2&query=<URL编码关键词>`
   - 两个通道结果合并去重
3. **指定网站定向搜索**：对每个 category 的 `sources.priority` 优先 `site:<domain>` 搜索或 WebFetch；同时合并 `shared_sources.priority`。`sources.normal` 和 `shared_sources.normal` 中的站点按需搜索
4. **厂商官网新闻**：对 `category.company_keywords` 中的核心厂商，WebFetch 其官网 news/press 页面

搜索时注意：
- 将时间窗口中的具体日期放入搜索词以缩小范围
- 结果不理想则调整关键词再搜
- 重点识别：技术突破、量产进展、投资融资、新品发布、公司观点、股价变动、技术路线、合作动态

## 第四步：筛选与交叉验证

1. **跨日去重**（优先执行）：
   - 先读取 `{output.base_dir}/{output.dedup_file}`，获取历史已报道新闻
   - URL 精确匹配 → 直接跳过
   - 标题核心关键词高度重合 → 标记为疑似重复
   - 同一事件不同来源但无新进展 → 跳过
   - 同一事件有实质性跟进报道 → 可收入，注明前情

2. **时效性验证**：每条候选新闻必须确认发生在时间窗口内
   - ⚠️ 新品发布类：极易混淆——某产品数月前已发布，近期仅被转载。必须核对至少 2 个来源的发布日期，追溯原始发布时间。发布时间不在窗口内的降级为"回顾"

3. **交叉确认**：重要资讯（技术突破、量产、投融资、新品）至少 2 个独立来源验证。单源标注"⚠️ 单源，待进一步确认"

4. **去重**：同一事件多家报道合并为一条，保留最详尽的一篇

5. **分类**：归入对应的 category，跨领域的归入更相关的板块并在另一板块末尾简要提及

## 第五步：生成报道

每条新闻格式：

```
### 新闻标题（简洁有力，15-25字）

[分类标签] 技术突破 | 量产进展 | 投资融资 | 新品发布 | 公司观点 | 合作动态 | 其他

正文 2-4 段：
- 第一段：事件核心（5W1H）
- 第二段：背景与意义
- 第三段（可选）：细节或行业影响

📌 来源：[来源名称](原始链接)
```

要求：150-400 字，数据准确，公司名首次出现时中英文对照，客观陈述不评论。

## 第六步：组装总览

为每个 category 生成总览：

```markdown
# 📰 {category.name} 行业日报 — YYYY年MM月DD日

> ⏰ 覆盖时间：YYYY/MM/DD HH:MM ~ YYYY/MM/DD HH:MM ({timezone})
> 📊 本期收录 N 条资讯 | 🌐 来源 M 家媒体

## 📌 本期总览

| 类型 | 数量 |
|------|------|
| 技术突破 | N |
| ... | ... |

快速摘要：一句话概括重点
```

周报/月报文末增加「📈 本期趋势观察」（2-4 句话概括整体趋势和值得关注的信号）。

## 第七步：推送到飞书

Webhook URL 来自 `config.json` 的 `feishu_webhook`。

推送 4 条消息（每个 category 各 1 条交互式卡片总览 + 1 条 text 详细报道）。

⚠️ **编码关键**：Windows 下 bash 直接传中文 `-d '...'` 会乱码。必须**先 Write JSON 到临时文件**，再 `curl -d @文件路径` + `charset=utf-8` 头发送。发送成功后删除临时文件。

交互式卡片模板：
```json
{"msg_type":"interactive","card":{"header":{"title":{"tag":"plain_text","content":"📰 {category.name}行业日报 | YYYY/MM/DD"},"template":"blue"},"elements":[{"tag":"markdown","content":"⏰ 覆盖时间：……\n📊 本期收录 **N** 条资讯\n\n**重点关注：**\n🔬 ……\n\n📎 详细报道见下一条消息"},{"tag":"hr"},{"tag":"note","elements":[{"tag":"plain_text","content":"由 Claude 自动生成 · 仅供参考"}]}]}}
```

## 第八步：保存文件

按第二步的文件命名规则，将每个 category 的完整报道（总览+所有详细内容）保存到 `{output.base_dir}/`。文件头部元信息：

```markdown
---
date: YYYY-MM-DD
type: 日报 | 周报 | 月报
category: {category.name}
coverage: YYYY/MM/DD HH:MM ~ YYYY/MM/DD HH:MM ({timezone})
source_count: N
---
```

保存完成后更新去重库 `{output.base_dir}/{output.dedup_file}`：
- 将本期所有报道的 title_hash、URL、日期、分类追加到 `seen` 数组
- 更新 `last_updated`

## 第九步：撰写深度报道

对每条新闻单独撰写深度长文，保存到 `{output.base_dir}/{output.articles_subdir}/`。

**命名**：`行业观察-主题slug.md`（不含日期）
**标题**：`# 行业观察 | 核心事件概括`
**字数**：1000~3000 字，周报/月报重大选题可至 5000 字

**写作规范**：详见 `references/writing-rules.md`。核心要点：
- 禁止宏大背景式开头，直接从事件切入
- 行业术语落地，数据有对比度
- 必须有人说话（直接引语或财报/分析师表态）
- 摒弃三方观点端水，给出核心判断
- 用动词不用名词化结构
- 允许留白和质疑收尾
- 平实克制，不卖弄口语（总纲，与其余规则冲突时优先）

**文章头部元信息**：
```markdown
---
date: YYYY-MM-DD
category: {category.name}
tags: [关键词1, 关键词2, ...]
sources: N
---
```

## 附加规则

- 某板块无新闻：如实说明"本期未监测到相关资讯动态"，不强行编造
- 搜索结果极少：扩大关键词，增英文搜索，对更多来源做 WebFetch
- 飞书单条消息约 20KB 上限，超过分段标注（1/N）
- 所有外部 URL 完整保留

## 去重库管理

用户可以随时通过对话管理去重库：

- **查询**：说「查看去重库」→ 读取 `.dedup.json`，带序号列表呈现
- **删除**：说「删除第 N 条」→ 移除对应记录，更新文件
- **重置**：说「重置去重库」→ 清空 `seen` 数组（需确认）
