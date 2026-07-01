# 橄榄健康科普 — 公众号发布 Skill

从 Word/Markdown 文件一键发布到微信公众号草稿箱。

## 🚀 快速开始

> "发布这篇 Word 到公众号"  —  粘贴或拖入 .docx/.md 文件
> "把飞书文档发公众号"     —  发飞书链接（自动走 feishu-draft Skill）

Skill 会依次完成：排版 → 封面图生成 → 微信草稿创建 → 返回草稿链接。
只需去公众号后台预览确认即可手动发布。

> 💡 只想看排版效果？用 `ganlanhealth-format` Skill（说"排版这篇文章"即可），不创建草稿。

## 触发条件

当用户提到以下关键词时触发此 Skill：
- 发布公众号、发布文章、创建草稿、发文章
- 把这篇文章发出去、推送到公众号
- 提到 .docx 或 Word 文档并提及公众号

## 输入

- Word 文档（.docx）路径
- 或 Markdown 文件（.md）路径
- 或用户直接粘贴的文章内容

## 依赖

| 依赖 | 用途 | 安装命令 |
|------|------|---------|
| [md2wechat](https://github.com/md2wechat) | Markdown → 微信 HTML 排版引擎 | `npm install -g md2wechat` |
| `ganlanhealth-format` Skill | 排版转换（内部调用） | 须存在于 `.claude/skills/ganlanhealth-format/SKILL.md` |
| Brand Profile | 品牌排版规则 | 须存在于 `~/.config/md2wechat/brand.md` |

## 执行流程

### Step -1: 环境检查（首次使用时自动执行）

在开始发布前，检查依赖是否就绪：

**检查 md2wechat**
```bash
md2wechat --version
```
- ❌ 未安装 → `npm install -g md2wechat`

**检查 Brand Profile**
```bash
cat ~/.config/md2wechat/brand.md
```
- ❌ 不存在 → 告知用户并询问是否根据模板自动创建

**检查 ganlanhealth-format Skill**
- ❌ 不存在 → 告知用户："发布 Skill 依赖排版 Skill，请先安装 ganlanhealth-format"

> 日常使用跳过检查。

### Step 0: 接收输入

**如果是 .docx 文件：**
1. 使用 `docx` Skill 提取文档内容
2. 提取文本转为 Markdown 格式
3. 提取文档中的图片保存到临时目录

**如果是 .md 文件或粘贴文本：**
- 直接作为 Markdown 使用

### Step 1: 排版转换

调用 `ganlanhealth-format` Skill：
- 传入上一步得到的 Markdown
- 获取排版后的微信公众号 HTML
- 记录使用的配色系

如果用户只想看排版效果不发布，到此停止并返回 HTML 预览。

### Step 2: 封面图生成

**2.1 生成提示词**
调用 `ganlanhealthy-wechat-cover-prompt-generator` Skill：
- 传入文章标题和摘要
- 获取封面图提示词

**2.2 生成图片**
调用火山引擎文生图：
- 模型：`doubao-seedream-4-5` 或 `doubao-seedream-5-0-lite`
- 使用上一步的提示词
- 图片尺寸：微信封面标准（900×383 或 2.35:1 比例）
- 输出：保存为 PNG 文件

**2.3 上传到微信**
```bash
# 获取 access_token
curl "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET"

# 上传封面图（注意：type=thumb，永久素材）
curl -F "media=@cover.png" "https://api.weixin.qq.com/cgi-bin/material/add_material?access_token=TOKEN&type=thumb"
```
返回的 `media_id` 即 `thumb_media_id`。

### Step 3: 创建公众号草稿

**3.1 确认公众号**
- 询问用户要发布到哪个公众号
- 如果只有一个已配置的公众号，直接使用
- 公众号凭据存放于 memory 文件中，从中读取 AppID 和 AppSecret

**3.2 准备参数**
```
title:      文章标题（纯文本，无 emoji）
author:     作者名
digest:     摘要（可选，54 字以内）
content:    Step 1 排版得到的 HTML
thumb_media_id: Step 2 上传得到的 media_id
```

**3.3 调用微信 API**

```bash
ACCESS_TOKEN=$(curl -s "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$APPID&secret=$SECRET" | jq -r '.access_token')

curl -X POST "https://api.weixin.qq.com/cgi-bin/draft/add?access_token=$ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "articles": [{
      "title": "文章标题",
      "author": "作者名",
      "digest": "摘要",
      "content": "HTML内容",
      "thumb_media_id": "封面media_id"
    }]
  }'
```

**⚠️ 关键注意事项：**
- 参数名是 `secret` 不是 `appsecret`
- `content` 中的 HTML 不要包含 `<a href>` 标签（微信会编码破坏）
- 返回的 `media_id` 是草稿 ID，不是发布后的文章 ID

**常见错误码：**

| 错误码 | 含义 | 处理 |
|--------|------|------|
| 40164 | IP 不在白名单 | 将当前服务器 IP 发给用户，添加到公众号 IP 白名单 |
| 41004 | secret missing | 微信服务器缓存问题，重试 2-3 次 |
| 42001 | token 过期 | 重新获取 access_token |

### Step 4: 自检

创建草稿后必须自行检查两遍：

**检查清单：**
- [ ] 错字、标点、格式
- [ ] 标题格式正确（无 emoji）
- [ ] 每个链接都是「文字 + URL」格式（如 `🔗 阅读原文 https://xxx`）
- [ ] 总结框有彩色边框/背景
- [ ] 配图尺寸正确
- [ ] HTML 编码用了 `data` 参数方式

如有问题，更新草稿：
```bash
POST /cgi-bin/draft/update?access_token=TOKEN
```

### Step 5: 通知用户

```
✅ 公众号草稿已创建！

📝 标题：[文章标题]
🎨 排版风格：[配色系]
🖼️ 封面图：已生成并上传
📱 请前往公众号后台预览确认后手动发布

草稿管理页：https://mp.weixin.qq.com/ → 创作管理 → 草稿箱
```

## 依赖

- `ganlanhealth-format` Skill（排版转换）
- `ganlanhealthy-wechat-cover-prompt-generator` Skill（封面提示词）
- `docx` Skill（Word 文档解析，仅 .docx 输入时需要）
- `memory/ganlanhealth-wechat-style.md`（风格参考）
- 微信公众平台 API（草稿创建）
- 火山引擎 API（文生图）

## 配置前提

- 微信公众号 AppID 和 AppSecret 已配置在 memory 中
- 火山引擎 AccessKey 和 SecretAccessKey 已配置
- 服务器 IP 已加入微信公众号 IP 白名单
