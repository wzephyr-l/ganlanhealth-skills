# 公众号发布 Skill

从 Word/Markdown 文件一键发布到微信公众号草稿箱，支持多品牌、多公众号。

## 🚀 快速开始

> "发布这篇 Word 到公众号"  —  粘贴或拖入 .docx/.md 文件
> "把飞书文档发公众号"     —  发飞书链接（自动走 feishu-draft Skill）

Skill 会依次完成：品牌选择 → 排版 → 封面图生成 → 账号确认 → 微信草稿创建 → 返回草稿链接。
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
| Brand Profile | 品牌排版规则 | 存放于 `~/.config/md2wechat/brands/` |

## 执行流程

### Step 0: 环境检查（首次使用时自动执行）

```bash
md2wechat --version
```
- ❌ 未安装 → `npm install -g md2wechat`

检查 `ganlanhealth-format` Skill：
- ❌ 不存在 → "发布 Skill 依赖排版 Skill，请先安装 ganlanhealth-format"

检查 Brand Profile：
```bash
ls ~/.config/md2wechat/brands/ 2>/dev/null
```
- ❌ 目录为空或不存在 → 提示用户先通过 `ganlanhealth-format` 配置品牌，或现在创建

> 日常使用跳过检查。

### Step 1: 接收输入

**如果是 .docx 文件：**
1. 使用 `docx` Skill 提取文档内容
2. 提取文本转为 Markdown 格式
3. 提取文档中的图片保存到临时目录

**如果是 .md 文件或粘贴文本：**
- 直接作为 Markdown 使用

### Step 2: 排版转换

调用 `ganlanhealth-format` Skill：
- 传入上一步得到的 Markdown
- 排版 Skill 会处理品牌选择（如果用户还没有选择过品牌）
- 获取排版后的微信公众号 HTML 和使用的品牌名称

如果用户只想看排版效果不发布，到此停止并返回 HTML 预览。

### Step 3: 封面图生成

**3.1 生成提示词**
调用 `ganlanhealthy-wechat-cover-prompt-generator` Skill：
- 传入文章标题、摘要和**当前使用的品牌名称**
- 封面生成器根据品牌名称选用对应的配色方案
- 获取封面图提示词

**3.2 询问用户是否代为生成封面图**

使用 `AskUserQuestion` 工具：

> **"封面图提示词已生成，是否需要我帮你生成封面图？"**

| 选项 | 说明 |
|------|------|
| `✅ 是，帮我生成（默认用豆包 doubao-seedream）` | 进入 3.3 检查配置 |
| `📋 把提示词发给我，我找其他 AI 做了再提供` | 将提示词完整展示给用户 |

- 用户选「是，帮我生成」→ 进入 3.3
- 用户选「把提示词发给我」→ 将提示词完整输出，记录 `cover_status: "pending_user"`，跳过 3.3-3.5，后续 Step 5 创建草稿时阻断提醒

**3.3 检查生图模型配置**

```bash
md2wechat config show --format json | grep image_api_key
```

- ✅ 已配置 → 进入 3.4 生图
- ❌ 未配置 → 展示引导：

```
🎨 默认使用豆包模型（doubao-seedream），新用户有 250 张免费额度。

开通步骤：

1️⃣  开通模型服务
   https://console.volcengine.com/ark/region:cn-beijing/openManagement?COMPUTER_VISION=%7B%7D&advancedActiveKey=model&tab=ComputerVision
   勾选 doubao-seedream-4-5 和 doubao-seedream-5-0-lite-260128

2️⃣  创建 API Key
   https://console.volcengine.com/ark/region:cn-beijing/apiKey?apikey=%7B%7D
   创建后将 Key 告诉我

3️⃣  保存配置
   收到 Key 后，我帮你保存到本地配置
   🔒 Key 仅存本地（~/.config/md2wechat/config.yaml），不上传任何云端服务
```

等用户提供 Key 后继续 3.4。

**3.4 生成图片**

调用火山引擎文生图 API（使用 md2wechat config 中配置的 image_api_key）：
- 模型：`doubao-seedream-4-5` 或 `doubao-seedream-5-0-lite-260128`
- 使用 3.1 生成的提示词
- 图片尺寸：微信封面标准（900×383，比例 2.35:1）
- 输出：保存为 PNG 文件

**3.5 上传到微信**

