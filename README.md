# 公众号发布 Skill 套件

将 Markdown / Word / 飞书文档一键排版并发布到微信公众号草稿箱。支持**多品牌、多公众号**。

## 包含 Skill

| Skill | 触发方式 | 功能 |
|-------|---------|------|
| `ganlanhealth-format` | "排版这篇文章" | Markdown → 微信公众号 HTML（多品牌支持） |
| `ganlanhealth-publish` | "发布这篇 Word" | Word/Markdown → 排版 → 封面图 → 账号确认 → 微信草稿 |
| `ganlanhealth-feishu-draft` | "飞书这篇发公众号" | 飞书 URL → 提取 → 排版 → 封面图 → 微信草稿 |
| `ganlanhealthy-wechat-cover-prompt-generator` | "生成封面图" | 自动生成封面提示词（支持多品牌配色 + 火山引擎/Midjourney） |

## 快速安装

### 1. 安装 Skill

```bash
# 克隆到 Claude Code skills 目录
git clone https://github.com/wzephyr-l/ganlanhealth-skills.git

# 将 Skill 复制到 Claude Code 项目或全局 skills 目录
cp -r ganlanhealth-skills/ganlanhealth-* ~/.claude/skills/
cp ganlanhealth-skills/ganlanhealthy-wechat-cover-prompt-generator.md ~/.claude/skills/

# 重启 Claude Code（或刷新 skills）
```

### 2. 安装依赖工具

```bash
npm install -g md2wechat
```

### 3. 首次使用

首次使用时，Skill 会自动引导你：
1. **选择或创建品牌** — 提供参考文章链接，自动提取排版风格
2. **配置公众号凭据** — 友好引导获取 AppID 和 AppSecret

所有配置保存在 `~/.config/md2wechat/` 下，不会上传到任何云端服务。

## 使用

```
"排版这篇文章"                 → 选择品牌 → 排版 → 返回 HTML，不创建草稿
"发布这篇 Word"                → 排版 + 封面 + 选择公众号 + 创建草稿
"把飞书这篇发公众号 [URL]"     → 从飞书读取 + 排版 + 发布
"生成封面图"                   → 根据文章内容和品牌生成封面提示词
```

## 配置结构

```
~/.config/md2wechat/
├── brand.md                     ← 当前激活的品牌 (md2wechat 自动读取)
├── brands/                      ← 品牌仓库
│   ├── ganlanhealth.md          ← 橄榄健康科普 (预置模板)
│   └── your-brand.md            ← 用户自建品牌
└── wechat-accounts.json         ← 公众号凭据
    {
      "accounts": {
        "公众号名称": {
          "appid": "wx...",
          "app_secret": "...",
          "added_at": "2026-07-01",
          "last_used": "2026-07-01"
        }
      },
      "default": "公众号名称"
    }
```

### 添加新公众号

发布时选择「➕ 添加新公众号」，Skill 会引导你获取凭据：

> 📖 获取 AppID 和 AppSecret：
> 1. 打开 https://mp.weixin.qq.com → 登录
> 2. 设置与开发 → 基本配置
> 3. 开发者ID → AppID（直接复制）
> 4. 开发者密码 → 重置获取 AppSecret（⚠️ 只显示一次）

### 添加新品牌

排版时选择「🆕 创建新品牌」，提供 2-4 篇参考文章链接，Skill 会自动提取配色、字号、标题样式、卡片系统等排版规则。

## 排版风格参考

当前预置「橄榄健康科普」品牌，基于以下 4 篇参考文章提炼：

1. [多减35%！埃诺格鲁肽VS司美格鲁肽头对头研究](https://mp.weixin.qq.com/s/lQUmIzw_Qd9HBmfOicbYYA)
2. [每周一次，让腰围从L到S的减重"神药"](https://mp.weixin.qq.com/s/xFrK8P-1UAnQya74DUezTg)
3. [问爆的埃诺格鲁肽！10大核心问题全解答](https://mp.weixin.qq.com/s/2NdI8xwXzOSgSi_LrgwZWA)
4. [减肥失败的人，90%都试过水煮菜](https://mp.weixin.qq.com/s/iSDY4DXHINUrXFhz6udfUg)

> 💡 可随时提供新的参考文章来更新或创建品牌风格。

## 文件结构

```
ganlanhealth-skills/
├── README.md
├── brands/
│   └── ganlanhealth.md                         ← 预置品牌 Profile
├── ganlanhealthy-wechat-cover-prompt-generator.md ← 封面提示词 Skill
├── ganlanhealth-format/
│   └── SKILL.md                                ← 排版 Skill（含品牌选择）
├── ganlanhealth-publish/
│   └── SKILL.md                                ← 发布 Skill（含账号确认）
├── ganlanhealth-feishu-draft/
│   └── SKILL.md                                ← 飞书发布 Skill
└── memory/
    └── ganlanhealth-wechat-style.md            ← 橄榄健康排版风格参考
```

## 依赖

| 工具 | 用途 | 安装 |
|------|------|------|
| md2wechat | Markdown → 微信 HTML | `npm install -g md2wechat` |
| lark-cli | 飞书文档操作（仅飞书发布） | `npm install -g @larksuite/cli` |
| web-access Skill | CDP 浏览器读取（仅飞书发布和品牌提取） | 需安装 |
| Node.js 18+ | 运行时 | https://nodejs.org |

## License

MIT
