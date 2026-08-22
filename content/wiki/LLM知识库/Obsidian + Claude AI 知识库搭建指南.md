# Obsidian + Claude AI 知识库搭建指南

> 基于 Karpathy LLM Wiki 方法论，30 分钟搭好 AI 知识管理员。你只管扔素材和提问，AI 负责整理归档。

## 系统概览

| 组件 | 作用 | 费用 |
|------|------|------|
| **Obsidian** | 笔记软件，知识库载体 | 免费 |
| **Claudian 插件** | 让 Claude 住进 Obsidian | 免费 |
| **Claude Code** | AI 引擎（命令行） | 包含在 Pro/Max 中 |
| **Claude 账号** | AI 能力 | Pro $20/月 或 Max $100/月 |
| **CLAUDE.md** | 规则文件，定义三个触发行为 | — |

> 费用说明：Pro 每天 ingest 1-2 篇素材基本够用。频繁遇到限速时升级 Max。

## 搭建步骤（4 阶段 12 步）

### 阶段一：安装环境

**Step 1：安装 Obsidian**
下载 [obsidian.md](https://obsidian.md/)，创建新 vault。注意 vault 路径不要有中文和空格。

**Step 2：安装 Claude Code**
```bash
npm install -g @anthropic-ai/claude-code
```
- 如果提示 `npm: command not found`，先装 [Node.js LTS](https://nodejs.org/)
- 装完运行 `claude --version` 验证

**Step 3：安装 Claudian 插件**
Obsidian → 设置 → 社区插件 → 搜索「Claudian」→ 安装启用

**Step 4：配置 Claudian**
- CLI Path: 通常自动检测，若失败用 `which claude` 找路径
- Safe Mode: 推荐 `acceptEdits`（AI 改文件前让你确认）

### 阶段二：搭建知识库结构

**Step 5：创建目录结构**
```
vault/
├── raw/          ← 原始素材（AI 只读，不修改）
├── wiki/         ← AI 维护的知识库
│   ├── index.md  ← 全局索引
│   └── log.md    ← 操作日志
└── assets/       ← 配图资源
```

**Step 6：创建 CLAUDE.md**
把规则文件放在 vault 根目录。这是整个系统最核心的文件——告诉 AI 知识库的结构和三个触发行为怎么执行。

**Step 7：建第一个主题目录**
在 `raw/` 和 `wiki/` 下各建一个主题。主题可以后续随时添加。

### 阶段三：日常使用

**Step 8：Ingest（扔素材）**

素材形式不限：

| 类型 | 操作 |
|------|------|
| 文章全文 | 直接复制粘贴 |
| 网页链接 | 贴 URL，AI 尝试抓取 |
| PDF 文件 | 拖入 `raw/`，告诉 AI 消化 |
| 视频笔记 | 粘贴转录或笔记 |
| 截图 | 拖入 vault 让 AI 查看 |

> 建议一次一篇文章，质量更好。AI 放到不合适目录可以纠正。

**Step 9：Query（提问）**
```
我的 wiki 里关于 XX 有什么？
根据我的 wiki，总结一下 XX
对比一下 A 和 B 的区别
```
AI 会索引定位 → 深入读取 → 综合回答并引用页面。加"存下来"可归档为新文章。

**Step 10：Lint（体检）**
```
lint wiki
```
每周跑一次，修复索引/链接问题，报告事实矛盾。

### 阶段四：进阶用法

**Step 11：自定义 CLAUDE.md**
用一两周后，根据你的领域添加规则：
- 主题分类定义
- 命名约定
- 交叉引用规则

**Step 12：知识库复合增长**
- 第 1 篇：一篇文章
- 第 10 篇：文章间开始交叉引用
- 第 50 篇：AI 能从多篇文章综合出你自己都没想到的答案
- 第 100 篇：知识库比你的记忆都好用

## 质量自检

- [ ] Obsidian 能正常打开 vault
- [ ] Claudian 对话框能正常响应
- [ ] vault 根目录有 `CLAUDE.md`
- [ ] 有 `raw/`、`wiki/`、`assets/` 三个目录
- [ ] `wiki/` 里有 `index.md` 和 `log.md`
- [ ] 能成功 ingest 一篇素材
- [ ] 能成功 query 一个问题
- [ ] 能成功跑一次 lint

## 命名约定

- **raw 文件** — 保持原名（PDF/视频/xlsx/图片永不改名）
- **wiki 文件** — 描述性命名，不强制日期格式
- **wiki 子目录** — 深度不限

## 数据安全

- Obsidian 文件存在本地，不依赖云端
- Claude API 处理相关文件内容，不永久存储在 Anthropic
- 敏感信息（密码、API Key）不要放进 vault
- 即使换 AI 服务，底层 Markdown 文件也完全可迁移

## 相关资源

- See Also: [[wiki/LLM知识库/LLM Wiki 方法论]] — 本指南背后的理论基础
- [Claudian 插件](https://github.com/YishenTu/claudian)
- [Claude Code 高阶指南](https://github.com/helloianneo/claude-code-handbook)
- [Karpathy LLM Wiki 原始 Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 原始素材: `raw/第二大脑/Obsidian + Claude AI 个人知识库搭建指南.md`

---

*Updated: 2026-07-18*