```bash
# 获取 access_token（使用确认后的公众号凭据）
curl "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET"

# 上传封面图（注意：type=thumb，永久素材）
curl -F "media=@cover.png" "https://api.weixin.qq.com/cgi-bin/material/add_material?access_token=TOKEN&type=thumb"
```
返回的 `media_id` 即 `thumb_media_id`。

---

### Step 4: 确认发布目标公众号（⭐ 核心步骤）

#### 4.0 检查默认账号

读取偏好文件：

```bash
cat ~/.config/md2wechat/preferences.json 2>/dev/null || echo '{}'
```

解析 `default_account` 字段：

| 值 | 行为 |
|----|------|
| `"橄榄健康科普"` 等有效账号名 | 该账号设为默认，进入 4.0.1（快捷确认） |
| `null` 或文件不存在 | 没有默认账号，进入 4.1（完整选择流程） |

##### 4.0.1 快捷确认（有默认账号时）

使用 `AskUserQuestion` 工具：

> **"发布到哪个公众号？"**

选项：

| 选项 | 说明 |
|------|------|
| `✅ [默认公众号名称]（默认）` | 直接使用默认账号 |
| `🔄 选择其他公众号` | 进入 4.1（完整选择流程） |

- 用户选「默认账号」→ 进入 Step 5（创建草稿），跳过 4.1-4.4
- 用户选「选择其他」→ 进入 4.1

#### 4.1 读取已配置的公众号

```bash
cat ~/.config/md2wechat/wechat-accounts.json 2>/dev/null || echo '{"accounts":{}}'
```

解析 JSON，列出已配置的公众号列表。

#### 4.2 询问用户选择目标公众号

使用 `AskUserQuestion` 工具：

> **"请选择发布到哪个公众号："**

选项（动态生成）：

| 选项 | 条件 |
|------|------|
| `[公众号名1] ✓` | 每个已配置的公众号 |
| `[公众号名2] ✓` | 同上 |
| `➕ 添加新公众号` | 始终提供 |

> 如果只有一个已配置的公众号，仍需要确认（防止误发到错误的号）。可以加一句 "(唯一已配置)" 提示。

#### 4.3 分支处理

- **用户选择了已配置的公众号** → 进入 Step 5（创建草稿），使用该账号凭据
- **用户选择了「添加新公众号」** → 进入 4.4

#### 4.4 添加新公众号

##### 4.4.1 友好引导用户获取凭据

告知用户需要以下信息，并给出获取方式：

```
📋 发布到微信公众号需要以下凭据：

1️⃣  **公众号名称** — 你给这个账号起的名字，方便以后识别（如：橄榄健康科普）

2️⃣  **AppID** — 公众号的唯一标识

3️⃣  **AppSecret** — 开发者密码

---

📖 获取 AppID 和 AppSecret 的方法：

  1. 浏览器打开 https://mp.weixin.qq.com
  2. 用公众号管理员微信扫码登录
  3. 左侧菜单 → 设置与开发 → 基本配置
  4. 在「开发者ID」下找到 **AppID**（公开可见，直接复制）
  5. 在「开发者密码」下点击「重置」获取 **AppSecret**
     ⚠️ AppSecret 生成后只显示一次，请立即复制保存！
        如果忘记了只能重新生成（旧密码会失效）

🔒 安全说明：
  AppSecret 仅保存在你本地电脑的 ~/.config/md2wechat/wechat-accounts.json
  不会被上传到任何云端服务。
```

##### 4.4.2 收集凭据

依次询问用户：
1. 公众号名称
2. AppID（格式：`wx` 开头，18 位字符）
3. AppSecret（32 位十六进制字符串）

##### 4.4.3 验证凭据有效性

```bash
# 用提供的凭据尝试获取 access_token
RESP=$(curl -s "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=<APPID>&secret=<SECRET>")

# 检查返回
echo "$RESP" | grep -q "access_token" && echo "✅ 验证成功" || echo "❌ 验证失败"
```

**验证结果处理：**

| 结果 | 行为 |
|------|------|
| ✅ 成功（返回 access_token） | 保存凭据，进入 4.4.4 |
| ❌ 失败（返回 errcode） | 根据错误码提示用户，让用户重新输入 |
| `41002` | AppSecret 缺失或错误 — 请检查是否完整复制 |
| `40013` | AppID 无效 — 请检查 AppID 是否正确 |
| `40164` | IP 不在白名单 — 提示用户将当前 IP 加入白名单（见下方处理） |

