---
name: xhs-writer
description: >-
  生成小红书(RedNote)爆款笔记：把主题或用户素材(文字/图片/视频/链接)做成一套 9:16
  竖版图文卡片组，或一段带分镜的视频脚本稿，配 caption + hashtags，落盘交付可直接发布。
  使用时机：用户说 写小红书 / 做小红书笔记 / 小红书图文 / 小红书种草 / 出一套 xhs 卡片 /
  推广这个项目 / 做一套小红书卡片 / 小红书视频脚本 / write a Xiaohongshu note / make a RedNote post。
  核心价值：卖点分析(稀缺性×实用性×可感知)、5种爆款标题公式、素人感设计、真实素材优先。
---

# xhs-writer — 小红书爆款笔记生成（豆包版）

把一个主题 + (可选)用户素材/链接，生成**图文卡片组**或**短视频脚本**，落盘交付。
此 skill 由 GitHub 仓库 `JuneYaooo/xhs-writer-skill` 改造适配豆包：图生图不再依赖
OpenAI API key，改由豆包侧 seedream 图像工具完成；脚本为本地纯 Python，无需外部服务。

## 本 skill 的目录与脚本
- 脚本目录：本仓库 `scripts\` 目录。在豆包等 Agent 环境中使用时，把整个仓库放入对应 skills 目录（如 `~/.user_skills/`），脚本按仓库内相对路径调用。
- 方法论参考：同目录下 `references\` 各 md 文件。
- 可用本地脚本（均纯 Python+Pillow，已就绪，用 `python <script>` 调用）：
  - `analyze_material.py`：素材分类、取分辨率/时长、抽帧 → 生成 materials.json
  - `normalize_slug.py`：目录名/时间戳标准化（勿手算）
  - `text_on_image.py`：单图 + 文字叠卡
  - `collage_3x4.py`：2-4 张图拼卡
  - `crop_watermark.py`：去水印
  - `validate_meta.py`：校验 meta.json（强制，非 0 退出即修再跑）

## 核心理念
小红书读者看的是**卡片组**或**短视频**，不是长文。长文原稿只是中间产物；caption 只是配文。
- 图文帖：3-9 张 9:16 竖版卡（cover + content×N + ending），每张 ≤80 字。
- 视频帖：15-90 秒竖屏脚本 + 封面卡，分镜写进 meta.json.shots。

### 爆款方法论（5大原则）
1. **真实素材优先**：项目截图/对比图/演示图 > 纯 AI 生图
2. **聚焦核心卖点**：得分 = 稀缺性 × 实用性 × 可感知，取 Top1-2
3. **素人感设计**：纯色背景 + 大 emoji + 口语化 > 品牌宣传风
4. **痛点导向**：说"解决什么问题" > 堆功能列表
5. **快速迭代**：V1 → 用户反馈 → V2

## 工作流（按顺序执行，不跳步）

### Step 0 — Intake（必问，一次问完）
收到请求先确认，尽量一条消息问完：
1. 主题 / 目标读者 / 核心观点
2. 输出形态：图文 or 视频（默认图文）
3. 素材：有没有已有文字/图片/视频？贴路径或拖文件；给了链接就先抓取素材
4. 风格：简约清新 / 科技感 / ins风 / 商务 / 文艺复古 / 可爱卡通（默认简约清新）
5. 卡片数量 / 视频时长：图文默认 5-7 张；视频默认 30-60 秒

### Step 0.5 — 卖点/亮点分析（推广类必做）
1. 列出所有功能/亮点
2. 让用户打分（每点稀缺性/实用性/可感知，1-5）
3. 得分 = 三者相乘，排序取 Top1-2 为核心卖点
输出卖点分析表（点 / 稀缺 / 实用 / 可感知 / 得分 / 优先级）。

### Step 1 — 素材清点（有素材才做）
跑 `scripts/analyze_material.py <path>... --out <work>/reference/materials.json --frames-dir <work>/reference/frames`
脚本只做确定性预处理；随后用视觉能力打开每条 image/frame，填 caption 与 usage。
素材按价值排序：对比图 > 功能演示 > 数据图表 > 品牌素材。
使用策略：真实素材 > 图生图(seedream) > 代码叠加 > 纯 AI 生图。

### Step 2 — 采集外部参考（观点/资讯类必做）
搜索同类爆款笔记，提取标题模式、卡片结构、钩子套路；核心数据 ≥2 来源交叉验证。
结果写入 `reference/`。

### Step 3 — 写长文原稿（2000-4000 字）
从 H2 开始写（不写 H1；标题入 meta.json.title）。写完先给用户看，确认主旨再继续。

### Step 4 — 去 AI 化
读 `references/humanizer-zh.md` 五层原则，完整扫描重写原稿，给出质量评分(满分50)。

### Step 4.5 — 图生图处理（有真实素材时必做）
**不用 OpenAI，改由豆包 seedream 工具完成**：
- 有真实素材（对比图/演示图/截图）→ 用 `image_edit` 工具做"保留原素材 + 叠加小红书风格文字"的图生图；
  把素材图作为 image_reference，prompt 里写"保留原图清晰可见，叠加标题与引导文字，小红书素人感风格"。
- 纯文字卡 → 用 `image_gen` 生成（纯色背景 + 大 emoji + 口语化，素人感）。
- 失败降级：重试1次 → 降低分辨率 → 改用 `text_on_image.py` 代码叠字。
比例：图文卡 9:16（或封面 3:4）。

### Step 5 — 分发
#### 5A. 图文帖：拆 3-9 张卡
- 结构：cover + content×N + ending；每张 ≤80 字；一张卡一个论点；数字/对比/金句优先上卡。
- 合成策略（写入 cards[i].synthesis_strategy）：
  `img2img`(有真实素材,seedream 图生图) / `text_on_photo`(单图+钩子,text_on_image.py) /
  `collage`(2-4 图,collage_3x4.py) / `pure_text`(无素材纯文字卡,image_gen) / `ai_generated`(慎用)。
- 素材有水印先跑 `crop_watermark.py`。

#### 5B. 视频帖：写分镜脚本
- 6-12 个分镜，每镜 2-8 秒，累计对齐预期时长。
- 每镜含：narration(口播≤30字)、on_screen_text(屏幕字≤15字)、visual(画面)、material_ref(可选)。
- 仍出一张 cover 卡(3:4)。视频本体由用户侧工具合成，本 skill 只产脚本。

### Step 6 — caption + hashtags + 标题
**标题 5 公式**：痛点+解决方案 / 提问式 / 发现式 / 热点词 / 身份共鸣。
**caption**：100-300 字，hook(数字/提问/惊叹)开头 → 关键信息 → CTA；带 emoji，闺蜜语气。
**hashtags**：5-8 个；核心4个(热点词+核心功能+差异化卖点+目标人群)+辅助(场景词+品类词)；避免 `#生活` 等泛化标签。只写进 meta.json。

