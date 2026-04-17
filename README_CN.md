# soft-ue-cli

[![PyPI version](https://img.shields.io/pypi/v/soft-ue-cli.svg)](https://pypi.org/project/soft-ue-cli/)
[![Python 3.10+](https://img.shields.io/pypi/pyversions/soft-ue-cli.svg)](https://pypi.org/project/soft-ue-cli/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

**从命令行控制 Unreal Engine 5。** soft-ue-cli 是一个 Python CLI 工具，让 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[CodeBuddy](https://www.codebuddy.ai/)、[Cursor](https://www.cursor.com/) 等 AI 编码代理——或任何终端工作流——能够在运行中的 UE5 编辑器或打包构建中生成 Actor、编辑蓝图、检查材质、运行 Play-In-Editor 会话、截屏、性能分析等 50+ 种操作。

一次 pip 安装，一次插件拷贝，零手动编辑器操作。

```
CodeBuddy / Claude Code / Cursor  -->  soft-ue-cli (Python)  -->  HTTP/JSON-RPC  -->  SoftUEBridge 插件（UE 进程内）
```

---

## 为什么选择 soft-ue-cli？

- **AI 原生 UE 自动化** —— 专为 Claude Code 等 AI 代理设计，无需人工操作编辑器即可读取、修改和测试 Unreal Engine 项目。
- **50+ 命令** —— 覆盖 Actor、蓝图、材质、StateTree、Widget、资产、PIE 会话、性能分析等。
- **UE 能运行的地方就能用** —— 编辑器、打包构建、Windows、macOS、Linux。
- **单一依赖** —— 仅需 `httpx`，无重型 SDK，无需编辑器脚本配置。
- **团队友好** —— 通过 `SOFT_UE_BRIDGE` 环境变量实现条件编译，不需要桥接功能的成员无需编译它。

---

## 前置条件

| 工具 | 是否必须 | 说明 |
|------|----------|------|
| **Unreal Engine 5.3 – 5.5** | ✅ 必须 | 需要源码编译或从 Epic Launcher 安装的引擎版本 |
| **Visual Studio 2022** | ✅ 必须 | UE5 的 C++ 编译器。安装时需勾选「使用 C++ 的游戏开发」工作负载，以及「Unreal Engine 安装程序」组件 |
| **Python 3.10+** | ✅ 必须 | 用于运行 soft-ue-cli。可从 [python.org](https://www.python.org/downloads/) 下载，或通过 Conda 管理 |
| **Conda 虚拟环境** | ⬜ 可选 | 建议使用 Conda 或 venv 创建隔离环境，避免与系统 Python 包冲突 |
| **Git** | ⬜ 可选 | 用于克隆仓库和版本控制。如果通过 `pip install soft-ue-cli` 安装则不需要 |

### Visual Studio 2022 配置要点

确保在 Visual Studio Installer 中安装以下组件：

- **工作负载**：「使用 C++ 的游戏开发」（Game development with C++）
- **单个组件**：「Unreal Engine 安装程序」（Unreal Engine installer）
- **Windows SDK**：10.0.18362.0 或更高版本

### （可选）使用 Conda 虚拟环境

```bash
# 创建并激活虚拟环境
conda create -n ue-cli python=3.12 -y
conda activate ue-cli

# 在虚拟环境中安装 soft-ue-cli
pip install soft-ue-cli
```

> 💡 使用虚拟环境的好处：避免不同项目之间的依赖冲突，也方便在 AI 代理的终端中指定正确的 Python 环境。

---

## 快速上手

### 1. 安装 CLI

```bash
pip install soft-ue-cli
```

### 2. 配置插件

在 UE 项目目录中打开你的 AI 编码代理（Claude Code、Cursor 等），然后运行：

```bash
soft-ue-cli setup
```

代理会读取输出并自动：
- 将内置的 SoftUEBridge C++ 插件复制到你的项目中
- 在 `.uproject` 文件中启用它
- 将 CLI 使用说明追加到你的 `CLAUDE.md`

### 3. 重新构建并启动 Unreal Engine

配置完成后，重新生成项目文件、重新构建并启动编辑器。查看此日志行以确认启动成功：

```
LogSoftUEBridge: Bridge server started on port 8080
```

### 4. 验证连接

让你的代理运行：

```bash
soft-ue-cli check-setup
```

所有检查应通过：

```
[OK]   Plugin files found.
[OK]   SoftUEBridge enabled in YourGame.uproject.
[OK]   Bridge server reachable.
```

你的 AI 编码代理现在可以控制 Unreal Engine 了。

---

## 工作原理

```
Claude Code
    |
    |  （在终端中运行 CLI 命令）
    v
soft-ue-cli（Python 进程）
    |
    |  HTTP / JSON-RPC 请求
    v
SoftUEBridge 插件（C++ UGameInstanceSubsystem，运行在 UE 进程内）
    |
    |  在游戏线程上调用原生 UE API
    v
Unreal Engine 5 编辑器或运行时
```

**SoftUEBridge** 插件是一个轻量级的 C++ `UGameInstanceSubsystem`，在 UE 启动时在 8080 端口启动一个嵌入式 HTTP 服务器。CLI 向该服务器发送 JSON-RPC 请求，插件在游戏线程上执行相应的 UE 操作，并返回结构化的 JSON 响应。

所有命令输出 JSON 到标准输出（`get-logs --raw` 除外）。退出码 0 表示成功，1 表示错误。

---

## 设计理念

### C++ 核心 —— UE 能运行的地方就能用

桥接是原生 C++ `UGameInstanceSubsystem`，不是 Python 或仅编辑器可用的脚本插件。这意味着它可以在**编辑器、Play-In-Editor 和打包构建**中以完全相同的行为运行。你的游戏能跑，桥接就能跑。

### 可通过 Python 扩展

C++ 核心负责可靠的底层 UE 集成。对于自定义工作流，`run-python-script` 让你可以在 UE 内嵌的 Python 解释器中执行任意 Python 脚本——无需重新编译 C++ 即可扩展桥接功能。

### 最小化桥接，最大化 LLM 自主性

CLI 刻意保持精简。它暴露**原始 UE 能力**（spawn、query、set、call）而非固化的高级工作流。LLM 客户端自带推理能力——它读取 `--help`，理解项目上下文，组合命令来实现目标。桥接不试图变聪明，而是让 AI 去聪明。

### 为什么是 CLI 而非 MCP？

MCP 服务器需要每个客户端的集成和一个运行中的服务器进程。CLI 可以与**任何**能运行 shell 命令的 LLM 工具配合使用——Claude Code、Cursor、Windsurf、自定义代理、Shell 脚本、CI 流水线。一个接口，所有客户端。

---

## 各 AI 客户端配置指南

soft-ue-cli 提供两种接入方式：**CLI 模式**和 **MCP 模式**。

- **CLI 模式**：AI 在终端执行 `soft-ue-cli` 命令。通用性强，但**必须通过 Rule 或提示词告知 AI 工具的存在**，否则 AI 不知道有这个命令可用。
- **MCP 模式**：AI 通过 MCP 协议直接发现和调用工具。配置好 `mcp.json` 后 AI **自动看到所有 50+ 工具**，不需要额外写 Rule。

```
CLI 模式：  AI → 读 Rule 知道命令 → shell → soft-ue-cli → HTTP → UE 插件
MCP 模式：  AI → 自动发现 tools/list → MCP stdio → soft-ue-cli mcp-serve → HTTP → UE 插件
```

### 方式一：MCP 模式（推荐 ⭐）

> **AI 自动发现所有工具，无需手写 Rule。** soft-ue-cli 内置了完整的 MCP 服务器（`mcp-serve` 子命令），
> 通过 stdio 传输暴露所有命令为 MCP 工具，所有 skills 为 MCP prompts。

#### 安装

```bash
# 安装 CLI + MCP 依赖
pip install soft-ue-cli[mcp]
```

#### CodeBuddy 中配置 MCP

在 UE 项目根目录创建 `.codebuddy/mcp.json`：

```json
{
  "servers": {
    "soft-ue-cli": {
      "type": "stdio",
      "command": "soft-ue-cli",
      "args": ["mcp-serve"]
    }
  }
}
```

> 如果使用 Conda 虚拟环境，`command` 需要写 `soft-ue-cli` 的完整路径，
> 例如 `"command": "C:/Users/你的用户名/miniconda3/envs/ue-cli/Scripts/soft-ue-cli.exe"`

配置完成后，重启 CodeBuddy 或重新打开项目。AI 会自动通过 `tools/list` 发现所有 50+ 个 UE 操作工具，你只需直接说「帮我生成一个点光源」即可。

#### Cursor 中配置 MCP

在 UE 项目根目录创建 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "soft-ue-cli": {
      "command": "soft-ue-cli",
      "args": ["mcp-serve"]
    }
  }
}
```

#### Claude Code 中配置 MCP

在 UE 项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "soft-ue-cli": {
      "command": "soft-ue-cli",
      "args": ["mcp-serve"]
    }
  }
}
```

#### 验证 MCP 是否生效

配置完成后，在 AI 对话中问：「你有哪些可用的 UE 工具？」

如果 MCP 配置正确，AI 应该能列出 `spawn-actor`、`query-level`、`query-material` 等工具，**无需你事先做任何说明**。

如果 AI 说找不到工具，检查：
1. `soft-ue-cli` 命令是否在 PATH 中可用（终端中运行 `soft-ue-cli --version` 确认）
2. 是否安装了 MCP 依赖（`pip install soft-ue-cli[mcp]`）
3. `mcp.json` 文件位置是否正确（必须在项目根目录的对应配置文件夹下）
4. 编辑器是否已重启/重新加载

---

### 方式二：CLI 模式（通用方案）

> 适用于所有能执行 shell 命令的 AI 客户端和自动化脚本。
> **但 AI 默认不知道 `soft-ue-cli` 的存在，必须通过 Prompt 文件告知。**

#### ❓ 我还得记那么多 CLI 命令吗？

**不需要！你只需要写一句话告诉 AI「有这个工具」，AI 会自己跑 `--help` 去探索所有命令。**

这就是 CLI 模式的 prompt 机制——整个 `CLAUDE.md`（Claude Code 的提示词文件）核心就三行：

```
`soft-ue-cli` controls this UE project via the SoftUEBridge plugin.
Run `soft-ue-cli --help` to see all available commands.
The game or editor must be running with SoftUEBridge enabled before using UE commands.
```

AI 看到这三行后会：
1. 知道有 `soft-ue-cli` 这个工具可用
2. 主动跑 `soft-ue-cli --help` 查看完整命令列表
3. 对感兴趣的命令跑 `soft-ue-cli <command> --help` 查看详细参数
4. 根据你的需求组合命令来完成任务

**你的工作只是「告诉 AI 工具存在」这一步。** 命令学习是 AI 的事。

#### Prompt 写在哪里？

不同 AI 客户端有不同的 prompt 文件位置：

| AI 客户端 | Prompt 文件位置 | 说明 |
|-----------|----------------|------|
| **Claude Code** | `CLAUDE.md`（项目根目录） | `soft-ue-cli setup` 自动生成 |
| **CodeBuddy** | `.codebuddy/rules/ue-bridge.md` | 手动创建 |
| **Cursor** | `.cursor/rules/ue-bridge.mdc` | 手动创建 |

这些文件的作用完全一样——**在 AI 开始对话时自动注入上下文，让它知道有哪些工具可用。**

#### 为什么 CLI 模式必须写 Prompt 文件？

这是 CLI 模式与 MCP 模式的核心区别：

| | MCP 模式 | CLI 模式 |
|---|---|---|
| **工具发现** | AI 通过 `tools/list` **自动发现** | AI **不知道有这个命令**，除非 Prompt 告诉它 |
| **参数格式** | AI 从 JSON Schema **自动获取** | AI 通过跑 `--help` 自己学习 |

当你在 CLI 模式下问 AI「帮我调整场景灯光」，如果没有 Prompt 文件，AI 看到的是一个普通的 UE C++ 项目——它不知道有 `soft-ue-cli` 可用，所以会尝试直接改 C++ 代码或者说「我无法直接操作编辑器」。

#### 调用流程

```
┌───────────────────────────────────────────────────────────┐
│  AI 启动时读取 Prompt 文件（CLAUDE.md / Rule）             │
│  → 知道了 soft-ue-cli 的存在                               │
│  → 跑 soft-ue-cli --help 自学所有命令                      │
└──────────┬────────────────────────────────────────────────┘
           │
           v
┌───────────────────────────────────────────────────────────┐
│  用户："帮我在关卡中生成一个点光源"                         │
│  AI 决策：应该用 soft-ue-cli spawn-actor                   │
└──────────┬────────────────────────────────────────────────┘
           │ 执行 shell 命令
           v
┌──────────────────────────────────────────────────────────┐
│  终端                                                     │
│  $ soft-ue-cli spawn-actor PointLight --location 0,0,300 │
└──────────┬───────────────────────────────────────────────┘
           │ Python CLI → HTTP/JSON-RPC
           v
┌──────────────────────────────────────────────────────────┐
│  SoftUEBridge 插件（UE 进程内，端口 8080）                │
│  → 游戏线程执行 → 返回 JSON                               │
└──────────────────────────────────────────────────────────┘
           │
           v
┌──────────────────────────────────────────────────────────┐
│  AI 读取 JSON 输出 → 理解结果 → 决定下一步                │
└──────────────────────────────────────────────────────────┘
```

#### Claude Code 中使用（CLI 模式）

本项目已提供完整的 [`CLAUDE.md`](./CLAUDE.md) 模板文件，包含所有 60+ 命令的分类参考和工作流约定。

**使用方法：** 将 `CLAUDE.md` 复制到你的 UE 项目根目录即可。Claude Code 启动时会自动读取。

> 也可以运行 `soft-ue-cli setup`，但它只会生成极简的 3 行版本。推荐直接用本项目提供的完整模板。

#### CodeBuddy 中使用（CLI 模式）

1. 安装：`pip install soft-ue-cli`
2. 配置插件：`soft-ue-cli setup`
3. 创建 Prompt 文件 `.codebuddy/rules/ue-bridge.md`：

**最简版（够用）：**

```markdown
# Unreal Engine Bridge

本项目已安装 soft-ue-cli，可通过终端命令控制运行中的 UE 编辑器。
UE 编辑器必须在运行中且 SoftUEBridge 插件已启用。

运行 `soft-ue-cli --help` 查看所有可用命令。
运行 `soft-ue-cli <command> --help` 查看某个命令的详细参数。

所有命令输出 JSON，退出码 0 = 成功，1 = 错误。
每次视觉修改后用 `soft-ue-cli capture-screenshot viewport` 确认效果。
```

就这么短——**AI 会自己跑 `--help` 探索命令**。

**进阶版（列出常用命令，减少 AI 探索时间）：**

```markdown
# Unreal Engine Bridge

本项目已安装 soft-ue-cli，可通过终端命令控制运行中的 UE 编辑器。
UE 编辑器必须在运行中且 SoftUEBridge 插件已启用。

## 快速参考

运行 `soft-ue-cli --help` 查看完整命令列表。
运行 `soft-ue-cli <command> --help` 查看某个命令的详细参数。

## 常用命令

### Actor 操作
- `soft-ue-cli spawn-actor <class> --location x,y,z` — 生成 Actor
- `soft-ue-cli query-level` — 列出关卡中的 Actor
- `soft-ue-cli set-property <actor> <prop> --value <val>` — 设置属性

### 蓝图
- `soft-ue-cli query-blueprint <path>` — 检查蓝图结构
- `soft-ue-cli add-graph-node <path> <class>` — 添加节点
- `soft-ue-cli connect-graph-pins <path> <src> <srcPin> <dst> <dstPin>` — 连接引脚

### 材质
- `soft-ue-cli query-material <path>` — 检查材质参数和节点
- `soft-ue-cli set-asset-property <path> <prop> --value <val>` — 设置材质实例参数

### 视觉确认
- `soft-ue-cli capture-screenshot viewport` — 截图查看效果

### Skills（内置工作流）
- `soft-ue-cli skills list` — 列出所有可用 skills
- `soft-ue-cli skills get <name>` — 获取某个 skill 的完整提示词

## 工作流约定
- 所有命令输出 JSON，退出码 0 = 成功，1 = 错误
- 每次视觉修改后用 capture-screenshot 确认效果
- 不确定用什么命令时，先跑 --help
- 桥接服务器默认在 http://127.0.0.1:8080
```

#### Cursor 中使用（CLI 模式）

创建 `.cursor/rules/ue-bridge.mdc`，内容与上方 CodeBuddy Rule 类似。

---

### CLI vs MCP 如何选择？

| | CLI 模式 | MCP 模式 |
|---|---|---|
| **AI 能否自动发现工具** | ❌ 不能，必须写 Rule | ✅ 自动发现所有 50+ 工具 |
| **配置复杂度** | 中（pip install + 写 Rule） | 低（pip install + mcp.json） |
| **每次调用开销** | ~100ms（启动 Python 进程） | 持久连接，几乎为零 |
| **参数校验** | AI 靠 `--help` 文本猜 | AI 从 JSON Schema 精确获取 |
| **客户端兼容** | 所有终端工具 + CI + 脚本 | 需要 MCP 客户端支持 |
| **调试便利** | 可终端手动测试 | 需要 MCP 调试工具 |

**推荐：优先用 MCP 模式。** 配置一个 `mcp.json` 就搞定，AI 自动看到所有工具，不用写 Prompt 文件。
只有在 CI/CD 脚本或不支持 MCP 的客户端中才需要 CLI 模式。

---

## Skills（内置工作流提示词）

soft-ue-cli 自带了一套 **Skills** 系统——本质上就是**预写好的结构化 Markdown 提示词**，教 AI 如何完成特定的复杂工作流。

### Skills 是什么？

每个 Skill 是一个 `.md` 文件，包含：
- **场景描述**：什么时候用这个 skill
- **分步工作流**：AI 应该按什么顺序操作
- **命令示例**：具体用哪些 `soft-ue-cli` 命令
- **注意事项**：常见坑和最佳实践

### 内置 Skills 列表

| Skill | 用途 |
|-------|------|
| `level-from-image` | 根据参考图片（概念图/照片）自动在关卡中摆放资产 |
| `blueprint-to-cpp` | 将蓝图资产转换为 C++ 头文件和源文件 |
| `replay-changes` | 解决 `.uasset` 版本控制冲突——提取修改、回放编辑 |
| `test-tools` | 对所有 50+ 工具的集成测试套件 |
| `author-test` | 编写蓝图/功能测试 |
| `author-anim-state-test` | 编写动画状态机测试 |
| `author-bp-parity-test` | 编写蓝图和 C++ 对等性测试 |
| `author-invariant-test` | 编写不变量测试 |
| `author-regression-test` | 编写回归测试 |
| `run-test` | 运行已有的测试 |

### 怎么用 Skills？

**MCP 模式下：** Skills 自动注册为 MCP prompts，AI 可以直接发现和使用。

**CLI 模式下：**

```bash
# 列出所有可用 skills
soft-ue-cli skills list

# 获取某个 skill 的完整提示词内容
soft-ue-cli skills get level-from-image
```

AI 读取 skill 内容后，会按照其中的工作流步骤自动执行。比如告诉 AI「用 level-from-image skill 根据这张图布置场景」，AI 会：
1. 跑 `soft-ue-cli skills get level-from-image` 读取工作流
2. 分析参考图片
3. 按照 skill 中定义的步骤逐步在关卡中摆放 Actor

### 自定义 Skills

Skills 就是 Markdown 文件——你完全可以仿照内置 skill 的格式自己写。比如写一个「emissive 材质调光工作流」：

```markdown
---
name: emissive-lighting
description: 调整 emissive 材质实现自发光效果
version: 1.0.0
---

## 工作流

1. 用 query-material 查看当前材质参数
2. 确认父材质暴露了 EmissiveColor 和 EmissiveIntensity 参数
3. 用 set-asset-property 调整参数
4. 用 capture-screenshot 确认效果
5. 迭代调整直到满意

## 经验值参考
- 微弱辉光: Intensity 1-5
- 明显发光: Intensity 10-30
- 强烈光源: Intensity 50-100
```

---

## 完整命令参考

每个命令均可通过 `soft-ue-cli <command>` 使用。运行 `soft-ue-cli <command> --help` 查看详细选项。

### 设置与诊断

| 命令 | 描述 |
|------|------|
| `setup` | 将 SoftUEBridge 插件复制到 UE 项目中 |
| `check-setup` | 验证插件文件、.uproject 设置和桥接服务器可达性 |
| `status` | 健康检查——返回服务器状态 |
| `project-info` | 获取项目名称、引擎版本、目标平台和模块信息 |

### Actor 和关卡操作

| 命令 | 描述 |
|------|------|
| `spawn-actor` | 在指定位置和旋转角度生成一个 Actor |
| `query-level` | 列出当前关卡中的 Actor 及其变换信息，可按类或名称过滤 |
| `call-function` | 调用 Actor 上的任意 `BlueprintCallable` `UFUNCTION` |
| `set-property` | 按名称设置 Actor 上的 `UPROPERTY` 值 |
| `add-component` | 为已有 Actor 添加组件 |

### 蓝图检查与编辑

| 命令 | 描述 |
|------|------|
| `query-blueprint` | 检查蓝图资产——组件、变量、函数、事件分发器 |
| `query-blueprint-graph` | 检查事件图、函数图和节点连接 |
| `add-graph-node` | 向蓝图或材质图添加节点 |
| `remove-graph-node` | 从图中移除节点 |
| `connect-graph-pins` | 连接两个图节点之间的引脚 |
| `disconnect-graph-pin` | 断开特定引脚 |
| `set-node-position` | 批量设置节点位置，用于图的布局 |

### 资产管理

| 命令 | 描述 |
|------|------|
| `query-asset` | 在内容浏览器中按名称、类或路径搜索——也可检查 DataTable |
| `create-asset` | 创建新的蓝图、材质、DataTable 或其他资产类型 |
| `delete-asset` | 删除资产 |
| `set-asset-property` | 设置蓝图 CDO 或组件上的属性 |
| `get-asset-diff` | 获取资产与版本控制中的属性级差异 |
| `get-asset-preview` | 获取资产的缩略图/预览图 |
| `open-asset` | 在编辑器中打开资产 |
| `find-references` | 查找引用了指定资产的资产、变量或函数 |

### 材质检查

| 命令 | 描述 |
|------|------|
| `query-material` | 检查材质或材质实例——参数、节点、连接 |

### 类和类型检查

| 命令 | 描述 |
|------|------|
| `class-hierarchy` | 检查类的继承链——祖先、后代或两者 |

### Play-In-Editor（PIE）控制

| 命令 | 描述 |
|------|------|
| `pie-session` | 启动、停止、暂停、恢复 PIE——也可查询运行时 Actor 状态 |
| `pie-input` | 向 PIE 会话发送键盘、鼠标、手柄或 AI 移动输入 |

### 截屏和视觉捕获

| 命令 | 描述 |
|------|------|
| `capture-screenshot` | 捕获编辑器视口、PIE 窗口或特定编辑器面板的截图 |

### 日志和控制台变量

| 命令 | 描述 |
|------|------|
| `get-logs` | 读取 UE 输出日志，可按类别和文本过滤 |
| `get-console-var` | 读取控制台变量（CVar）的值 |
| `set-console-var` | 设置控制台变量值 |

### UE 内 Python 脚本

| 命令 | 描述 |
|------|------|
| `run-python-script` | 在 UE 内嵌的 Python 解释器中执行 Python 脚本 |
| `save-script` | 保存可复用的 Python 脚本到本地脚本库 |
| `list-scripts` | 列出所有已保存的 Python 脚本 |
| `delete-script` | 删除已保存的脚本 |

### StateTree 编辑

| 命令 | 描述 |
|------|------|
| `query-statetree` | 检查 StateTree 资产——状态、任务、转换 |
| `add-statetree-state` | 向 StateTree 添加状态 |
| `add-statetree-task` | 向 StateTree 状态添加任务 |
| `add-statetree-transition` | 在 StateTree 状态之间添加转换 |
| `remove-statetree-state` | 从 StateTree 移除状态 |

### Widget 蓝图检查

| 命令 | 描述 |
|------|------|
| `inspect-widget-blueprint` | 检查 UMG Widget 蓝图的层级、绑定和属性 |
| `add-widget` | 向 Widget 蓝图添加控件 |

### DataTable 编辑

| 命令 | 描述 |
|------|------|
| `add-datatable-row` | 向 DataTable 资产添加或更新行 |

### 性能分析（UE Insights）

| 命令 | 描述 |
|------|------|
| `insights-capture` | 启动或停止 UE Insights 追踪捕获 |
| `insights-list-traces` | 列出可用的追踪文件 |
| `insights-analyze` | 分析追踪文件的 CPU、GPU 或内存热点 |

### 构建与 Live Coding

| 命令 | 描述 |
|------|------|
| `build-and-relaunch` | 触发完整 C++ 重新构建，可选重新启动编辑器 |
| `trigger-live-coding` | 触发 Live Coding 编译（热重载） |

---

## 使用示例

### 在指定位置生成 Actor

```bash
soft-ue-cli spawn-actor BP_Enemy --location 100,200,50 --rotation 0,90,0
```

### 查询特定类的所有 Actor

```bash
soft-ue-cli query-level --class-filter StaticMeshActor --limit 50
```

### 调用 BlueprintCallable 函数

```bash
soft-ue-cli call-function BP_GameMode SetDifficulty --args '{"Level": 3}'
```

### 检查蓝图的组件和变量

```bash
soft-ue-cli query-blueprint /Game/Blueprints/BP_Player --include components,variables
```

### 启动 PIE 会话并发送输入

```bash
soft-ue-cli pie-session start --mode SelectedViewport
soft-ue-cli pie-input press --key SpaceBar
soft-ue-cli pie-session stop
```

### 捕获编辑器视口截图

```bash
soft-ue-cli capture-screenshot viewport --output screenshot.png
```

### 以编程方式编辑蓝图图

```bash
soft-ue-cli add-graph-node /Game/BP_Player K2Node_CallFunction \
  --properties '{"FunctionReference": {"MemberName": "PrintString"}}'
soft-ue-cli connect-graph-pins /Game/BP_Player node1 "exec" node2 "execute"
```

### 使用 UE Insights 进行性能分析

```bash
soft-ue-cli insights-capture start --channels CPU,GPU
# ... 运行你的测试场景 ...
soft-ue-cli insights-capture stop
soft-ue-cli insights-analyze latest --analysis-type cpu
```

---

## 配置

### 环境变量

| 变量 | 默认值 | 描述 |
|------|--------|------|
| `SOFT_UE_BRIDGE_URL` | *（无）* | 桥接服务器完整 URL 覆盖（如 `http://192.168.1.10:8080`） |
| `SOFT_UE_BRIDGE_PORT` | `8080` | 使用 localhost 时的端口覆盖 |
| `SOFT_UE_BRIDGE` | *（无）* | 设为 `1` 以在 `Target.cs` 中启用条件编译 |

### 服务器发现顺序

CLI 按以下优先级查找桥接服务器：

1. `--server` 命令行参数
2. `SOFT_UE_BRIDGE_URL` 环境变量
3. `SOFT_UE_BRIDGE_PORT` 环境变量（构造 `http://127.0.0.1:<port>`）
4. `.soft-ue-bridge/instance.json` 文件（从当前工作目录向上搜索——由插件启动时自动写入）
5. `http://127.0.0.1:8080`（默认回退）

### 团队条件编译

如果你只希望特定开发者编译桥接插件（避免给美术或策划带来额外开销），可以在 `Target.cs` 中使用 `SOFT_UE_BRIDGE` 环境变量：

```csharp
// MyGameEditor.Target.cs
if (Environment.GetEnvironmentVariable("SOFT_UE_BRIDGE") == "1")
{
    ExtraModuleNames.Add("SoftUEBridge");
}
```

需要桥接的开发者在环境中设置 `SOFT_UE_BRIDGE=1`，其他人正常构建即可。

---

## 兼容性

| 要求 | 支持版本 |
|------|----------|
| **Unreal Engine** | 5.3、5.4、5.5 |
| **Python** | 3.10+ |
| **平台** | Windows、macOS、Linux |
| **构建类型** | Editor、Development、Shipping（打包构建） |
| **依赖** | `httpx >= 0.27`（唯一运行时依赖） |

---

## 开发

```bash
git clone https://github.com/softdaddy-o/soft-ue-cli
cd soft-ue-cli
pip install -e .
pytest -v
```

---

## 常见问题

### soft-ue-cli 是什么？

soft-ue-cli 是一个 Python 命令行工具，用于从终端控制 Unreal Engine 5。它通过 HTTP/JSON-RPC 与运行在 UE 内部的 C++ 插件（SoftUEBridge）通信，实现 Actor 生成、蓝图编辑、材质检查、Play-In-Editor 会话、截屏、性能分析等 50+ 种操作的自动化。

### Claude Code 如何使用 soft-ue-cli 控制 Unreal Engine？

Claude Code 像开发者一样在终端中运行 soft-ue-cli 命令。通过在 UE 项目中添加一个描述可用命令的 `CLAUDE.md` 文件，Claude Code 可以自主查询关卡、生成 Actor、编辑蓝图、运行 PIE 会话并迭代你的游戏——全程无需手动操作编辑器。

### 不用 Claude Code 也能使用 soft-ue-cli 吗？

可以。soft-ue-cli 是一个标准的 Python CLI。你可以在 Shell 脚本、CI/CD 流水线、自定义自动化工具或任何能调用命令行程序的工作流中使用它。每个命令都输出结构化的 JSON，便于解析和集成。

### 支持打包构建吗？

支持。SoftUEBridge 插件在 UE 编辑器和打包构建（Development 和 Shipping 配置）中均可工作，适用于自动化测试打包游戏。

### 支持哪些 Unreal Engine 版本？

soft-ue-cli 支持 Unreal Engine 5.3、5.4 和 5.5。C++ 插件使用稳定的引擎 API，跨版本无需修改即可兼容。

### 有运行时性能影响吗？

SoftUEBridge 插件添加了一个监听单个端口的轻量级 HTTP 服务器。没有请求时开销可以忽略不计。服务器在游戏线程上处理请求以确保与 UE API 的线程安全。对于不需要桥接的生产构建，可以使用 `SOFT_UE_BRIDGE` 环境变量进行条件编译。

### 如何修改默认端口？

在启动 UE 之前设置 `SOFT_UE_BRIDGE_PORT` 环境变量，或在运行 CLI 命令时使用 `--server` 参数。默认端口为 8080。

### 可以同时运行多个 UE 实例吗？

可以。每个 UE 实例会将其端口写入项目目录的 `.soft-ue-bridge/instance.json` 文件。多个实例运行时，使用 `SOFT_UE_BRIDGE_URL` 或 `--server` 指定目标实例。

### 如何从命令行编辑蓝图？

使用 `query-blueprint-graph` 检查已有图节点，`add-graph-node` 创建新节点，`connect-graph-pins` 连线，`remove-graph-node` 删除节点。这使得完全以编程方式构建蓝图成为可能——适用于 AI 驱动的开发和自动化测试。

### soft-ue-cli 与 Unreal Engine Remote Control 有什么区别？

UE 内置的 Remote Control API 侧重于属性访问和预设工作流。soft-ue-cli 提供了专为 AI 编码代理设计的更广泛的命令集——包括蓝图图编辑、StateTree 操作、PIE 会话控制、UE Insights 性能分析、Widget 检查和资产创建——并且配置流程更简单（一次 pip 安装，一次插件拷贝）。

### CodeBuddy / Cursor 可以作为客户端使用吗？

可以。soft-ue-cli 是标准 CLI 工具，任何能执行 shell 命令的 AI 代理都可以使用——包括 CodeBuddy、Cursor、Windsurf、Claude Code 等。也支持通过 MCP 协议直连 UE 插件。详见上方「各 AI 客户端配置指南」章节。

### CLI 和 MCP 有什么区别？我该选哪个？

**推荐 MCP 模式。** 配一个 `mcp.json` 就搞定——AI 自动发现所有 50+ 工具，不用写 Rule。CLI 模式需要你手写 Rule 告诉 AI 有哪些命令可用，否则 AI 不知道 `soft-ue-cli` 的存在。CLI 模式适合 CI/CD 脚本和不支持 MCP 的工具。

---

## 许可证

MIT 许可证。详见 [LICENSE](https://github.com/softdaddy-o/soft-ue-cli/blob/main/LICENSE)。

---

## 链接

- **PyPI**: [pypi.org/project/soft-ue-cli](https://pypi.org/project/soft-ue-cli/)
- **GitHub**: [github.com/softdaddy-o/soft-ue-cli](https://github.com/softdaddy-o/soft-ue-cli)
- **CodeBuddy**: [codebuddy.ai](https://www.codebuddy.ai/)
- **Cursor**: [cursor.com](https://www.cursor.com/)
- **Claude Code**: [docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code)
