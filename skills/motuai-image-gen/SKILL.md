---
name: motuai-image-gen
description: |
  用宝玉的5维提示词框架生成提示词，再通过魔兔AI（motuai.cn）网页版生图。
  适合公众号封面图、配图生成。需要用户已登录 motuai.cn。
  触发方式：/motuai-image-gen、「帮我生图」「生成配图」「做一张图」「用魔兔生图」
  Trigger: /motuai-image-gen, "帮我生图", "生成配图", "做一张图"
---

# motuai-image-gen：魔兔AI生图

## 工作流程

**第一步：分析内容，确认5维参数 → 第二步：生成提示词 → 第三步：用魔兔AI生图**

---

## 第一步：分析内容，确认5维参数

收到内容（文章/主题/关键词）后，按以下规则自动推荐参数，再向用户确认：

### 自动选择规则

**类型（Type）**

| 内容信号 | 推荐类型 |
|----------|----------|
| 产品、发布、公告 | hero |
| 框架、系统、方法论、技术 | conceptual |
| 金句、观点、洞察 | typography |
| 哲学、成长、抽象含义 | metaphor |
| 故事、旅程、生活方式 | scene |
| 极简、核心、本质 | minimal |

**色调（Palette）**

| 内容信号 | 推荐色调 |
|----------|----------|
| 个人故事、情感、生活方式 | warm（橙/黄/奶油，友好温暖） |
| 商业、专业、思想领导力 | elegant（深蓝/金/米白，沉稳高级） |
| 技术、系统、架构、代码 | cool（蓝/青/白，理性现代） |
| 娱乐、高端、电影感 | dark（深色背景，高级感） |
| 自然、健康、有机 | earth（绿/棕/米，自然沉静） |
| 产品发布、活动、促销 | vivid（高饱和，活力冲击） |
| 轻柔、创意、童趣 | pastel（低饱和粉彩，温柔） |
| 极简、禅意、专注 | mono（黑白灰，克制） |
| 历史、复古、经典 | retro（复古色，时间感） |
| 电影海报、双色调 | duotone（两色叠加，戏剧感） |

**风格（Rendering）**

| 内容信号 | 推荐风格 |
|----------|----------|
| 现代、科技、公众号图文 | flat-vector（扁平矢量，干净） |
| 手稿、随笔、温暖感 | hand-drawn（手绘，亲切） |
| 艺术、水彩、梦幻 | painterly（绘画感，柔和） |
| 数据、企业、SaaS | digital（数字感，精致） |
| 游戏、复古像素 | pixel（像素风） |
| 教学、教程 | chalk（黑板风） |
| 海报、电影、限定 | screen-print（丝网印刷风） |

**文字层级（Text）**

| 场景 | 文字层级 |
|------|----------|
| 纯视觉封面 | none |
| 标准公众号封面 | title-only（默认） |
| 系列/教程 | title-subtitle |
| 发布、功能列举 | text-rich |

**情绪强度（Mood）**

| 场景 | 情绪 |
|------|------|
| 专业、商务、学术 | subtle（低对比，克制） |
| 一般文章、教育 | balanced（均衡，默认） |
| 发布、活动、冲击 | bold（高对比，强烈） |

### 确认参数

推荐完后告知用户推荐理由，询问是否调整，确认后进入第二步。

---

## 第二步：生成提示词

**生成提示词前，必须读取以下参考文件，获取精确的颜色值、风格特征和构图规则：**

```
~/.claude/skills/motuai-image-gen/references/palettes/{确认的palette}.md   # 精确颜色值
~/.claude/skills/motuai-image-gen/references/renderings/{确认的rendering}.md  # 风格特征
~/.claude/skills/motuai-image-gen/references/types.md                        # 构图规则
~/.claude/skills/motuai-image-gen/references/dimensions/mood.md              # 情绪强度
~/.claude/skills/motuai-image-gen/references/dimensions/text.md              # 文字层级
```

可按需参考：
- `references/visual-elements.md`：主题→图标词汇对照（如"成长"→火箭/植物/箭头）

基于以上参考文件内容，生成一段**中文描述性提示词**，直接粘贴进魔兔AI输入框。

提示词结构：
```
[内容主题]，[类型描述]，[色调描述]，[风格描述]，[构图描述]，[文字要求]，[情绪氛围]
```

示例（type=conceptual, palette=elegant, rendering=flat-vector, text=title-only, mood=balanced）：
```
关于个人IP打造方法论的封面图，概念型构图，展示系统框架，深蓝金米白配色，专业沉稳，
扁平矢量风格，线条简洁干净，主视觉居中，标题文字"[标题]"叠加在图上，均衡对比度，
适合公众号封面，4:3比例
```

**提示词要求：**
- 中文描述，魔兔AI对中文响应更好
- 具体、可操作，避免"漂亮""好看"等模糊词
- 明确比例（4:3 封面 / 9:16 竖版 / 1:1 方图）
- 如有标题文字，写明"标题文字'[XXX]'"

---

## 第三步：用魔兔AI生图（CDP操作）

**必须先加载 web-access skill，启动 CDP Proxy：**
```bash
node ~/.claude/skills/web-access/scripts/check-deps.mjs
```

### 3.1 打开魔兔AI

