---
name: ganlanhealth-wechat-style
description: 橄榄健康科普公众号排版风格完整规范——CSS参数、配色、HTML模板、品牌尾部代码
metadata: 
  node_type: memory
  type: reference
  originSessionId: 462aef9f-4629-4287-bdb7-fd78bc76a8ce
---

# 橄榄健康科普公众号排版风格规范

## 来源

分析 4 篇橄榄健康科普公众号参考文章（2026-06-25），提取共用的排版模式。

参考文章：
1. 多减35%！埃诺格鲁肽VS司美格鲁肽头对头研究
2. 每周一次，让腰围从L到S的减重"神药"
3. 问爆的埃诺格鲁肽！10大核心问题全解答
4. 减肥失败的人，90%都试过水煮菜

## 核心 CSS 参数

```css
/* 正文 */
.article-body {
  font-family: PingFangSC-light, "PingFang SC", system-ui, -apple-system, "Helvetica Neue", "Hiragino Sans GB", "Microsoft YaHei UI", "Microsoft YaHei", Arial, sans-serif;
  font-size: 16px;
  color: #3E3E3E;
  line-height: 1.75; /* 28px */
  letter-spacing: 2px;
}

/* 正文段落 */
.article-body p {
  font-size: 16px;
  color: #3E3E3E;
  line-height: 28px;
  letter-spacing: 2px;
  margin-bottom: 0;
}

/* 二级标题 — 橙蓝渐变标签 */
.section-title-tag {
  font-size: 18px;
  font-weight: 700;
  color: #FFFFFF;
  letter-spacing: 2px;
  line-height: 1.75;
}

/* 三级标题 — 青绿药丸 */
.subsection-pill {
  font-size: 18px;
  font-weight: 700;
  color: #47C1A8;
  background-color: #E8FFFB;
  padding: 5px 0 5px 22px;
  border-radius: 0 45px 45px 0;
  letter-spacing: 2px;
}

/* 四级标题 — 居中橙色 */
.sub-heading-orange {
  font-size: 18px;
  font-weight: 700;
  color: #E36C09;
  text-align: center;
  padding: 10px 2em;
  line-height: 1.75;
  letter-spacing: 2px;
}
```

## 完整配色表

| 用途 | 色值 | 说明 |
|------|------|------|
| 品牌主色 | `#FF6827` | 暖橙，品牌标识、CTA |
| 强调色 | `#E36C09` | 深橙，标题强调、链接 |
| 青绿辅助 | `#47C1A8` | Q&A 标题、健康感 |
| 科技蓝 | `#1DA5FB` | 数据、专业感 |
| 卡片浅蓝 | `#E9F8FF` | 信息卡片背景 |
| 卡片浅绿 | `#E8FFFB` | 药丸标题背景 |
| 答案浅绿 | `#F0FFFD` | Q&A 答案卡片背景 |
| 卡片浅橙 | `#FFF7F0` | 健康 tips 背景 |
| 正文文字 | `#3E3E3E` | 深灰（非纯黑） |
| 二级文字 | `#333333` | 次要信息 |
| 图片标注 | `#B2B2B2` | 12px 图注 |

## 品牌尾部 HTML 模板

每篇文章末尾必须包含以下 HTML（可直接复用）：

```html
<!-- 关注引导 -->
<section style="text-align: center; font-family: PingFangSC-light; color: #333333; letter-spacing: 2px; line-height: 1.75;">
  <p style="font-size: 17px; color: rgba(0,0,0,0.9); margin: 0; padding: 0;">
    <strong>-关注我们-</strong>
  </p>
  <p style="margin: 8px 0; padding: 0;">👇</p>
  <p style="font-size: 16px; color: #3E3E3E; margin: 0; padding: 0;">
    分享📨、点赞👍、推荐3连击
  </p>
  <p style="font-size: 16px; color: #3E3E3E; margin: 0; padding: 0;">
    后续新文章才会及时被你看到
  </p>
  <p style="font-size: 16px; color: #3E3E3E; margin: 0; padding: 0;">
    我们可不想错过任何一个关心你的机会
  </p>
</section>

<!-- CTA 按钮 -->
<section style="text-align: center; margin-top: 16px;">
  <span style="background-color: #E36C09; color: #FFFFFF; font-size: 16px; font-weight: 700; padding: 4px 8px; letter-spacing: 2px;">
    点赞分享❣，一起拿捏好身材🤏
  </span>
</section>
```

## 温馨提示模板

```html
<section style="background-color: #FFFFFF; padding: 15px 12px; font-family: PingFangSC-light; font-size: 16px; color: #3E3E3E; letter-spacing: 2px; line-height: 1.75;">
  <p style="margin: 0; padding: 0;">
    <strong>温馨提示：</strong>本文仅为健康科普，不构成任何用药/医疗建议。具体用药请务必在专业医生指导下进行，切勿自行购买或滥用。
  </p>
</section>
```

## 信息卡片模板

```html
<!-- 浅蓝信息卡 -->
<section style="background-color: #E9F8FF; padding: 15px 12px; font-family: PingFangSC-light; font-size: 16px; color: #3E3E3E; letter-spacing: 2px; line-height: 1.75;">
  <!-- 卡片内容 -->
</section>

<!-- 浅绿答案卡 -->
<section style="background-color: #F0FFFD; padding: 15px 12px; font-family: PingFangSC-light; font-size: 16px; color: #3E3E3E; letter-spacing: 2px; line-height: 1.75;">
  <!-- 答案内容 -->
</section>

<!-- 暖橙提示卡 -->
<section style="background-color: #FFF7F0; padding: 15px 12px; font-family: PingFangSC-light; font-size: 16px; color: #3E3E3E; letter-spacing: 2px; line-height: 1.75;">
  <!-- 提示内容 -->
</section>
```

## 文章类型判断

| 类型 | 特征 | 配色系 | 标题样式 |
|------|------|--------|---------|
| 数据解读/对比 | 有临床数据、百分比、头对头研究 | 蓝色系 (#E9F8FF) | 橙蓝渐变标签条 |
| Q&A 问答 | "10大问题""全解答""FAQ"格式 | 青绿色系 (#47C1A8) | 青绿药丸形状 |
| 痛点科普/带货 | 生活场景、减肥失败、产品植入 | 暖橙色系 (#FFF7F0) | 居中橙色文字 |

## 文章结构模板

```
[开篇钩子] — 痛点场景或冲击数据，2-3段短句
    ↓
[正文展开] — 每个大段用标题条标记，关键信息放卡片
    ↓
[温馨提示] — 健康科普类强制（白色背景区分）
    ↓
[参考文献] — 如有，14px，编号格式
    ↓
[-关注我们-] — 品牌尾部固定模板
    ↓
[CTA按钮] — 橙底白字
```

## 相关资源

- Brand Profile: `~/.config/md2wechat/brand.md`
- 排版 Skill: `.claude/skills/ganlanhealth-format/SKILL.md`
- 发布 Skill: `.claude/skills/ganlanhealth-publish/SKILL.md`
- 飞书发布 Skill: `.claude/skills/ganlanhealth-feishu-draft/SKILL.md`
- 参考文章分析: [[ganlanhealth-reference-analysis]]
