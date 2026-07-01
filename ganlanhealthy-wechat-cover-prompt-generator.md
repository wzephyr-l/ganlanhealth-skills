---
name: ganlanhealthy-wechat-cover-prompt-generator
description: >
  微信公众号封面图提示词生成器，支持多品牌配色方案。
  触发条件：用户提供文章链接、上传附件（PDF/Word/TXT）或直接粘贴文章内容，并提出"生成封面图"、"封面提示词"、"做封面"等请求。
  品牌感知：由调用方 Skill 传入品牌名称，自动匹配对应配色；无品牌信息时使用通用医疗健康配色。
---

# 公众号封面图提示词生成器

## 工作流指令（严格执行）

### 第一步：品牌感知（优先于归属确认）

**品牌信息来源优先级：**
1. 调用方 Skill 传入的品牌名称（如 `ganlanhealth-publish` 或 `ganlanhealth-format` 传入）→ 直接使用
2. 当前激活的 Brand Profile：读取 `~/.config/md2wechat/brand.md` 头部 `## 品牌身份` 段
3. 用户明确提及的公众号名称 → 匹配 `~/.config/md2wechat/brands/` 下对应的 Profile

**如果获取到了品牌名称和配色方案** → 跳到第三步（核心生成逻辑），使用该品牌的配色。

**如果无法确定品牌** → 进入第二步（归属确认）。

---

### 第二步：归属确认（无品牌信息时触发）

**判断规则**：检查用户请求中是否包含以下指代（不区分大小写）：
- "橄榄健康科普" / "橄榄健康" + "公众号" / "我们公众号" + 健康科普

**若用户已明确提及** → 跳过确认，使用橄榄健康科普配色进入第三步。

**若用户未明确提及** → 调用 `AskUserQuestion`：

> "检测到您需要生成封面图提示词。请问这篇文章属于哪个公众号？"

选项：
["橄榄健康科普", "其他公众号（请告知名称）", "通用生成（不限品牌）"]

分支处理：

| 用户选择 | 行为 |
|---------|------|
| **橄榄健康科普** | ✅ 使用橄榄健康科普配色，进入第三步 |
| **其他公众号** | 提示用户提供公众号名称，尝试从 `~/.config/md2wechat/brands/` 查找对应配色。找到则使用，未找到则使用通用医疗健康配色 |
| **通用生成** | 使用通用医疗健康配色（主色 `#2C7A6D` 医疗绿，底色 `#F5F7FA` 冷灰，点缀 `#E8734A` 警示橙），进入第三步 |

---

### 第三步：核心生成逻辑（确认品牌后执行）

#### 3.1 获取品牌配色

根据确定的品牌，读取对应 Brand Profile 中的配色方案。如果没有品牌 Profile，使用以下通用配色：

```
主色调: #2C7A6D (医疗绿) + #F5F7FA (冷灰)
点缀色: #E8734A (警示橙)
```

> **品牌配色提取方法**：从 `~/.config/md2wechat/brands/<brand-slug>.md` 的 `## 配色方案` 段读取。取「品牌主色」为第一主色，取最浅的卡片背景色为第二主色，取「强调色」为点缀色。

#### 3.2 输入解析与文章分类

执行以下分类逻辑：

