# 飞书文档公众号发布 Skill

从飞书文档 URL 读取内容，排版后发布到微信公众号草稿箱。支持多品牌、多公众号。

## 🚀 快速开始

> "把这篇飞书发公众号 https://xxx.feishu.cn/wiki/xxx"

Skill 会自动：CDP 打开飞书 → 提取内容 → 品牌选择 → 排版 → 封面图 → 账号确认 → 创建微信草稿。

> 💡 如果想在飞书文档里直接协作编辑后再发布，这个 Skill 就是为你设计的。
> 如果文章在本地 Word/Markdown 里，用 `ganlanhealth-publish` 更直接。

## 触发条件

当用户提到以下关键词时触发此 Skill：
- 飞书发布、飞书转公众号、飞书文章发公众号、飞书到微信
- 把飞书这篇发出去、飞书文档转草稿
- 提供飞书文档 URL 并要求发布到公众号

## 输入

- 飞书文档 URL（如 `https://xxx.feishu.cn/wiki/xxx`）

## 依赖

| 依赖 | 用途 | 安装命令 |
|------|------|---------|
| [md2wechat](https://github.com/md2wechat) | Markdown → 微信 HTML 排版引擎 | `npm install -g md2wechat` |
| `ganlanhealth-format` Skill | 排版转换 + 品牌选择（内部调用） | 须存在于 `.claude/skills/ganlanhealth-format/SKILL.md` |
| `web-access` Skill | CDP 浏览器读取飞书文档 | 须存在于 `.claude/skills/web-access/SKILL.md` |
| `lark-cli` | 飞书 API 操作 | `npm install -g @larksuite/cli` |
| Brand Profile | 品牌排版规则 | 存放于 `~/.config/md2wechat/brands/` |

## 执行流程

### Step 0: 环境检查（首次使用时自动执行）

```bash
md2wechat --version        # ❌ → npm install -g md2wechat
lark-cli --version         # ❌ → npm install -g @larksuite/cli
```

检查 Brand Profile：
```bash
ls ~/.config/md2wechat/brands/ 2>/dev/null
```
- ❌ 目录为空或不存在 → 提示用户在后续排版环节配置品牌

检查 `ganlanhealth-format` Skill → ❌ → "请先安装 ganlanhealth-format Skill"
检查 `web-access` Skill → ❌ → "请先安装 web-access Skill"

> 日常使用跳过检查。

### Step 1: 读取飞书文档

使用 CDP 浏览器模式读取飞书文档（飞书是 SPA，需要真实浏览器渲染）：

**1.1 打开文档**
```bash
curl -s "http://localhost:3456/new?url=<飞书文档URL>"
```

**1.2 展开所有折叠内容**
```bash
curl -s -X POST "http://localhost:3456/eval?target=<TARGET_ID>" \
  -d "document.querySelectorAll('[class*=fold],[class*=expand],[class*=collapse]').forEach(e=>e.click())"
```

**1.3 滚动加载全部内容**
```bash
curl -s "http://localhost:3456/scroll?target=<TARGET_ID>&direction=bottom"
```

**1.4 提取内容**
```bash
curl -s -X POST "http://localhost:3456/eval?target=<TARGET_ID>" \
  -d "Array.from(document.querySelectorAll('[data-block-id]')).map(b=>({id:b.getAttribute('data-block-id'),text:b.innerText?.trim()})).filter(b=>b.text?.length>2)"
```

**1.5 关闭标签**
```bash
curl -s "http://localhost:3456/close?target=<TARGET_ID>"
```

### Step 2: 飞书内容 → Markdown

将提取的飞书块内容转换为 Markdown：

- 标题块（heading）→ `##` / `###` 标题
- 文本块 → 段落
- 列表块（bullet/numbered）→ `-` / `1.` 列表
- 图片块 → `![描述](图片URL)` （记录图片 URL 后续下载）
- 表格块 → Markdown 表格
- 代码块 → ` ``` ` 代码块
- 引用块 → `>` 引用
- 分隔线 → `---`

**图片处理：**
- 飞书图片 URL 需要登录态才能访问
- 在 CDP 中 navigate 到图片 URL 截图保存
- 或跳过图片，在排版 HTML 中标注图片占位

### Step 3: 排版转换

调用 `ganlanhealth-format` Skill：
- 传入 Step 2 得到的 Markdown
- **排版 Skill 会自动处理品牌选择**（列出已有品牌或引导创建新品牌）
- 获取排版后的微信公众号 HTML 和使用的品牌名称

### Step 4: 封面图生成

**与 `ganlanhealth-publish` Skill 的 Step 3 完全相同。**

包括：3.1 生成提示词 → 3.2 询问是否代为生成 → 3.3 检查生图配置 → 3.4 生成图片 → 3.5 上传微信。详细流程参见 `ganlanhealth-publish` Skill 的 Step 3。

### Step 5: 确认发布目标公众号

**与 `ganlanhealth-publish` Skill 的 Step 4 完全相同。**

详细流程参见 `ganlanhealth-publish` Skill 的 Step 4。

### Step 6: 创建公众号草稿

**与 `ganlanhealth-publish` Skill 的 Step 5 完全相同（含 5.0 封面图检查阻断）。**

详细流程参见 `ganlanhealth-publish` Skill 的 Step 5。

### Step 7: 自检

**与 `ganlanhealth-publish` Skill 的 Step 6 完全相同。**

### Step 8: 通知用户

```
✅ 公众号草稿已创建！

📝 标题：[文章标题]
📄 来源：飞书文档 [URL]
🏷️ 目标公众号：[公众号名称]
🎨 排版品牌：[品牌名称] / [配色系]
🖼️ 封面图：已生成并上传
📱 请前往公众号后台预览确认后手动发布

草稿管理页：https://mp.weixin.qq.com/ → 创作管理 → 草稿箱
```

## 飞书特有的注意事项

- 飞书文档是 SPA，内容懒加载，必须 CDP 滚动到底部才能获取全部内容
- 飞书图片 URL 带登录态，不能直接用于微信正文（需要下载后重新上传到微信）
- 飞书文档中的折叠块内容必须在提取前手动展开
- 飞书表格可能渲染为复杂 DOM，提取时需要解析行列结构
- 完成操作后关闭自己创建的 CDP tab，不污染用户浏览器

## 依赖

- `web-access` Skill（CDP 浏览器操作）
- `ganlanhealth-format` Skill（排版转换 + 品牌选择）
- `ganlanhealthy-wechat-cover-prompt-generator` Skill（封面提示词）
- 微信公众平台 API（草稿创建）
- 火山引擎 API（文生图）
