# Continue 项目整理

> 本文面向首次接触代码库的维护者，按“功能定位、目录结构、依赖关系、常用命令、开发入口”整理当前仓库。仓库根目录已经存在 `docs/`（面向用户的产品文档内容）；本目录 `doc/` 用于存放代码库梳理类文档。

## 1. 项目定位

Continue 是一个开源 AI 编程代理，主要交付形态包括：

- **VS Code 扩展**：编辑器内的聊天、编辑、自动补全、代码库上下文等能力，源码位于 `extensions/vscode/`。
- **JetBrains 插件**：历史插件实现，源码位于 `extensions/intellij/`。
- **共享核心能力**：模型适配、上下文、索引、diff、工具调用、配置解析等跨端复用逻辑，源码位于 `core/`。
- **Web GUI**：嵌入 IDE/插件中的 React 前端界面，源码位于 `gui/`。

## 2. 顶层目录速览

| 路径                      | 作用                                                            |
| ------------------------- | --------------------------------------------------------------- |
| `actions/`                | GitHub Action / 自动化评审相关说明与配置。                      |
| `binary/`                 | 将核心能力打包为可执行二进制的构建工程，依赖 `core/`。          |
| `core/`                   | Continue 共享核心库，供 VS Code、GUI、binary 等复用。           |
| `doc/`                    | 代码库梳理文档，本文件所在目录。                                |
| `docs/`                   | 面向用户的产品文档、指南、参考手册、图片和站点配置。            |
| `eval/`                   | 评测、实验或质量验证相关内容。                                  |
| `extensions/intellij/`    | JetBrains 插件源码。                                            |
| `extensions/vscode/`      | VS Code 扩展源码、打包、E2E 测试和扩展声明。                    |
| `gui/`                    | React + Vite 前端 UI，通常嵌入扩展或其他宿主中。                |
| `manual-testing-sandbox/` | 手工测试用示例项目和沙盒。                                      |
| `media/`                  | README、市场页或通用展示所需媒体资源。                          |
| `packages/`               | 多个可复用 npm 包，例如配置类型、YAML 配置、fetch、LLM 信息等。 |
| `scripts/`                | 仓库级脚本，例如依赖安装、包构建、热点分析等。                  |
| `sync/`                   | 同步相关模块说明和源码。                                        |

## 3. 核心模块说明

### 3.1 `core/`：共享业务内核

`core/` 是跨端复用最多的 TypeScript 包，`package.json` 描述为“可在 Web、VS Code 或 Node.js 间共享的 Continue Core”。它主要覆盖：

- LLM 与 provider 适配：OpenAI、Anthropic、AWS Bedrock、Ollama、Replicate 等。
- 配置加载与解析：继续复用 `@continuedev/config-types`、`@continuedev/config-yaml`。
- 上下文与索引：代码库索引、向量库、tree-sitter、文档读取等。
- AI 编程能力：聊天、编辑、diff、自动补全、next edit、工具调用等。
- 运行时工具：网络请求、证书、代理、日志、SQLite、Postgres、MCP SDK 等。

常用命令：

```bash
cd core
npm run tsc:check
npm run test
npm run build
```

### 3.2 `extensions/vscode/`：VS Code 扩展

VS Code 扩展包名为 `continue`，提供编辑器内的 Continue 视图、命令、语言声明、配置项、自动补全、quick actions、next edit、索引控制和 E2E 测试能力。它依赖 `core`、`gui` 和若干 VS Code/Electron 生态工具。

常用命令：

```bash
cd extensions/vscode
npm run tsc:check
npm run vitest
npm run esbuild
npm run package
```

### 3.3 `gui/`：前端界面

`gui/` 是 React + Vite 应用，用于渲染 Continue 的聊天、编辑、上下文选择、配置、模型选择等界面。它使用 Redux、React Router、Tiptap、Markdown/代码高亮、Mermaid、Tailwind 等前端依赖。

常用命令：

```bash
cd gui
npm run dev
npm run tsc:check
npm run test
npm run build
```

### 3.4 `packages/`：共享 npm 包

| 包                               | 目录                          | 作用                                                          |
| -------------------------------- | ----------------------------- | ------------------------------------------------------------- |
| `@continuedev/config-types`      | `packages/config-types/`      | 使用 Zod 定义配置类型和校验结构。                             |
| `@continuedev/config-yaml`       | `packages/config-yaml/`       | YAML 配置解析、构建和 JSON Schema 生成。                      |
| `@continuedev/sdk-generator`     | `packages/continue-sdk/`      | Continue SDK 生成器，包含 TypeScript/Python client 生成流程。 |
| `@continuedev/fetch`             | `packages/fetch/`             | 统一 fetch、代理、证书相关网络访问封装。                      |
| `@continuedev/llm-info`          | `packages/llm-info/`          | LLM 元信息与模型信息维护。                                    |
| `@continuedev/openai-adapters`   | `packages/openai-adapters/`   | 将多家 provider 适配到 OpenAI 风格接口/消息格式。             |
| `@continuedev/terminal-security` | `packages/terminal-security/` | 终端命令安全评估，依赖 `shell-quote`。                        |

