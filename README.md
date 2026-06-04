# write-article

公众号写作流程 skill，适用于 Claude Code。

帮你按照自己的风格写文章：强制检索素材库 → 写草稿 → 优化开头 → 生成标题 → 事实核查 → 去 AI 味 → 沉淀素材。

---

## 安装

```bash
npx skills add sungwong/write-article
```

安装后，把 `skills/` 目录下的三个依赖 skill 也分别安装到 `~/.claude/skills/`：

```bash
cp -r ~/.claude/skills/write-article/skills/fact-check ~/.claude/skills/
cp -r ~/.claude/skills/write-article/skills/humanizer-zh ~/.claude/skills/
cp -r ~/.claude/skills/write-article/skills/web-access ~/.claude/skills/
```

## 使用

触发方式：输入 `/write-article`，或直接说「帮我写文章」「写一篇」「写篇文章」。

第一次使用前需要先完成配置（见下方）。

---

## 配置

在 `~/.claude/skills/write-article/config.md` 中填入你的内容系统路径：

```markdown
# write-article 配置

CONTENT_BASE: /你的路径/content-system

## 素材库目录（相对于 CONTENT_BASE）
核心概念库: 02-素材库/核心概念库/
金句库: 02-素材库/金句库/金句库.md
案例库: 02-素材库/案例库/
已发布: 03-已发布/
方法论: 04-方法论/

## 草稿保存路径（相对于 CONTENT_BASE）
默认草稿目录: 01-内容生产/

## 作者风格库（可选）
作者风格库: 06-品牌/作者风格库/
```

你的内容系统目录可以自由组织，只要在配置里写清楚路径就行。

### 微信公众号配置（使用上传功能时必填）

在 `~/.claude/skills/baoyu-post-to-wechat/EXTEND.md` 中填入你的账号信息：

```yaml
accounts:
  - name: 你的公众号名称
    alias: your_alias
    default: true
    default_publish_method: api
    default_author: 你的名字
    need_open_comment: 1
    only_fans_can_comment: 0
    app_id: 你的AppID
    app_secret: 你的AppSecret
```

AppID 和 AppSecret 在微信公众平台 → 设置与开发 → 基本配置中获取。**此文件包含私密信息，不要提交到 git。**

---

## 写作流程

1. **找核心感受** — 先问你这篇文章真正想说的是什么
2. **读取风格指南** — 内化写作规范后再动笔
3. **检索素材库** — 从核心概念库、金句库、案例库里找相关内容
4. **汇报检索结果** — 列出找到的内容，等你确认要复用哪些
5. **选定文章结构** — 声明用哪种结构（SCQA）及理由
6. **生成草稿** — 保留你的原始细节，不压缩不总结
7. **优化开头** — 前 150 字让读者决定继续读
8. **生成标题** — 5 个候选，每个标注类型和钩子
9. **检查禁用规则** — 对照风格规范主动检查
10. **事实核查** — 调用 fact-check skill，核实所有事实宣称
11. **去除 AI 味** — 调用 humanizer-zh skill 跑一遍
12. **沉淀素材** — 写完后把有价值的内容存入素材库

---

## 依赖 Skills

| Skill | 用途 | 来源 |
|-------|------|------|
| `fact-check` | 文章发布前事实核查 | 本 repo `skills/fact-check/` |
| `humanizer-zh` | 去除 AI 写作痕迹 | 本 repo `skills/humanizer-zh/` |
| `web-access` | 抓取网页内容（热点类文章用） | 本 repo `skills/web-access/` |
| `baoyu-post-to-wechat` | 上传文章到微信公众号 | 本 repo `skills/baoyu-post-to-wechat/` |

---

## 核心原则

- **用户的原话 > AI 的总结**：你说了什么细节，全部保留
- **先检索，再创作**：跳过检索就是在浪费素材库
- **60 分就够**：快速完成，不反复打磨
- **复用优于新建**：找到相关旧内容，先问你要不要复用
