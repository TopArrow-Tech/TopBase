# TopBase

English | [中文](README.zh-CN.md)

TopBase is the starter scaffold for apps that run on TopArrow. It contains no application code. It gives AI coding tools (any tool that reads `AGENTS.md`, such as Cursor, Claude Code, or Codex) the platform rules and a documentation layout, so the app they build runs on TopArrow and stays easy to maintain.

TopArrow runs apps locally on the user's own computer. It ships PHP, Node, Python, and Electron runtimes, and SQLite is the only database.

## What's inside

```text
./
├── AGENTS.md          Rules for AI coding tools: runtime, platform contract, docs, local git
├── manifest.json      Tells TopArrow how to install and run the app
├── .gitignore
├── _doc/              Project notes the AI reads first and keeps up to date
│   ├── projectinfo/   App structure, modules, and stack
│   ├── changelog/     Running change log
│   ├── plan/          Plans for larger features
│   ├── testcase/      Test plans and cases
│   ├── references/    External API and third-party docs
│   ├── writing/       Design notes and other writing
│   └── aicode/        Platform rules (runtime.md, git.md, publish_check.md) and examples
├── _aiact/            Scratch space for temporary AI code, not committed
└── lang/docs/zh/      Chinese version of AGENTS.md, _doc, and _aiact
```

## Using it

**In TopArrow**: creating a new app starts from this scaffold automatically.

**From GitHub**: click **Use this template** or clone the repository, then:

1. Fill in `id`, `name`, `description`, and `runtime.type` in `manifest.json`. `id` cannot change after the app is published.
2. Open the folder in your AI coding tool and describe the app you want. The tool reads `AGENTS.md` and follows it.
3. You can delete `README.md`, `README.zh-CN.md`, and `LICENSE`. They describe this repository, not your app.

`manifest.json` ships in the PHP shape. For Node, Python, or Electron, see `_doc/aicode/rules/runtime.md`.

## Languages

The files at the root are in English. `lang/docs/zh/` holds the Chinese version. When TopArrow creates an app in a Chinese environment, it replaces `AGENTS.md`, `_doc/`, and `_aiact/` with the Chinese copies and removes `lang/docs/`.

To do the same by hand, copy `AGENTS.md`, `_doc/`, and `_aiact/` from `lang/docs/zh/` over the ones at the root (replace `_doc/` as a whole, since some folder names differ), then delete `lang/docs/`.

## About this repository

This repository is a mirror of TopArrow's internal repository. Issues and pull requests are welcome. Accepted changes are merged internally and appear here with the next sync.

`README.md`, `README.zh-CN.md`, and `LICENSE` exist only in this GitHub repository. They are left out of the scaffold package TopArrow distributes.

## License

[MIT-0](LICENSE). Apps built from TopBase do not need to keep this license or any copyright notice.
