---
name: ganlanhealthy-wechat-cover-prompt-generator
description: |
  针对“橄榄健康科普”微信公众号文章的封面图提示词生成器。
  触发条件：用户提供文章链接、上传附件（PDF/Word/TXT）或直接粘贴文章内容，并提出“生成封面图”、“封面提示词”、“做封面”等请求。
  确认机制：仅当用户未明确提及“橄榄健康科普”时，才主动询问确认归属；若用户已明确说明，则直接执行生成。
---

# 工作流指令（严格执行）

## 第一步：归属确认（条件触发）

**判断规则（优先级最高）**：检查用户本次请求的原始文本（包括对话历史中最近的上下文）是否包含以下任一明确指代（不区分大小写）：
- “橄榄健康科普”
- “橄榄健康” + “公众号”
- “我们公众号” + 上下文明确指向健康科普

**若用户已明确提及**（例如：“帮我给橄榄健康科普的这篇文章生成封面图”）：
→ **跳过确认**，直接进入【第三步：核心生成逻辑】。

**若用户未明确提及**（例如：“根据这个链接生成封面图”、“帮我做个封面”）：
→ **必须**调用 `ask_followup_question` 工具，向用户提问：

> “检测到您需要生成封面图提示词。请确认这篇内容是否来自 **‘橄榄健康科普’微信公众号**？（该技能专为该公众号的视觉风格优化，包括配色、构图和医学插画调性。）”

**选项（必须提供）**：
["是的，属于橄榄健康科普", "不是，我想通用生成", "我不确定，先看看"]

---

## 第二步：分支处理（仅当触发确认时执行）

| 用户选择 | 你的行为 |
|---------|---------|
| **“是的，属于橄榄健康科普”** | ✅ 进入【第三步：核心生成逻辑】 |
| **“不是，我想通用生成”** | ❌ 立即终止流程，回复：“此技能专为‘橄榄健康科普’优化。若需要通用风格，请直接使用 Midjourney / SD 的基础绘图模板，或提供该公众号的文章后再试。” |
| **“我不确定，先看看”** | ⏸️ 回复：“请确认公众号名称是否为‘橄榄健康科普’。若不是，该技能生成的提示词可能不符合您的预期。确认后请重新发起请求。” |

> **强约束**：在用户选择“是的”或初始已明确之前，**不得**执行任何内容解析、关键词提取或提示词生成动作。

---

## 第三步：核心生成逻辑（确认后执行）

### 3.1 输入解析与文章分类

执行以下分类逻辑（按此规则执行）：

