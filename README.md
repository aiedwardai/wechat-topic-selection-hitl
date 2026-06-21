# 微信公众号自动发文 Skill

一个面向 Hermes 的“开箱即用”技能包。

> 目标：装这个 Skill，能在单个 cron 任务里完成公众号人类在环的自动发布流程。

---

## ✨ 这是什么

这是我把“公众号自动发文”整套链路，重新打包成的一个独立 Skill：

`wechat-auto-publishing-one-click`

它不是单纯的“写文章模板”，而是一整套可复用的内容生产流水线，包含：

- 选题研究
- 子 Agent 并行调研
- GitHub / Twitter 真实素材抓取
- 独立真实封面规则
- 架构图 PNG 生成
- 中文公众号文章模板
- preflight 发布前校验
- 微信草稿发布模板
- bootstrap 自举脚本

---

## 🚀 它能解决什么问题

如果你也在做 AI / 开源项目公众号，通常会踩这些坑：

- 文章能写出来，但结构很散
- 会自动发稿，但封面像 AI 拼出来的假图
- 有内容，没有真正可用的项目架构图
- 想做单 cron 自动化，但流程一长就乱
- 想分享给别人，还得再装一堆 Skill 和脚本

这个仓库就是专门解决这些问题的。

---

## 🧩 内置能力

目前内置的核心能力：

1. GitHub 候选项目筛选
2. 用户反馈调研模板
3. 子 Agent 并行调研模板
4. GitHub / README / Twitter 真实图片规则
5. 独立封面规则
6. 架构图 HTML → PNG 渲染
7. 中文公众号文章模板
8. preflight 校验
9. draft_only 发布模板
10. 单 cron 提示词模板
11. bootstrap 自举安装脚本

---

## 📌 强约束

这套 Skill 有几个我强制写死的规则：

- 封面禁止 AI 生图
- 封面必须来自 GitHub 或 Twitter/X 真实项目图
- 架构图最终必须输出 PNG
- 必须在一个 cron 任务内完成，不拆多段 cron
- 默认作者名固定：`老夏的金库`
- 默认发布模式：`draft_only`

这些规则不是装饰，而是为了让成品更像“真正能发的公众号稿件”，而不是测试稿。

---

## 🗂️ 仓库结构

```text
wechat-auto-publishing-one-click/
├── SKILL.md
├── assets/
│   └── architecture_template.html
├── references/
│   ├── assets.md
│   ├── publishing.md
│   ├── runbook.md
│   ├── share-readme.md
│   └── workflow.md
├── scripts/
│   ├── bootstrap.sh
│   ├── make_real_cover.py
│   └── render_arch_png.py
└── templates/
    ├── article-template.md
    ├── cron_prompt.txt
    ├── env.example.txt
    └── publish.mjs
```

---

## 🛠️ 安装方式

把整个 Skill 目录复制到 Hermes：

```bash
cp -r wechat-auto-publishing-one-click ~/.hermes/skills/
```

然后进入目录执行：

```bash
cd ~/.hermes/skills/wechat-auto-publishing-one-click
bash scripts/bootstrap.sh
```

---

## 🧪 环境要求

建议环境至少具备：

- `python3`
- `node`
- `npm`
- `chromium`（或可被 Playwright 拉起的浏览器）

bootstrap 会尽量补这些依赖：

- `pillow`
- `playwright`
- `cairosvg`
- `pyyaml`
- `requests`

但如果你是极简环境，还是要自己确认能安装成功。

---

## ⚙️ 配置方式

参考这个模板：

```text
templates/env.example.txt
```

至少需要你自己填：

- `WECHAT_APP_ID`
- `WECHAT_APP_SECRET`

注意：

不要把真实凭据提交进仓库。

---

## ⏱️ 如何接入 Hermes cron

最简单的方法：

直接把这个文件内容作为 cron prompt：

```text
templates/cron_prompt.txt
```

这样一个 cron 就能完成：

1. 初始化
2. research
3. 素材准备
4. 架构图生成
5. 写文章
6. preflight
7. 草稿发布

---

## 🖼️ 为什么要单独做架构图 PNG

很多“自动发文”方案，最后都停在：

- 有一堆字
- 没有真正可读的系统结构图

这个 Skill 里，架构图是明确阶段，而且强制输出 PNG。

原因很简单：

- PNG 更适合直接插进公众号正文
- 比纯 HTML 中间文件更稳定
- 更容易做 preflight 校验
- 也更符合真实发布素材需求

生成方式：

```bash
python3 scripts/render_arch_png.py \
  --meta output/project_meta.json \
  --notes output/architecture_notes.md \
  --html output/architecture-diagram.html \
  --png images/arch.png
```

---

## 🖼️ 为什么封面禁止 AI 生图

因为 AI 生图虽然快，但很容易出现这些问题：

- 看起来很假
- 信息密度低
- 和项目本身没关系
- 一眼像“生成稿”

真正适合作为封面的，往往还是：

- GitHub README 图
- 仓库首页截图
- Dashboard 截图
- 官方功能图
- Twitter/X 项目实图

所以这里明确规定：

- 不允许 AI 生图当封面
- 不允许纯文字卡片当封面
- 不允许直接把架构图拿去当封面

封面处理脚本：

```bash
python3 scripts/make_real_cover.py \
  --input images/github_repo.png \
  --fallback images/github_readme.png \
  --output cover.png
```

---

## 📄 会产出哪些文件

正常跑完，至少应该看到这些：

- `output/project_candidates.json`
- `output/project_meta.json`
- `output/user_signals.json`
- `output/architecture_notes.md`
- `output/research_report.json`
- `output/asset_report.json`
- `output/architecture-diagram.html`
- `images/arch.png`
- `cover.png`
- `article.md`
- `output/preflight_report.json`
- `output/full_publish_result.json`

这些文件不齐，就说明流程并没有真正完成。

---

## ✅ 成功标准

只有同时满足这些，才算真正成功：

- 已完成并行调研
- 已拿到真实项目图
- 已生成真实封面
- 已生成架构图 PNG
- `article.md` 已写出
- `preflight` 通过
- 微信草稿创建成功

也就是说，不是“发出去了就算完”，而是“成品质量过线才算完”。

---

## 🤝 如果你要分享给别人

最推荐的方式就是：

1. 分享这个 GitHub 仓库
2. 让对方复制 Skill 到 `~/.hermes/skills/`
3. 跑 `bootstrap.sh`
4. 配好 `.env`
5. 把 `templates/cron_prompt.txt` 接进 Hermes cron

这样别人不用再自己一点点拼 Skill、脚本和规则。

---

## 📍 当前状态

这个仓库现在已经是一个：

- 可分享
- 可复用
- 可继续迭代

的一键 Skill 版本。

它不保证在所有系统上都 100% 零配置即用，
但已经把最麻烦的那部分——**规则、流程、脚本、模板、校验逻辑**——都打包进来了。

如果你也在做 Hermes 的公众号自动化，这个仓库应该能直接省掉一大半折腾时间。

---

## 🔗 仓库地址

```text
https://github.com/NanSsye/wechat-auto-publishing-one-click
```

如果这个 Skill 对你有帮助，点个 Star 就行。