### Step 7 — 落盘
输出到用户可见目录（当前对话工作目录或用户指定目录），结构：
```
output/小红书/{YYYY-MM-DD}/{短标题}_{YYYYMMDDHHmm}/
├── {完整标题}.md        # 长文原稿
├── meta.json            # 卡片/分镜/caption/hashtags/materials
├── images/              # 最终卡图(9:16) / 封面
└── reference/           # materials.json / 搜索结果 / 思考过程
```
目录名与时间戳**必须**跑 `scripts/normalize_slug.py "标题" --with-ts`，勿手算。
交付：用 present_files 把卡片图(md 里相对引用)与 meta 交付给用户。

### Step 8 — 校验（强制）
跑 `python scripts/validate_meta.py <work>/meta.json`，非 0 退出就读报错修 meta 再跑，直到 clean。不把未校验产物交给用户。

## 不要做的事
- 不写 H1；标题只放 meta.json.title
- 不在正文末尾写"参考来源/References"；只落到 reference/
- 不在 .md 留"【插入图片：...】"占位；图片下载后用相对路径引用
- 不跳过 Step4(去 AI 化) 与 Step8(validate)
- 不手算目录名/时间戳，一律走 normalize_slug.py
- 不向用户索要任何 API key

## 交付形态
- 图文帖：交付一套 9:16 卡片图 + caption + hashtags + 长文原稿 + meta.json
- 视频帖：交付分镜脚本(meta.json.shots) + 封面卡 + caption + hashtags
- 有素材优先出"真实素材 + 小红书风格"卡片；纯文字主题出素人感卡片。
