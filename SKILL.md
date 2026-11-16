---
name: poster-style-clone
description: 海报风格克隆与同风格海报生成。当用户提供若干参考海报图片（外加一张产品/主体图），要求"总结海报风格特点"并"生成一张同风格类型的海报"时使用本技能。触发词：海报风格、同风格海报、仿海报、风格迁移海报、poster style、生成同款海报。
agent_created: true
---

# Poster Style Clone — 海报风格克隆生成

将用户提供的参考海报风格要素提炼成结构化分析，再用 ImageGen 工具把指定主体图合成为同风格海报。

## 工作流程

### 第一步：收集与区分素材

用户输入通常包含两类图片，必须先区分：

- **参考海报**（1-N 张）：用于提炼风格。特征是带有排版、文字、构图设计的完整海报。
- **主体图**（1 张）：新海报的核心对象。特征是产品/物体照片（通常白底或纯净背景）。

若用户未明确指出哪张是主体图，取最后一张或唯一一张"无排版设计的纯物体图"作为主体图；判断有歧义时向用户确认，不要猜。

### 第二步：风格分析（先输出分析，再生成）

按 `references/style-framework.md` 的六个维度逐一分析参考海报，并以 markdown 列表形式向用户输出风格总结：

1. 标题字体与占比（超大字？衬线/无衬线？占画面宽度比例）
2. 图文层次关系（照片叠压文字？文字穿插？谁前谁后）
3. 手写体/花体点缀元素
4. 配色方案（背景色与主体色的关系：同色系/互补撞色；总色数）
5. 版式结构（角落信息块位置、分隔线、色卡、编号等编辑元素）
6. 整体气质（瑞士极简、杂志编辑、复古等）

分析要具体到可执行的程度——结论将直接转化为生成 prompt。

### 第三步：调用 ImageGen 生成海报

通过 ToolSearch 加载 ImageGen，再用 DeferExecuteTool 调用。参数策略：

- **image1**：主体图（保留主体原貌）
- **image2**：选一张最具代表性的参考海报作为版式参照
- **input_fidelity**：`high`（保住主体与版式）
- **size**：`1024x1536`（竖版海报）
- **quality**：`high`
- **style**：`graphic design poster`

Prompt 必须是英文、自包含，并按以下模板逐项填入第二步的分析结论：

```
Minimalist Swiss editorial poster design.
Keep the [主体描述] from image1 exactly as the hero subject.
Background: flat solid [背景色] color, monochrome, no texture.
Giant bold [字体描述] word "[标题词]" in [文字颜色] fills the width of the poster,
the [主体] photo is positioned HIGH and CENTERED as the visual center of the poster, sitting ON TOP OF the giant word and covering roughly the upper half of the letterforms - letters clearly run BEHIND the subject with strong occlusion; the title may be significantly covered by the subject (IMPORTANT COMPOSITION RULE, user-mandated).
A SMALL delicate elegant cursive handwritten script word "[点缀词]" in [与主标题明显不同的颜色], about one third of the big title's letter height and spanning only the central third of the poster width, like a light little signature. It is placed ON TOP OF and crossing over the middle of the big title, layered in front of the letters, NOT floating separately above the title and NOT oversized.
Small magazine-style info text blocks in the corners: [左上内容], [右上内容], [左下内容], [右下角为时间表达而非日期].
[右下角元素（用户明确要求，默认强制）：不放任何色块/色卡/实心方块装饰，右下角用时间表达，如 the time "07:30 AM" in clean typography。 Absolutely NO color swatch square, NO color chip, NO solid square accent anywhere on the poster.]
Only [N] colors total: [色1], [色2], [主体色].
Flat graphic design, high contrast, poster typography layout, vertical format.
```

标题词选择规则：优先用主体的英文单词或核心卖点词（橙子→ORANGE、冰棒→COOL、番茄→FRESH/TOMATO），3-8 个字母为佳。

**生成前必须告知用户：单张图片生成约消耗 5-10 积分。**

### 第四步：校验与交付

1. 用 ViewImage 查看生成结果，核对：主体是否保留、大标题是否完整可读、图文叠压层次是否成立、角落信息块是否就位、文字有无乱码。
2. 明显缺陷（乱码文字、主体变形、层次错乱）→ 修正 prompt 后重新生成一次，最多重试一次。
3. 通过后用 present_files 呈现，并在回复中包含：风格总结要点、生成海报的文件路径、本次积分消耗提示、可调整项（换标题词/背景色/文案）。

## 注意事项

- 用户用中文交流时，分析总结和最终回复用中文输出；仅 ImageGen prompt 用英文。
- **花体点缀规则（用户明确要求，默认强制）**：点缀字颜色必须与大标题明显不同（可选点缀色：柔粉、金黄、天蓝等，从主体取色呼应）；必须叠压覆盖在大标题字母之上，不得单独漂浮在标题上方的空白处；字号要小而轻盈，约为大标题字高的 1/3、只横跨画面中部约 1/3 宽度，不得喧宾夺主。
- **主体遮挡规则（用户明确要求，默认强制）**：主体照片必须位于画面视觉中心（偏上居中），明显叠压大标题——可遮住字母上半部甚至大半，字母清晰可见地"藏"在主体后面；不能只是轻微接触字母边缘。
- **主体细节参与遮挡（经验规则）**：若主体有延伸性细节（滴落的酱汁、飘出的丝带、垂下的绳线等），在 prompt 中让该细节越过主体轮廓、流过或垂挂在字母表面，使细节本身成为遮挡层次的一部分（参考：STICKY 海报中焦糖垂滴流过字母）。
- **文案防乱码（经验规则）**：左下角段落只写一句短句，并在 prompt 中明确 "exactly this short text and nothing else, spelled correctly"；长段落容易生成乱码。校验发现乱码时，缩短文案重试一次。
- **右下角规则（用户明确要求，默认强制）**：不放任何色块/色卡/实心方块装饰；右下角用时间表达（如 "03:30 PM"），不用日期，且 prompt 中注明 on a single line。
- 若用户对生成结果提出修改意见，按第三步模板只改动对应变量后重新生成，不要推翻整体结构。
- 若参考海报多于 3 张且风格不一致，先指出差异并请用户指定以哪张为准。
