# 公众号排版 Skill

将 Markdown 文章转换为微信公众号兼容 HTML，支持多品牌排版风格。

## 🚀 快速开始

**安装后第一次使用，只需说：**

> "帮我把这篇文章排版成公众号格式"
> "排版 [Markdown文件路径]"

Skill 会先询问你使用哪个品牌风格排版，然后自动完成转换。
排版完成后得到一段微信公众号可用的 HTML，可直接粘贴到公众号后台。

> 💡 **多品牌支持**：首次使用时会引导你选择或创建品牌。已配置的品牌会记住，下次直接选用。

## 触发条件

当用户提到以下关键词时触发此 Skill：
- 排版、格式化、公众号排版、微信排版
- 把这篇文章排版、帮我排一下、转成公众号格式
- 当 `ganlanhealth-publish` 或 `ganlanhealth-feishu-draft` Skill 内部调用时

## 输入

- Markdown 文件路径
- 或用户直接粘贴的 Markdown 内容
- 或从其他 Skill 传递过来的 Markdown 字符串
- 可选：品牌名称（由调用方 Skill 传入时，跳过品牌选择步骤）

## 依赖

| 依赖 | 用途 | 安装命令 |
|------|------|---------|
| [md2wechat](https://github.com/md2wechat) | Markdown → 微信 HTML 排版引擎 | `npm install -g md2wechat` |
| Brand Profile | 品牌排版规则 | 存放于 `~/.config/md2wechat/brands/` 目录 |
| `web-access` Skill | 提取参考文章样式（仅新建品牌时需要） | 须已安装 |

## 执行流程

---

### Step 0: 品牌选择（⭐ 核心步骤 — 排版前必须完成）

**如果调用方 Skill 已明确传入品牌名称，跳过此步骤，直接进入 Step 1。**

#### 0.0 检查默认品牌

读取偏好文件：

```bash
cat ~/.config/md2wechat/preferences.json 2>/dev/null || echo '{}'
```

解析 `default_brand` 字段：

| 值 | 行为 |
|----|------|
| `"ganlanhealth"` 等有效 slug | 该品牌设为默认，进入 0.0.1（快捷确认） |
| `null` 或文件不存在 | 没有默认品牌，进入 0.1（完整选择流程） |

##### 0.0.1 快捷确认（有默认品牌时）

使用 `AskUserQuestion` 工具：

> **"排版品牌："**

选项：

| 选项 | 说明 |
|------|------|
| `✅ [默认品牌名称]（默认）` | 直接使用默认品牌 |
| `🔄 选择其他品牌` | 进入 0.1（完整选择流程） |

- 用户选「默认品牌」→ 进入 0.5（激活品牌），跳过 0.1-0.4
- 用户选「选择其他」→ 进入 0.1

#### 0.1 扫描已配置的品牌

```bash
ls ~/.config/md2wechat/brands/ 2>/dev/null || echo "目录不存在"
```

列出所有 `.md` 文件，每个文件就是一个已配置的品牌。

如果 `brands/` 目录不存在或为空，跳过选择，直接进入「创建新品牌」流程（0.4）。

#### 0.2 询问用户选择品牌

使用 `AskUserQuestion` 工具：

> **"请选择排版使用的品牌风格："**

选项（动态生成，最多 4 项）：

| 选项 | 条件 |
|------|------|
| `[品牌名1] (已配置)` | 遍历 `brands/` 下的每个 `.md` |
| `[品牌名2] (已配置)` | 同上，按最近使用排序 |
| `🆕 创建新品牌` | 始终提供 |

> 如果已配置品牌超过 3 个，只显示最近使用的 3 个 + `🆕 创建新品牌`。

#### 0.3 分支处理

- **用户选择了已有品牌** → 进入 0.5（激活品牌）
- **用户选择了「创建新品牌」** → 进入 0.4

#### 0.4 新建品牌

##### 0.4.1 收集基本信息

使用 `AskUserQuestion` 询问：

> **"请输入新品牌的公众号名称（如：橄榄健康科普）："**

##### 0.4.2 收集参考文章

使用 `AskUserQuestion` 询问：

> **"请提供 2-4 篇该公众号的参考文章链接（微信公众平台 URL），用于提取排版风格。如果暂时没有，可以选择跳过，使用通用模板。"**

提供选项：
- `我提供链接` → 让用户粘贴 URL（逗号或换行分隔）
- `暂时跳过` → 使用通用医疗健康风格模板

##### 0.4.3 提取排版风格（如果用户提供了参考文章）

对每篇参考文章，使用 `web-access` Skill 的 CDP 浏览器模式提取关键样式：

```bash
# 打开文章
curl -s "http://localhost:3456/new?url=<文章URL>"

# 提取关键样式信息
curl -s -X POST "http://localhost:3456/eval?target=<TARGET_ID>" \
  -d "提取以下信息：
1. 正文字号(font-size)、字色(color)、行高(line-height)、字间距(letter-spacing)
2. 各级标题样式(字号、颜色、背景色)
3. 信息卡片样式(背景色、边框、内边距、圆角)
4. 品牌主色和辅助色(从标题/卡片/按钮提取HEX值)
5. 文章末尾的品牌尾部结构(关注引导、CTA按钮)
6. 段落间距和留白风格"
```

多篇文章提取后交叉对比，取**共用模式**作为品牌规则，单篇特有的作为可选项标注。

##### 0.4.4 生成 Brand Profile

根据提取结果，生成 Brand Profile 文件。文件名使用品牌名称的 slug 形式（小写、连字符分隔）：

```markdown
# [公众号名称]

<!--
  排版规则来源（YYYY-MM-DD 提炼）：
  1. [参考文章URL] — [文章标题]
  ...
-->

## 品牌身份
- 公众号名称：[名称]
- 领域：[从文章内容推断]
- 调性：[口语化/专业/亲切/严肃...]
- 读者画像：[目标读者描述]

## 视觉规范

### 正文
- 字号：[提取值或默认 16px]
- 字色：[提取值或默认 #3E3E3E]
- 行高：[提取值或默认 1.75]
- 字间距：[提取值或默认 2px]
- 字体族：[提取值或安全字体栈]
- 段落间距：[提取值]

### 配色方案
- 品牌主色：[HEX] — [用途]
- 强调色：[HEX] — [用途]
- ...（根据实际提取结果填充）

## 标题风格

### 二级标题（大段落起头）
（根据实际提取结果描述）

### 三级标题（子段落）
（根据实际提取结果描述）

## 信息卡片系统
（根据实际提取结果描述卡片类型和样式）

## 必备元素

### 温馨提示（健康科普类文章强制）
（根据实际提取结果或默认文案）

### 品牌尾部
（根据实际提取的 HTML 模板）

## 写作风格
（根据实际提取结果描述）

## 参考文章
（列出所有用于提炼的参考文章）
```

保存文件：

```bash
mkdir -p ~/.config/md2wechat/brands/
# 使用 Write 工具写入 ~/.config/md2wechat/brands/<brand-slug>.md
```

##### 0.4.5 写入 claude-mem 记忆

简要记录新品牌信息，方便后续跨会话回忆。

#### 0.5 激活选中品牌

将选中的 Brand Profile 复制到 md2wechat 的读取位置：

```bash
cp ~/.config/md2wechat/brands/<brand-slug>.md ~/.config/md2wechat/brand.md
```

> `~/.config/md2wechat/brand.md` 始终指向**当前激活**的品牌，md2wechat CLI 自动读取此文件。

#### 0.6 设置默认品牌

品牌激活后，询问用户是否需要设为默认：

使用 `AskUserQuestion` 工具：

> **"是否设置默认排版品牌？设置后，下次排版会优先使用该品牌。"**

选项（动态生成）：

| 选项 | 说明 |
|------|------|
| `✅ [当前品牌]（默认）` | 设为默认，下次自动选用 |
| `[其他已有品牌1]` | 遍历 `brands/` 下的每个品牌 |
| `[其他已有品牌2]` | 同上 |
| `❌ 不需要默认，每次都要选择` | 清除默认，以后每次都询问 |

处理：
- 用户选了某个品牌 → 写入 `~/.config/md2wechat/preferences.json` 的 `default_brand` 字段
- 用户选了「不需要默认」→ 将 `default_brand` 设为 `null`

```bash
# 更新 preferences.json
# 使用 Edit 或 Write 工具修改 ~/.config/md2wechat/preferences.json
```

`preferences.json` 格式：

```json
{
  "default_brand": "ganlanhealth",
  "default_account": null
}
```

#### 0.7 确认

```
✅ 当前排版品牌：[公众号名称]
📁 Brand Profile: ~/.config/md2wechat/brands/<brand-slug>.md

> 💡 下次排版时可以直接选用其他品牌。如需更新品牌风格，提供新的参考文章即可。
```

---

### Step 1: 环境检查（首次使用时自动执行）

**1.1 检查 md2wechat**
```bash
md2wechat --version
```
- ✅ 已安装 → 继续
- ❌ 未安装 → 告知用户并执行：`npm install -g md2wechat`

**1.2 检查 Node.js**
```bash
node --version
```
- 需要 Node.js 18+，不满则提示升级

> **注意**：日常使用跳过检查直接排版。

### Step 2: 读取输入并分析文章类型

读取 Markdown 内容，根据内容特征判断文章类型：

| 类型 | 识别特征 | 配色选择 |
|------|---------|---------|
| 数据解读 | 含临床数据、百分比、对比研究 | 品牌数据色系（或蓝色系） |
| Q&A 问答 | "N大问题""全解答""FAQ""问爆" | 品牌 Q&A 色系（或青绿色系） |
| 痛点科普 | 生活场景、健康话题、产品 | 品牌主色系（或暖橙色系） |

> 配色参考当前激活的 Brand Profile。如果 Brand Profile 没有指定文章类型配色，使用通用色系。

### Step 3: 预处理 Markdown

在调用 md2wechat 之前，对 Markdown 做以下预处理：

**3.1 段落拆分**
- 超过 3 句话的段落拆短
- 确保手机屏阅读有足够的视觉断点

**3.2 品牌尾部**
- 检查文章末尾是否已有品牌尾部段落
- 如果没有，从当前激活的 Brand Profile 中读取「必备元素 → 品牌尾部」模板并追加
- 如果 Brand Profile 提供的是 HTML 模板，转换为 Markdown 后追加

**3.3 温馨提示**
- 如果文章涉及药物/医疗/健康建议，检查是否有"温馨提示"段
- 如果没有，在参考文献之前插入（内容来自 Brand Profile 或以下默认模板）：

```markdown
> **温馨提示：** 本文仅为健康科普，不构成任何用药/医疗建议。具体用药请务必在专业医生指导下进行，切勿自行购买或滥用。
```

**3.4 超链接处理**
- 微信会破坏 `<a href>` 标签，将 Markdown 链接 `[文字](url)` 改为：
  ```
  🔗 文字 URL地址
  ```
- 示例：`[阅读原文](https://xxx.com)` → `🔗 阅读原文 https://xxx.com`

**3.5 参考文献**
- 如有 `[1]` `[2]` 格式的引用，确保有"参考文献"段落
- 参考文献使用 14px 字号提示（在 md2wechat 中通过 Brand Profile 处理）

### Step 4: 调用 md2wechat 转换

```bash
md2wechat convert <article.md> --mode ai --theme custom
```

- Brand Profile（`~/.config/md2wechat/brand.md`）会自动被 md2wechat 读取
- AI 模式根据 Brand Profile 中的描述生成匹配风格的 CSS/HTML
- 如果用户要求预览，先执行 `md2wechat preview <article.md>`

### Step 5: 输出

- 返回生成的微信公众号兼容 HTML
- 告知文章类型、所使用的品牌名称和配色系
- 如果被其他 Skill 调用，将 HTML 和品牌名称作为返回值传递

## 配置依赖

此 Skill 依赖以下文件：

1. `~/.config/md2wechat/brand.md` — 当前激活的品牌排版规则（Step 0.5 自动管理）
2. `~/.config/md2wechat/brands/` — 品牌仓库（存放所有已配置的品牌）

## 更新已有 Brand Profile

当用户提供新的参考文章来更新已有品牌时：

### 1. 读取参考文章
使用 `web-access` Skill 打开文章链接，提取排版数据：

```bash
curl -s "http://localhost:3456/new?url=<文章URL>"
curl -s -X POST "http://localhost:3456/eval?target=<ID>" \
  -d "提取：正文字号/字色/行高/字间距、各级标题样式、卡片配色/内边距/圆角、品牌尾部结构、高频色值"
```

### 2. 对照现有 Brand Profile
将提取的新文章数据与现有 `~/.config/md2wechat/brands/<brand-slug>.md` 对比：
- **一致的规则** → 保留（验证了稳定性）
- **新增的规则** → 追加
- **冲突的规则** → 询问用户偏好

### 3. 更新 Brand Profile 和参考文章列表
- 编辑 `~/.config/md2wechat/brands/<brand-slug>.md`
- 在文件头部注释中补充新参考文章链接

### 4. 验证
用一篇已有文章测试：`md2wechat preview article.md`，与参考文章对比确认。

---

## 微信 HTML 排版实战技巧

> 以下技巧来自实际排版、微信预览、用户反馈、修复的循环踩坑。

### 1. 微信 CSS 兼容性（微信会吞掉的样式）

| 写法 | 结果 | 替代方案 |
|------|------|---------|
| `linear-gradient(...)` 做背景 | 微信内置浏览器直接丢弃 | 用纯色 `background` |
| `<div>` 嵌套 `<h2>` 做章节标题 | 可能被标签清洗器吃掉 | 用 `<section>` + `<p>`，样式写 inline |
| 左边框 4px | 手机屏上几乎看不见 | 至少用 **6px** |
| CSS 值差 2px | 微信编辑器看不出变化 | 调整至少 **12px 起步** |

### 2. 章节标题的微信兼容写法

❌ **被吞写法：**
```html
<div style="background:linear-gradient(135deg, #1DA5FB 0%, #4A90D9 100%);">
  <h2 style="color:#ffffff;">标题</h2>
</div>
```

✅ **安全写法：**
```html
<section style="background:#0D7BCF;border-radius:6px;margin:32px 0 20px;padding:16px 20px;text-align:center;">
  <p style="margin:0;font-size:20px;font-weight:700;color:#ffffff;letter-spacing:2px;">标题</p>
</section>
```

要点：不用渐变 / 不用 `<h2>` / 颜色比 Web 上深一档（微信 WebView 会减淡）。

### 3. 相邻色块的视觉层级

| | 数据卡片（主角） | 专家点评（配角） |
|------|---------|---------|
| 左边框 | `border-left:6px solid` 品牌色 | 去掉 |
| 背景 | 品牌色浅底 | 中性灰 `#F5F7FA` |
| 字号 | 15-16px | 14px |
| 字色 | `#3E3E3E` | `#5a5a5a` |
| 引号 | 无 | 「」替代 "" |
| 内边距 | 18-20px | 14px |

### 4. 引用降级技巧

- 去掉彩色左边框 / 字号缩小一档 / 字色减淡 / 引号用「」/ 内边距减小

### 5. 统一间距体系

| 元素 | 值 |
|------|-----|
| 容器 padding | `20px` |
| 段落 margin-bottom | `24px` |
| 卡片 margin-bottom | `20px` |
| 章节标题 margin | `32px 0 20px` |
| 图片上边距 | `0`，下边距 `4px` |

**禁止**在 12/14/16/18/20/22/24 之间混用。

### 6. 段落区分技巧

- 段落间距 ≥ 24px / 章节间用虚线 `<hr>` / 开篇放引导卡片

### 7. 标题不重复原则

微信草稿 metadata 已有 `title`，正文区不要再出现 `<h1>`。

### 8. 改完要验证

- 每次改完打印文件大小——大小没变说明编辑失败
- `md2wechat create_draft` 只创建不覆盖，草稿箱会堆积
- 提醒用户看**最新一条**草稿

---

## 注意事项

- 此 Skill 只做排版转换，不生成封面图，不创建微信草稿
- 封面和草稿由 `ganlanhealth-publish` 或 `ganlanhealth-feishu-draft` 负责
- 排版 HTML 中不要包含 `<a href>` 标签
- AI 模式不使用 `:::module` 高级布局语法
- 排版时严格遵守上方「微信 HTML 排版实战技巧」8 条规则
