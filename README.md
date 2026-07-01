# 橄榄健康科普 — 微信公众号发布 Skill 套件

将 Markdown / Word / 飞书文档一键排版并发布到微信公众号草稿箱。

## 包含 Skill

| Skill | 触发方式 | 功能 |
|-------|---------|------|
| `ganlanhealth-format` | "排版这篇文章" | Markdown → 微信公众号 HTML |
| `ganlanhealth-publish` | "发布这篇 Word" | Word/Markdown → 排版 → 封面图 → 微信草稿 |
| `ganlanhealth-feishu-draft` | "飞书这篇发公众号" | 飞书 URL → 排版 → 封面图 → 微信草稿 |
| `ganlanhealthy-wechat-cover-prompt-generator` | "生成封面图" | 自动生成公众号封面提示词（适配火山引擎/Midjourney） |

## 快速安装

### 1. 安装 Skill

```bash
# 克隆到 Claude Code 全局 skills 目录
git clone https://github.com/YOUR_USERNAME/ganlanhealth-skills.git ~/.claude/skills/ganlanhealth/

# 重启 Claude Code（或刷新 skills）
```

### 2. 安装依赖工具

```bash
npm install -g md2wechat
```

### 3. 配置 Brand Profile

```bash
cp ~/.claude/skills/ganlanhealth/brand.md ~/.config/md2wechat/brand.md
```

### 4. 配置凭据（如需发布和封面图生成）

- **微信公众号**：AppID 和 AppSecret → 告诉 Claude "配置公众号"
- **火山引擎**：AccessKey 和 SecretAccessKey → 告诉 Claude "配置火山引擎文生图"
- **飞书发布**：额外需要 `npm install -g @larksuite/cli` 和 `web-access` Skill

## 使用

```
"排版这篇文章"                 → 仅排版，返回 HTML，不创建草稿
"发布这篇 Word"                → 排版 + 封面 + 微信草稿，给链接
"把飞书这篇发公众号 [URL]"     → 从飞书读取 + 排版 + 发布
"生成封面图"                   → 根据文章内容生成封面提示词
```

首次使用时 Skill 会自动检查环境，缺什么帮你装什么。

## 排版风格

当前排版规则基于以下 4 篇参考文章提炼：

1. [多减35%！埃诺格鲁肽VS司美格鲁肽头对头研究](https://mp.weixin.qq.com/s/lQUmIzw_Qd9HBmfOicbYYA)
2. [每周一次，让腰围从L到S的减重"神药"](https://mp.weixin.qq.com/s/xFrK8P-1UAnQya74DUezTg)
3. [问爆的埃诺格鲁肽！10大核心问题全解答](https://mp.weixin.qq.com/s/2NdI8xwXzOSgSi_LrgwZWA)
4. [减肥失败的人，90%都试过水煮菜](https://mp.weixin.qq.com/s/iSDY4DXHINUrXFhz6udfUg)

如需更新风格，提供新的公众号文章链接，Skill 会自动提取并更新 Brand Profile。

## 文件结构

```
ganlanhealth-skills/
├── README.md
├── brand.md                                    ← Brand Profile 模板
├── ganlanhealthy-wechat-cover-prompt-generator.md ← 封面提示词 Skill
├── ganlanhealth-format/
│   └── SKILL.md                                ← 排版 Skill
├── ganlanhealth-publish/
│   └── SKILL.md                                ← 发布 Skill
├── ganlanhealth-feishu-draft/
│   └── SKILL.md                                ← 飞书发布 Skill
└── memory/
    └── ganlanhealth-wechat-style.md            ← 排版风格参考
```

## 依赖

| 工具 | 用途 | 安装 |
|------|------|------|
| md2wechat | Markdown → 微信 HTML | `npm install -g md2wechat` |
| lark-cli | 飞书文档操作（仅飞书发布） | `npm install -g @larksuite/cli` |
| web-access Skill | CDP 浏览器读取（仅飞书发布） | 见 [web-access](https://github.com/xxx) |
| Node.js 18+ | 运行时 | https://nodejs.org |

## License

MIT