```python
import re

def classify_article_type(title, summary):
    text = (title + " " + summary).lower()
    
    # 类型1：药物/疗法对比类
    if re.search(r'vs|对决|pk|对比|还是|哪个好|胜出|头对头', text):
        return "TYPE_PK"
    # 类型2：数据/效果展示类
    if re.search(r'\d+%|减重|下降|降低|提升|增加|倍', text):
        return "TYPE_DATA"
    # 类型3：问答/清单/盘点类
    if re.search(r'几大|问|答|问题|盘点|清单|指南|必看', text):
        return "TYPE_QA"
    # 类型4：避坑/误区/纠正认知类
    if re.search(r'误区|错|坑|别|不要|避开|真相|其实', text):
        return "TYPE_MISCONCEPTION"
    # 默认兜底
    return "TYPE_GENERAL"

### 3.2 提取核心视觉元素与文案
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
        entities = re.findall(r'([A-Za-z\u4e00-\u9fa5]{2,8}(?:肽|鲁肽|格鲁肽|药|片|针))', title)
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
    
    elif article_type == "TYPE_MISCONCEPTION":
        elements["main_subject"] = "打叉的餐盘/对比图(错误vs正确)"
        elements["action"] = "左右对比/错误划掉"
        match = re.search(r'(水煮|生酮|断食|代餐|低碳|高蛋白)', title)
        elements["key_text"] = match.group(1) if match else "避坑"
    
    else:  # TYPE_GENERAL
        elements["main_subject"] = "胶囊/听诊器/医疗图标"
        elements["action"] = "居中展示"
        elements["key_text"] = "科普"
    
    return elements

### 3.3 组装固定模板（强约束输出）
def generate_prompt(article_type, elements):
    # ====== 固定参数（橄榄健康科普专属风格，不可擅自修改）======
    STYLE = "扁平插画风格，医疗健康科普风"
    COLOR = "主色调为#2B7A8A（深青）和#F0F4F8（浅灰蓝），点缀色用#E8845A（暖橙）突出数字"
    COMPOSITION = "中心构图，主体占画面60%，背景为纯色或极简渐变，留白充足"
    NEGATIVE = "禁止：写实照片、3D渲染、复杂背景、人脸、血腥、注射器扎入皮肤、暗黑风格、过度装饰"
    # ======================================================
    
    subject_map = {
        "TYPE_PK": f"画面中央为{elements['main_subject']}，呈左右对峙排列，中间有闪电或天平符号连接",
        "TYPE_DATA": f"画面中央为{elements['main_subject']}，旁边有巨大向下的绿色箭头，{elements['key_number']}数字以超大粗体置于视觉焦点",
        "TYPE_QA": f"画面中央为{elements['main_subject']}，数字/文字'{elements['key_text']}'以序号标签形式分散排列",
        "TYPE_MISCONCEPTION": f"画面中央为{elements['main_subject']}，左侧为错误做法（灰色打叉），右侧为正确做法（绿色对勾）",
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
### 3.4 主执行流程
从用户提供的链接、附件或文本中提取 文章标题 和 核心摘要（若无法提取，仅使用标题，并向用户说明：“摘要信息较少，提示词可能不够精准，建议补充核心数据。”）。

调用 classify_article_type(title, summary) 获取文章类型。

调用 extract_elements(title, summary, article_type) 获取视觉元素。

调用 generate_prompt(article_type, elements) 生成最终提示词。

按【第四步】的格式输出。
## 第四步：最终输出格式（严格约束）
最终输出必须为纯中文，分段清晰，每行以 【】 开头。

不得在最终提示词中包含任何解释性文字（如“这是基于...生成的”），只输出可直接复制粘贴的提示词本体。

在提示词正文之后，空一行，附带工具和比例信息（不计入提示词本体）。
输出示例（严格遵循）：
【风格】扁平插画风格，医疗健康科普风
【构图】中心构图，主体占画面60%，背景为纯色或极简渐变，留白充足
【主体】画面中央为"埃诺格鲁肽 vs 司美格鲁肽"，呈左右对峙排列，中间有闪电或天平符号连接
【色彩】主色调为#2B7A8A（深青）和#F0F4F8（浅灰蓝），点缀色用#E8845A（暖橙）突出数字
【文字】画面中嵌入大号文字：'35%'，字体为无衬线粗体，位于画面正下方或左下角，白色带阴影，确保高辨识度
【禁止】禁止：写实照片、3D渲染、复杂背景、人脸、血腥、注射器扎入皮肤、暗黑风格、过度装饰

【适用工具】Midjourney V6 / Stable Diffusion XL / Flux.1
【宽高比】16:9 (适合微信公众号封面)

##附加约束与提醒
色值锁定：#2B7A8A、#F0F4F8、#E8845A 是品牌固定色，不可擅自调整。

领域二次校验：若用户提供的文章明显不属于医疗健康领域（如财经、娱乐、科技硬件等），即便用户已明确或确认属于“橄榄健康科普”，你也应在生成前主动提醒：“该内容风格与健康科普领域不符，确认要继续生成吗？”（需用户二次确认方可继续）。

占位符兜底：若提取不到任何数字和关键词，key_text 默认使用“科普”，main_subject 默认使用“胶囊/听诊器”。

禁止编造数据：若文章中没有提及具体数字，不得自行编造百分比或数据填入提示词，应留空或不填该元素。
