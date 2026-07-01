# 橄榄健康科普 — 公众号排版 Skill

将 Markdown 文章转换为匹配"橄榄健康科普"公众号风格的微信公众号 HTML。

## 🚀 快速开始

**安装后第一次使用，只需说：**

> "帮我把这篇文章排版成公众号格式"
> "排版 [Markdown文件路径]"

Skill 会自动检查环境（md2wechat、Brand Profile），缺什么帮你装什么。
排版完成后得到一段微信公众号可用的 HTML，可直接粘贴到公众号后台。

**当前排版风格基于以下 4 篇参考文章提炼：**

1. [多减35%！埃诺格鲁肽VS司美格鲁肽头对头研究](https://mp.weixin.qq.com/s/lQUmIzw_Qd9HBmfOicbYYA)
2. [每周一次，让腰围从L到S的减重"神药"](https://mp.weixin.qq.com/s/xFrK8P-1UAnQya74DUezTg)
3. [问爆的埃诺格鲁肽！10大核心问题全解答](https://mp.weixin.qq.com/s/2NdI8xwXzOSgSi_LrgwZWA)
4. [减肥失败的人，90%都试过水煮菜](https://mp.weixin.qq.com/s/iSDY4DXHINUrXFhz6udfUg)

> 💡 **风格更新**：如需调整排版风格或适配新的品牌视觉，提供新的公众号文章链接即可——详见文末「如何基于新文章更新 Brand Profile」。

## 触发条件

当用户提到以下关键词时触发此 Skill：
- 排版、格式化、公众号排版、微信排版
- 把这篇文章排版、帮我排一下、转成公众号格式
- 当 `ganlanhealth-publish` 或 `ganlanhealth-feishu-draft` Skill 内部调用时

## 输入

- Markdown 文件路径
- 或用户直接粘贴的 Markdown 内容
- 或从其他 Skill 传递过来的 Markdown 字符串

## 依赖

| 依赖 | 用途 | 安装命令 |
|------|------|---------|
| [md2wechat](https://github.com/md2wechat) | Markdown → 微信 HTML 排版引擎 | `npm install -g md2wechat` |
| Brand Profile | 品牌排版规则 | 须存在于 `~/.config/md2wechat/brand.md` |

## 执行流程

### Step 0: 环境检查（首次使用时自动执行）

在开始排版前，检查依赖是否就绪：

**0.1 检查 md2wechat**
```bash
md2wechat --version
```
- ✅ 已安装 → 继续
- ❌ 未安装 → 告知用户并执行：`npm install -g md2wechat`

**0.2 检查 Brand Profile**
```bash
cat ~/.config/md2wechat/brand.md
```
- ✅ 已存在 → 继续
- ❌ 不存在 → 告知用户："Brand Profile 缺失，是否根据橄榄健康科普模板自动创建？"
  - 用户同意 → 将本 Skill 附带的模板写入 `~/.config/md2wechat/brand.md`
  - 用户拒绝 → 使用 md2wechat 默认样式（排版质量会下降，仅作为提示）

**0.3 检查 Node.js**
```bash
node --version
```
- 需要 Node.js 18+，不满则提示升级

> **注意**：Step 0 仅在首次使用或用户报告错误时执行。日常使用跳过检查直接排版。

### Step 1: 读取输入并分析文章类型

读取 Markdown 内容，根据内容特征判断文章类型：

| 类型 | 识别特征 | 配色系 |
|------|---------|--------|
| 数据解读 | 含临床数据、百分比、对比研究 | 蓝色系 |
| Q&A 问答 | "N大问题""全解答""FAQ""问爆" | 青绿色系 |
| 痛点科普 | 生活场景、减肥/健康话题、产品 | 暖橙色系 |

### Step 2: 预处理 Markdown

在调用 md2wechat 之前，对 Markdown 做以下预处理：

**2.1 段落拆分**
- 超过 3 句话的段落拆短
- 确保手机屏阅读有足够的视觉断点

**2.2 品牌尾部**
- 检查文章末尾是否已有 `-关注我们-` 段落
- 如果没有，自动追加：

```markdown
---

-关注我们-

👇

分享📨、点赞👍、推荐3连击
后续新文章才会及时被你看到
我们可不想错过任何一个关心你的机会
```

**2.3 温馨提示**
- 如果文章涉及药物/医疗/健康建议，检查是否有"温馨提示"段
- 如果没有，在参考文献之前插入：

```markdown
> **温馨提示：** 本文仅为健康科普，不构成任何用药/医疗建议。具体用药请务必在专业医生指导下进行，切勿自行购买或滥用。
```

**2.4 超链接处理**
- 微信会破坏 `<a href>` 标签，将 Markdown 链接 `[文字](url)` 改为：
  ```
  🔗 文字 URL地址
  ```
- 示例：`[阅读原文](https://xxx.com)` → `🔗 阅读原文 https://xxx.com`

**2.5 参考文献**
- 如有 `[1]` `[2]` 格式的引用，确保有"参考文献"段落
- 参考文献使用 14px 字号提示（在 md2wechat 中通过 Brand Profile 处理）

### Step 3: 调用 md2wechat 转换

```bash
md2wechat convert <article.md> --mode ai --theme custom
```

- Brand Profile（`~/.config/md2wechat/brand.md`）会自动被 md2wechat 读取
- AI 模式根据 Brand Profile 中的描述生成匹配风格的 CSS/HTML
- 如果用户要求预览，先执行 `md2wechat preview <article.md>`

### Step 4: 输出

- 返回生成的微信公众号兼容 HTML
- 告知文章类型和所使用的配色系
- 如果被其他 Skill 调用，将 HTML 作为返回值传递

## 配置依赖

此 Skill 依赖以下文件（缺失时排版质量下降但不会失败）：

1. `~/.config/md2wechat/brand.md` — 品牌排版规则（自动读取）
2. `memory/ganlanhealth-wechat-style.md` — 排版风格参考（手动读取作为补充上下文）

## 如何基于新文章更新 Brand Profile

当用户提供新的公众号参考文章时，按以下流程更新排版规则：

### 1. 读取参考文章
使用 `web-access` Skill（CDP 浏览器模式）打开文章链接，提取排版数据：

```bash
# 打开文章
curl -s "http://localhost:3456/new?url=<文章URL>"

# 提取关键样式
curl -s -X POST "http://localhost:3456/eval?target=<ID>" \
  -d "提取：正文字号/字色/行高/字间距、各级标题样式、卡片配色/内边距/圆角、品牌尾部结构、高频色值"
```

### 2. 对照现有 Brand Profile
将提取的新文章数据与现有 `~/.config/md2wechat/brand.md` 对比：
- **一致的规则** → 保留（验证了稳定性）
- **新增的规则** → 追加（如新的卡片类型、新的标题样式）
- **冲突的规则** → 问用户偏好，或改为"优先用新规则"

### 3. 更新 Brand Profile
```bash
# 编辑 Brand Profile
edit ~/.config/md2wechat/brand.md
```

### 4. 更新参考文章列表
在本文件顶部「快速开始」中补充新参考文章链接。

### 5. 验证
用一篇已有文章测试：`md2wechat preview article.md`，与参考文章对比确认。

---

## 注意事项

- 此 Skill 只做排版转换，不生成封面图，不创建微信草稿
- 封面和草稿由 `ganlanhealth-publish` 或 `ganlanhealth-feishu-draft` 负责
- 排版 HTML 中不要包含 `<a href>` 标签，微信会将其编码破坏
- AI 模式不使用 `:::module` 高级布局语法（那是 API 模式的功能）