```bash
TARGET=$(curl -s "http://localhost:3456/new?url=https://motuai.cn/" | python3 -c "import sys,json; print(json.load(sys.stdin)['targetId'])")
sleep 3
```

截图确认页面已加载、左下角显示用户ID（说明已登录）。

**未登录时：** 告知用户「请在 Chrome 中登录 motuai.cn，完成后告诉我继续。」

### 3.2 选择类型（可选）

如果不是默认的"海报设计"，点击类型标签切换：

```bash
# 展开类型面板
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "类型")?.click()
'
sleep 1

# 选择目标类型（替换为实际类型名）
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "海报设计")?.click()
'
sleep 1
```

**魔兔AI可用类型：**
全部、读书笔记、思维导图、信息图表、流程指南、漫画故事、时间线、对比分析、
教程指南、概念地图、视觉总结、关系图、故事板、演示文稿、海报设计、百科词典、
名著导读、诗词解读、公式原理、单词解读、古文解读、历史事件、思维模型、
食物营养、成分说明、商业模式、Logo设计、成语解读、非遗科普、人物档案

### 3.3 选择风格（可选）

```bash
# 展开风格面板
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "风格")?.click()
'
sleep 1

# 选择风格（替换为实际风格名）
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "手绘")?.click()
'
sleep 1
```

**魔兔AI可用风格：** 扁平、插图、手绘、马克笔、顺手手绘、科技、复古、卡通、商务、插画

**5维风格 → 魔兔AI风格对照：**

| 5维风格（rendering） | 魔兔AI风格 |
|---------------------|-----------|
| flat-vector | 扁平 |
| hand-drawn | 手绘 / 顺手手绘 |
| painterly | 插画 |
| digital | 商务 / 科技 |
| pixel | 复古 |
| chalk | 手绘 |
| screen-print | 插图 |

### 3.4 填入提示词并生成

```bash
PROMPT="第二步生成的提示词"
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d "
const el = document.querySelector('textarea') || document.querySelector('[contenteditable=true]');
if (el) {
  el.focus();
  el.value = '$PROMPT';
  el.dispatchEvent(new Event('input', {bubbles: true}));
}
"
sleep 1
```

截图确认内容填入正确后，点击生成按钮。

**生成按钮识别**：魔兔AI的生成按钮是右侧橙色按钮（无文字，只有箭头图标），背景色为 `rgb(243, 113, 32)`：

```bash
# 用背景色找橙色生成按钮（已验证有效）
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
var btns = Array.from(document.querySelectorAll("button"));
var orangeBtn = btns.filter(function(b){
  return window.getComputedStyle(b).backgroundColor === "rgb(243, 113, 32)";
});
var sendBtn = orangeBtn[orangeBtn.length - 1];
sendBtn && sendBtn.click();
sendBtn ? "clicked" : "not found"
'
```

> 如果返回 "not found"，截图查看页面状态再判断。

### 3.5 等待生成

**提交后立即跳转到「我的作品」页面等待**，比留在首页更容易判断进度：

```bash
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("button")).find(function(b){
  return b.innerText.trim() === "我的作品";
}).click()
'
sleep 3
curl -s "http://localhost:3456/screenshot?target=$TARGET&file=/tmp/motuai_progress.png"
```

生成通常需要 30-60 秒。截图里左上角的图片还在转圈说明未完成，出现完整图片说明完成。

每隔 15 秒截图一次，直到完成：

```bash
sleep 15
curl -s "http://localhost:3456/screenshot?target=$TARGET&file=/tmp/motuai_result.png"
```

### 3.5b 下载图片

**图片存储在 `todaylab.cn/generations/` 下**（非页面 logo）。提取并下载：

```bash
IMGS=$(curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
var imgs = Array.from(document.querySelectorAll("img")).filter(function(img){
  return img.naturalWidth > 300 && img.src && img.src.includes("todaylab.cn/generations");
});
imgs.map(function(img){ return img.src; }).join("\n")
' | python3 -c "import sys,json; print(json.load(sys.stdin).get('value',''))")

# 下载第一张（通常是最新生成的）
IMG_URL=$(echo "$IMGS" | head -1)
FILENAME="魔兔_$(date +%Y%m%d_%H%M)"
curl -L "$IMG_URL" -o ~/Desktop/"$FILENAME".png
open -a "Preview" ~/Desktop/"$FILENAME".png
```

### 3.6 关闭 tab

```bash
curl -s "http://localhost:3456/close?target=$TARGET"
```

---

## 公众号封面推荐配置

| 账号风格 | 类型 | 色调 | 风格 | 比例 |
|----------|------|------|------|------|
| 商业/创业 | conceptual / hero | elegant / cool | flat-vector → 扁平 | 4:3 |
| 个人成长 | metaphor / scene | warm / earth | hand-drawn → 手绘 | 4:3 |
| 方法论/干货 | conceptual | cool / mono | flat-vector → 扁平 | 4:3 |
| 故事/日记 | scene | warm / pastel | painterly → 插画 | 4:3 |

---

## 重要原则

- 提示词用**中文**，魔兔AI对中文描述响应更准确
- 类型和风格选择对最终效果影响最大，确认前多想一步
- 生成失败时截图给用户看，不自行猜测
- 图片文件名用内容关键词命名，不用纯时间戳
