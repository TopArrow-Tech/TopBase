# TopBase

[English](README.md) | 中文

TopBase 是在顶箭（TopArrow）上运行的应用的起步脚手架。它不包含业务代码，作用是给 AI 编程工具（Cursor、Claude Code、Codex 等会读 `AGENTS.md` 的工具）提供平台规则和一套文档结构，让 AI 写出来的应用能在顶箭上跑起来，也方便后续维护。

顶箭在用户自己的电脑上本地运行应用，内置 PHP、Node、Python、Electron 运行时，数据库只用 SQLite。

## 获取顶箭

顶箭支持 Windows 10 x64 及以上，以及 macOS 11 及以上的 Apple 芯片 Mac。

- 官网：<https://www.toparrow.top/cn/>
- 下载：<https://www.toparrow.top/cn/web/download.php>

## 目录内容

```text
./
├── AGENTS.md          给 AI 编程工具的规则：运行时、平台接口、文档、本地 git
├── manifest.json      告诉顶箭怎么安装和运行应用
├── .gitignore
├── _doc/              项目文档，AI 开工前先读，改完后同步更新
│   ├── projectinfo/   应用结构、模块、技术栈
│   ├── changelog/     变更记录
│   ├── plan/          较大功能的实现方案
│   ├── testcase/      测试方案和用例
│   ├── references/    外部 API 和第三方文档
│   ├── writing/       设计文档等输出物
│   └── aicode/        平台规则（runtime.md、git.md、publish_check.md）和参考实现
├── _aiact/            AI 临时代码的草稿区，不入库
└── lang/docs/zh/      AGENTS.md、_doc、_aiact 的中文版
```

## 怎么用

**在顶箭里**：新建应用时会自动用这套脚手架。

**从 GitHub**：点 **Use this template** 或者 clone 本仓库，然后：

1. 在 `manifest.json` 里填好 `id`、`name`、`description`、`runtime.type`。`id` 发布后不能再改。
2. 用 AI 编程工具打开这个目录，描述你要做的应用。工具会读 `AGENTS.md` 并照着做。
3. `README.md`、`README.zh-CN.md`、`LICENSE` 可以删掉，它们说明的是这个仓库，不是你的应用。

`manifest.json` 默认是 PHP 的写法。Node、Python、Electron 的写法见 `_doc/aicode/rules/runtime.md`。

## 中英文

根目录的文件是英文，`lang/docs/zh/` 是中文版。顶箭在中文环境新建应用时，会用中文版替换 `AGENTS.md`、`_doc/`、`_aiact/`，并删掉 `lang/docs/`。

`lang/docs/zh/manifest.json` 不是完整清单，顶箭只把其中的 `name` 和 `description` 合并进根目录的 `manifest.json`。

手动使用时如果要中文：把 `lang/docs/zh/` 下的 `AGENTS.md`、`_doc/`、`_aiact/` 覆盖到根目录（`_doc/` 要整个替换，中英文版有几个目录名不同），需要的话把 `lang/docs/zh/manifest.json` 里的 `name`、`description` 填进根目录清单，再删掉 `lang/docs/`。

## 关于本仓库

本仓库是顶箭内部仓库的镜像。欢迎提 issue 和 pull request，采纳的改动会先在内部合并，下次同步时出现在这里。

`README.md`、`README.zh-CN.md`、`LICENSE` 只存在于 GitHub 仓库，顶箭分发的脚手架包里不带它们。

## 许可证

[MIT-0](LICENSE)。用 TopBase 做出来的应用不需要保留本许可证或任何版权声明。