### 3.5 `binary/`：二进制打包

`binary/` 通过 `pkg`、`esbuild`、`ncc` 等工具将 Node.js 逻辑打包为二进制，包含 tree-sitter、SQLite、向量库等运行时资产。它适合需要脱离源码运行核心能力的分发场景。

### 3.6 `docs/`：文档内容

`docs/` 存放面向用户的 MDX 文档内容，包括 IDE 扩展、autocomplete、chat、reference、guides、troubleshooting、FAQ 等。

## 4. 依赖关系概览

```text
用户入口
├─ extensions/vscode ──┐
├─ extensions/intellij │
├─ gui ────────────────┼─> core ──> packages/*
└─ binary ─────────────┘
```

更细一点看：

- `core` 是主要业务中枢，直接依赖多个 `packages/*` 包。
- `extensions/vscode` 使用 `core` 和扩展宿主 API，并将 `gui` 作为用户界面的一部分。
- `gui` 使用 `core` 与配置/安全相关包，提供跨宿主 UI。
- `binary` 使用 `core`，并额外处理 native/runtime 资产打包。

## 5. 技术栈与关键依赖

### 5.1 语言与构建

- **TypeScript**：仓库主体语言，根目录和各子包均有独立 `tsconfig.json`。
- **Node.js**：多个包要求 Node `>=20.20.1`。
- **npm**：仓库使用多个 `package-lock.json`，不是 pnpm/yarn workspace。
- **Vite**：用于 `gui/` 前端开发和构建。
- **esbuild / pkg / ncc**：用于扩展、binary 等打包流程。

### 5.2 AI / 模型 / 协议

- Provider SDK：`openai`、`@anthropic-ai/sdk`、`@aws-sdk/*`、`ollama`、`replicate`、Google/Azure/DeepSeek/xAI 相关适配包等。
- MCP：`@modelcontextprotocol/sdk`、`@modelcontextprotocol/ext-apps`。
- 向量与索引：`vectordb`、`sqlite/sqlite3`、`tree-sitter-wasms`、`web-tree-sitter`、`@xenova/transformers`。

### 5.3 前端与编辑器

- React、React DOM、Redux Toolkit、React Redux。
- Tiptap、Markdown/rehype/remark、Mermaid、lowlight、styled-components、Tailwind。
- VS Code 扩展 API、Electron rebuild、ripgrep 等。

### 5.4 测试与质量

- Jest：`core/`、`binary/`、部分 packages。
- Vitest：`gui/`、`extensions/vscode/` 等。
- ESLint、Prettier、TypeScript typecheck。
- VS Code E2E：`extensions/vscode` 下有完整 e2e 脚本链。

## 6. 常用开发命令

### 仓库根目录

```bash
npm run format:check
npm run format
npm run tsc:watch
```

### 类型检查

```bash
npm run tsc:watch:core
npm run tsc:watch:gui
npm run tsc:watch:vscode
npm run tsc:watch:binary
```

### 单模块构建/测试

```bash
cd core && npm run test && npm run build
cd gui && npm run test && npm run build
cd extensions/vscode && npm run vitest && npm run esbuild
```

## 7. 新人阅读建议

1. 先读根目录 `README.md`，确认项目交付形态和维护状态。
2. 如果关注产品能力，从 `docs/` 中的 `index.mdx`、`ide-extensions/`、`reference/` 开始。
3. 如果关注 VS Code 扩展，从 `extensions/vscode/package.json` 的 `contributes`、`extensions/vscode/src/`、`gui/` 开始。
4. 如果关注核心逻辑，从 `core/package.json`、`core/config/`、`core/llm/`、`core/context/`、`core/indexing/`、`core/tools/` 开始。
5. 如果关注配置体系，从 `packages/config-types/`、`packages/config-yaml/` 和 `docs/reference/` 开始。

## 8. 文档维护约定建议

- `docs/`：继续存放面向最终用户的公开产品文档。
- `doc/`：存放面向维护者/代码阅读者的内部梳理文档，例如架构图、目录说明、依赖说明、调试手册。
- 更新大型模块时，建议同步更新本文件中的目录说明、命令和依赖关系。