**IP 白名单处理：**
- 获取当前服务器外网 IP：`curl -s ifconfig.me`
- 告知用户："检测到当前 IP 不在公众号白名单中。请将 `[IP地址]` 添加到公众号后台 → 设置与开发 → 基本配置 → IP 白名单。"

##### 4.4.4 保存凭据

```bash
# 读取现有配置（如有）
cat ~/.config/md2wechat/wechat-accounts.json

# 更新并写回
# 使用 Edit 或 Write 工具更新 wechat-accounts.json
```

保存格式：

```json
{
  "accounts": {
    "[公众号名称]": {
      "appid": "wxXXXXXXXXXXXXXXXX",
      "app_secret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "added_at": "2026-07-01",
      "last_used": "2026-07-01"
    }
  },
  "default": "[公众号名称]"
}
```

##### 4.4.5 确认

```
✅ 公众号「[名称]」已配置并验证通过
📁 凭据文件: ~/.config/md2wechat/wechat-accounts.json
```

#### 4.5 设置默认发布账号（每次确认账号后询问）

无论用户选择了已有账号还是新添加了账号，在进入 Step 5 之前询问：

使用 `AskUserQuestion` 工具：

> **"是否将「[当前账号名]」设为默认发布账号？设置后，下次发布会优先使用该账号。"**

选项（动态生成）：

| 选项 | 说明 |
|------|------|
| `✅ [当前账号]（默认）` | 设为默认，下次自动选用 |
| `[其他已配置账号1]` | 遍历 `wechat-accounts.json` 中的其他账号 |
| `[其他已配置账号2]` | 同上 |
| `❌ 不需要默认，每次都要选择` | 清除默认，以后每次都询问 |

处理：
- 用户选了某个账号 → 写入 `~/.config/md2wechat/preferences.json` 的 `default_account` 字段
- 用户选了「不需要默认」→ 将 `default_account` 设为 `null`

```bash
# 更新 preferences.json
# 使用 Edit 或 Write 工具修改 ~/.config/md2wechat/preferences.json
```

> 注：`preferences.json` 同时管理默认品牌和默认账号，格式为：
> ```json
> { "default_brand": "ganlanhealth", "default_account": "橄榄健康科普" }
> ```

---

### Step 5: 创建公众号草稿

> 使用 Step 4 确认的公众号凭据。

**5.0 检查封面图**

```
├── ✅ 已有 thumb_media_id（Step 3 生成/上传成功）→ 继续 5.1
└── ❌ 无封面图（用户选了"找其他 AI 做"）
       → 告知用户：
         "创建公众号草稿需要封面图。请提供封面图片（可以直接发图给我），
          我来帮你上传到微信。

          之前生成的封面提示词供参考：
          ---
          [3.1 生成的提示词内容]
          ---"
       → 等待用户提供图片 → 上传微信 add_material?type=thumb → 获取 thumb_media_id → 继续 5.1
```

**5.1 准备参数**
```
title:      文章标题（纯文本，无 emoji）
author:     作者名
digest:     摘要（可选，54 字以内）
content:    Step 2 排版得到的 HTML
thumb_media_id: Step 3 上传得到的 media_id
```

**5.2 调用微信 API**

```bash
# 从 wechat-accounts.json 读取选中账号的凭据
ACCESS_TOKEN=$(curl -s "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$APPID&secret=$APPSECRET" | jq -r '.access_token')

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
| 40001 | access_token 无效 | AppSecret 可能已过期/被重置，引导用户更新 |

### Step 6: 自检

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

### Step 7: 通知用户

```
✅ 公众号草稿已创建！

📝 标题：[文章标题]
🏷️ 目标公众号：[公众号名称]
🎨 排版品牌：[品牌名称] / [配色系]
🖼️ 封面图：已生成并上传
📱 请前往公众号后台预览确认后手动发布

草稿管理页：https://mp.weixin.qq.com/ → 创作管理 → 草稿箱
```

## 依赖

- `ganlanhealth-format` Skill（排版转换 + 品牌选择）
- `ganlanhealthy-wechat-cover-prompt-generator` Skill（封面提示词）
- `docx` Skill（Word 文档解析，仅 .docx 输入时需要）
- 微信公众平台 API（草稿创建）
- 火山引擎 API（文生图）

## 配置前提

- 至少一个 Brand Profile 配置在 `~/.config/md2wechat/brands/`
- 至少一个公众号凭据配置在 `~/.config/md2wechat/wechat-accounts.json`
- 火山引擎 AccessKey 和 SecretAccessKey 已配置
- 服务器 IP 已加入微信公众号 IP 白名单
