# Industry News Daily

一个通用的行业资讯日报/周报/月报生成器——Claude Code 技能（Skill）。

配置你的行业关键词和资讯来源，Claude 自动搜集、交叉验证、生成中文报道，推送到飞书群，并保存本地 Markdown 文件和深度分析文章。

## 特性

- **日报 / 周报 / 月报**：默认日报（昨天 00:01 至今），周一自动周报，每月 25 日月报，也可手动指定
- **多板块分类**：支持多个行业板块独立搜索、独立生成报道
- **多源搜索**：WebSearch 全网搜索 + 指定行业站点定向抓取 + 厂商官网 news 页面 + 社交媒体内容搜索
- **交叉验证**：时效性核验（尤其严查新品发布的真实时间）、重要资讯至少双源确认
- **飞书推送**：交互式卡片总览 + 富文本详细报道，自动分段防截断
- **Markdown 存档**：按日期 + 板块分类保存本地文件
- **深度分析文章**：每条新闻自动生成「行业观察 | xxx」格式的深度分析文章（1000~3000 字），适合发布到内容平台
- **跨日去重**：维护去重索引，同一条新闻不会在第二天重复出现
- **去 AI 味写作**：内置 10 条写作规则，确保文章平实、有判断、不套话
- **去重库管理**：可随时查看/删除/重置已报道新闻记录

## 快速开始

### 1. 安装技能

将本仓库克隆到你的 Claude Code 项目的 `.claude/skills/` 目录下：

```bash
cd your-project
mkdir -p .claude/skills
git clone https://github.com/YOUR_USERNAME/industry-news-daily.git .claude/skills/industry-news-daily
```

### 2. 配置你的行业

```bash
cp .claude/skills/industry-news-daily/config.example.json .claude/skills/industry-news-daily/config.json
```

编辑 `config.json`，填入你自己的行业配置。详细字段说明见下方「配置说明」。

**不想手动找信源？** 直接对 Claude 说：

```
帮我找出动力电池领域最权威的 20 个媒体和消息来源
```

Claude 会自动搜索、排名、展示候选列表，你勾选后直接写入 config.json。

### 3. 开始使用

在 Claude Code 对话中说：

```
查一下最近的行业动态
```

或者指定模式：

```
这周动力电池有什么新闻？整理成周报发飞书
```

## 配置说明

`config.json` 是唯一需要你修改的文件（`.gitignore` 已排除它）。所有字段：

### categories（必需）

你要追踪的行业板块，至少一个。每个板块包含：

| 字段 | 说明 | 示例 |
|------|------|------|
| `id` | 唯一标识，用于文件命名 | `"battery"` |
| `name` | 显示名称，用于报道标题 | `"动力电池"` |
| `search_keywords_cn` | 中文搜索关键词 | `["固态电池", "锂电池"]` |
| `search_keywords_en` | 英文搜索关键词 | `["solid state battery"]` |
| `company_keywords` | 核心公司名（会去官网抓新闻） | `["宁德时代 CATL"]` |
| `site_keywords` | 行业站点关键词（辅助搜索） | `["高工锂电"]` |

### sources（每个 category 内，推荐填写）

每个板块可以有自己的信源列表，放在 `category.sources` 下：

- **priority**：该板块核心媒体，每次运行优先抓取首页/新闻列表
- **normal**：该板块补充媒体，按需搜索

### shared_sources（可选）

所有板块共享的通用行业媒体（如 36氪、投资界、IEEE Spectrum）。搜索每个板块时，会自动合并「板块专属信源 + 共享信源」。

**怎么填？** 把你平时每天刷的行业网站放进 `priority`，偶尔看的放进 `normal`。每项填网站名和 URL 即可。

### feishu_webhook（如需飞书推送）

飞书机器人 Webhook 地址。如果不填，技能会跳过飞书推送，仅生成本地文件。

**获取方式：**
1. 打开飞书，进入你要接收消息的群聊
2. 点击群设置 → 群机器人 → 添加机器人 → 自定义机器人
3. 机器人名字填「行业资讯日报」，安全设置建议勾选「仅我一个人」（或按需配置）
4. 复制生成的 Webhook 地址，填入此处

```json
"feishu_webhook": "https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxxx"
```

也可以直接对 Claude 说：「帮我配置飞书机器人，Webhook 地址是 xxx」。

### output（可选）

生成文件的存储设置，默认即可。

### reports（可选）

日报/周报/月报的时间窗口和触发条件，默认即可。

### timezone（可选）

时区，默认 `"UTC+8"`（北京时间）。

## 使用示例

### 日报

```
最近两天半导体行业有什么新闻？
查一下MRAM和TMR传感器的最新动态
帮我看看今天锂电行业有什么值得关注的
```

### 周报

```
这周自动驾驶领域有什么进展？整理周报
本周储能行业动态
```

### 月报

```
本月AI芯片行业月报
这个月光伏行业有什么大事
```

### 管理去重库

```
查看去重库
删除第 3 条
重置去重库
```

## 文件结构

```
industry-news-daily/
├── SKILL.md                    # 技能主文件（9步工作流）
├── config.example.json         # 配置模板（复制为 config.json 填入你的行业）
├── README.md                   # 本文件
├── .gitignore                  # 排除 config.json 和生成内容
└── references/
    └── writing-rules.md        # 深度文章写作规范（10条去AI味规则）
```

运行后会在你的项目目录下生成：

```
your-project/
├── config.json                 # 你的配置（不追踪）
└── daily-reports/              # 生成的内容（不追踪）
    ├── .dedup.json             # 去重索引
    ├── YYYY-MM-DD-板块名.md    # 日报
    ├── weekly-YYYY-MM-DD-板块名.md   # 周报
    ├── monthly-YYYY-MM-DD-板块名.md  # 月报
    └── articles/               # 深度长文
        └── 行业观察-主题.md
```

## 依赖

- Claude Code（提供 WebSearch、WebFetch、Bash、Write 等工具）
- 飞书机器人 Webhook（用于推送消息）
- 无需额外安装任何 Python 包或 Node 模块

## 常见问题

**Q: 为什么飞书消息是乱码？**
A: Windows 下 curl 直接传中文会编码错误。本技能已处理——先写临时文件再用 `@文件路径` 发送。

**Q: 我改完 config.json 后需要重启 Claude Code 吗？**
A: 不需要。每次触发技能时会重新读取 config.json。

**Q: 怎么才能不让别人用我的配置生成和我一样的报告？**
A: `config.json` 已被 `.gitignore` 排除，不会被提交到 GitHub。你的行业配置和飞书 Webhook 仅保存在本地。

## 许可

MIT