```python
import re

def classify_article_type(title, summary):
    text = (title + " " + summary).lower()
    
    # 类型1：药物/疗法对比类
    if re.search(r'vs|对决|pk|对比|还是|哪个好|胜出|头对头', text):
        return "TYPE_PK"
    # 类型2：问答/清单/盘点类（优先级高于DATA，避免数据语言截胡）
    if re.search(r'几大|问|答|问题|盘点|清单|指南|必看', text):
        return "TYPE_QA"
    # 类型3：避坑/误区/纠正认知类
    if re.search(r'误区|错|坑|别|不要|避开|真相|其实', text):
        return "TYPE_MISCONCEPT"
    # 类型4：数据/效果展示类
    if re.search(r'\d+%|减重|下降|降低|提升|增加|倍', text):
        return "TYPE_DATA"
    # 默认兜底
    return "TYPE_GENERAL"

### 3.3 提取核心视觉元素与文案
def extract_elements(title, summary, article_type):
    elements = {
        "main_subject": "",      # 核心物体
        "key_number": "",        # 关键数字（如有）
        "key_text": "",          # 关键短词（≤4个字）
        "action": ""             # 动作/状态
    }
    
    # 提取数字（第一个出现的百分比或整数）
    num_match = re.search(r'(\d+(?:\.\d+)?%?)', summary + title)
    if num_match:
        elements["key_number"] = num_match.group(1)
    
    if article_type == "TYPE_PK":
        entities = re.findall(r'([A-Za-z一-龥]{2,8}(?:肽|鲁肽|格鲁肽|药|片|针))', title)
        elements["main_subject"] = f"{entities[0]} vs {entities[1]}" if len(entities)>=2 else "两个药瓶/针管对峙"
        elements["action"] = "对峙/对决"
        elements["key_text"] = elements["key_number"] if elements["key_number"] else "PK"
    
    elif article_type == "TYPE_DATA":
        elements["main_subject"] = "体重秤/腰围尺/脂肪细胞"
        elements["action"] = "箭头下降/数字突出"
        elements["key_text"] = elements["key_number"] if elements["key_number"] else "数据"
    
    elif article_type == "TYPE_QA":
        elements["main_subject"] = "大型问号/对话气泡/数字标签"
        elements["action"] = "罗列/排列"
        match = re.search(r'(\d+大|\d+个)', summary + title)
        elements["key_text"] = match.group(1) if match else "干货"
    
    elif article_type == "TYPE_MISCONCEPT":
        elements["main_subject"] = "打叉的餐盘/对比图(错误vs正确)"
        elements["action"] = "左右对比/错误划掉"
        match = re.search(r'(水煮|生酮|断食|代餐|低碳|高蛋白)', title)
        elements["key_text"] = match.group(1) if match else "避坑"
    
    else:  # TYPE_GENERAL
        elements["main_subject"] = "胶囊/听诊器/医疗图标"
        elements["action"] = "居中展示"
        elements["key_text"] = "科普"
    
    return elements

### 3.4 组装最终提示词
def generate_prompt(article_type, elements, brand_colors):
    STYLE = "扁平插画风格，医疗健康科普风"
    COLOR = f"主色调为{brand_colors['primary']}和{brand_colors['secondary']}，点缀色用{brand_colors['accent']}突出数字"
    COMPOSITION = "中心构图，主体占画面60%，背景为纯色或极简渐变，留白充足"
    NEGATIVE = "禁止：写实照片、3D渲染、复杂背景、人脸、血腥、注射器扎入皮肤、暗黑风格、过度装饰"
    
    subject_map = {
        "TYPE_PK": f"画面中央为{elements['main_subject']}，呈左右对峙排列，中间有闪电或天平符号连接",
        "TYPE_DATA": f"画面中央为{elements['main_subject']}，旁边有巨大向下的绿色箭头，{elements['key_number']}数字以超大粗体置于视觉焦点",
        "TYPE_QA": f"画面中央为{elements['main_subject']}，数字/文字'{elements['key_text']}'以序号标签形式分散排列",
        "TYPE_MISCONCEPT": f"画面中央为{elements['main_subject']}，左侧为错误做法（灰色打叉），右侧为正确做法（绿色对勾）",
        "TYPE_GENERAL": f"画面中央为{elements['main_subject']}，四周环绕简约医疗图标"
    }
    subject_desc = subject_map.get(article_type, subject_map["TYPE_GENERAL"])
    text_desc = f"画面中嵌入大号文字：'{elements['key_text']}'，字体为无衬线粗体，位于画面正下方或左下角，白色带阴影，确保高辨识度"
    
    return "\n".join([
        f"【风格】{STYLE}",
        f"【构图】{COMPOSITION}",
        f"【主体】{subject_desc}",
        f"【色彩】{COLOR}",
        f"【文字】{text_desc}",
        f"【禁止】{NEGATIVE}"
    ])
```

**主执行流程：**
1. 从用户提供的链接、附件或文本中提取文章标题和核心摘要
2. 调用 `classify_article_type(title, summary)` 获取文章类型
3. 调用 `extract_elements(title, summary, article_type)` 获取视觉元素
4. 获取当前品牌配色（从 Brand Profile 或使用通用配色）
5. 调用 `generate_prompt(article_type, elements, brand_colors)` 生成最终提示词
6. 按【第四步】的格式输出

---

## 第四步：最终输出格式（严格约束）

最终输出必须为纯中文，分段清晰，每行以 【】 开头。
不得包含解释性文字，只输出可直接复制粘贴的提示词本体。
在提示词正文之后，空一行，附带工具和比例信息。

输出示例：
```
【风格】扁平插画风格，医疗健康科普风
【构图】中心构图，主体占画面60%，背景为纯色或极简渐变，留白充足
【主体】画面中央为"埃诺格鲁肽 vs 司美格鲁肽"，呈左右对峙排列，中间有闪电或天平符号连接
【色彩】主色调为#2B7A8A（深青）和#F0F4F8（浅灰蓝），点缀色用#E8845A（暖橙）突出数字
【文字】画面中嵌入大号文字：'35%'，字体为无衬线粗体，位于画面正下方或左下角，白色带阴影，确保高辨识度
【禁止】禁止：写实照片、3D渲染、复杂背景、人脸、血腥、注射器扎入皮肤、暗黑风格、过度装饰

【适用工具】Midjourney V6 / Stable Diffusion XL / Flux.1
【宽高比】16:9 (适合微信公众号封面)
```

## 附加约束与提醒

- **品牌配色优先**：如果调用方传入了品牌名称，使用该品牌的配色方案替代硬编码的 `#2B7A8A`/`#F0F4F8`/`#E8845A`
- **领域二次校验**：若文章内容明显不属于医疗健康领域（财经、娱乐、科技硬件等），主动提醒："该内容风格与健康科普领域不符，确认要继续生成吗？"
- **占位符兜底**：若提取不到数字和关键词，`key_text` 默认使用"科普"，`main_subject` 默认使用"胶囊/听诊器"
- **禁止编造数据**：若文章中没有具体数字，不自行编造百分比或数据

## 预置品牌配色表

| 品牌 | 主色 | 底色 | 点缀色 |
|------|------|------|--------|
| 橄榄健康科普 | `#2B7A8A` 深青 | `#F0F4F8` 浅灰蓝 | `#E8845A` 暖橙 |
| 通用医疗健康 | `#2C7A6D` 医疗绿 | `#F5F7FA` 冷灰 | `#E8734A` 警示橙 |

> 更多品牌配色从 `~/.config/md2wechat/brands/` 中的 Brand Profile 动态读取。
