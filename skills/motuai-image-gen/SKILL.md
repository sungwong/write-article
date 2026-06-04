---
name: motuai-image-gen
description: |
  用魔兔AI（motuai.cn）生成公众号配图。支持海报设计、演示文稿、思维导图等30+类型，
  手绘/扁平/科技等多种风格。通过浏览器CDP操作，需要用户已登录魔兔AI。
  触发方式：/motuai-image-gen、「帮我生图」「生成配图」「做一张图」「用魔兔生图」
  Trigger: /motuai-image-gen, "帮我生图", "生成配图", "做一张图"
---

# motuai-image-gen：魔兔AI生图

用魔兔AI网页版生成公众号配图，通过 web-access skill 的 CDP 模式操作浏览器。

---

## 前置条件

**必须先加载 web-access skill：**
```
必须加载 web-access skill 并遵循指引
```

然后检查 CDP 可用性：
```bash
node "$CLAUDE_SKILL_DIR/../web-access/scripts/check-deps.mjs"
```

---

## 第一步：确认生图需求

向用户确认以下参数（已有的跳过不问）：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 内容 | 要可视化的文字内容 | 必填 |
| 类型 | 见下方类型列表 | 海报设计 |
| 风格 | 见下方风格列表 | 手绘 |
| 比例 | 4:3 / 9:16 / 5:4 / 1:1 | 4:3 |
| 画质 | 4K GPT / 2K Banana Pro | 4K GPT |

**常用类型：**
海报设计、演示文稿、思维导图、信息图表、流程指南、漫画故事、读书笔记、
时间线、对比分析、教程指南、概念地图、视觉总结、故事板、Logo设计、人物档案

**风格列表：**
手绘（默认）、扁平、插图、马克笔、顺手手绘、科技、复古、卡通、商务、插画

**公众号封面推荐配置：** 类型=海报设计，比例=4:3，风格=手绘或扁平

---

## 第二步：打开魔兔AI

```bash
TARGET=$(curl -s "http://localhost:3456/new?url=https://motuai.cn/" | python3 -c "import sys,json; print(json.load(sys.stdin)['targetId'])")
sleep 2
```

截图确认页面已加载，检查是否已登录（左下角显示用户ID则已登录）。

**未登录时：** 告知用户「请在 Chrome 中登录 motuai.cn，完成后告诉我继续。」

---

## 第三步：设置类型

点击输入框上方的类型标签（默认显示当前选中类型）：

```bash
# 点击当前类型标签，展开类型面板
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "类型")?.click()
'
sleep 1
# 选择目标类型
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d "
Array.from(document.querySelectorAll(\"*\")).find(el => el.innerText?.trim() === \"$TYPE\")?.click()
"
sleep 1
```

---

## 第四步：设置风格

```bash
# 展开风格面板
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
Array.from(document.querySelectorAll("*")).find(el => el.innerText?.trim() === "风格")?.click()
'
sleep 1
# 选择目标风格
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d "
Array.from(document.querySelectorAll(\"*\")).find(el => el.innerText?.trim() === \"$STYLE\")?.click()
"
sleep 1
```

---

## 第五步：输入内容并生成

```bash
# 找到输入框并填入内容
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d "
const textarea = document.querySelector('textarea, [contenteditable=true], input[type=text]');
if (textarea) {
  textarea.focus();
  textarea.value = '$CONTENT';
  textarea.dispatchEvent(new Event('input', {bubbles: true}));
}
"
sleep 1
# 截图确认内容已填入
```

截图确认内容正确后，点击生成按钮：

```bash
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
// 找橙色的生成按钮
const btn = document.querySelector("button[class*=submit], button[class*=generate], [class*=btn-primary]");
btn?.click();
'
```

---

## 第六步：等待生成并下载

生成通常需要 10-30 秒，每隔 5 秒截图检查进度：

```bash
sleep 10
curl -s "http://localhost:3456/screenshot?target=$TARGET&file=/tmp/motuai_result.png"
```

**生成完成的判断：** 页面出现完整图片，不再有加载动画。

图片生成后，从 DOM 提取图片 URL 并下载：

```bash
curl -s -X POST "http://localhost:3456/eval?target=$TARGET" -d '
// 找生成结果图片
Array.from(document.querySelectorAll("img")).filter(img => 
  img.src && img.naturalWidth > 200
).map(img => img.src).join("\n")
'
```

下载到本地：
```bash
curl -L "$IMG_URL" -o ~/Desktop/"$FILENAME".png
open -a "Preview" ~/Desktop/"$FILENAME".png
```

---

## 第七步：关闭 tab

```bash
curl -s "http://localhost:3456/close?target=$TARGET"
```

---

## 配置（config.md）

在 `~/.claude/skills/motuai-image-gen/config.md` 中可设置默认值：

```markdown
# motuai-image-gen 配置

默认类型: 海报设计
默认风格: 手绘
默认比例: 4:3
默认画质: 4K GPT
默认保存目录: ~/Desktop/
```

---

## 重要原则

- **CDP 直接操作**：魔兔AI是动态渲染页面，必须用 CDP，不能用 WebFetch
- **等待生成完成再截图**：生图有延迟，不要过早截图
- **文件名有意义**：保存时用内容关键词命名，不用时间戳
- **生成失败时**：截图给用户看，不要自行猜测原因
