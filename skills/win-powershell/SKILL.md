---
name: win-powershell
version: 1.0.0
display_name: Windows PowerShell
display_name_en: Win PowerShell
description: Windows command-execution guide for coding agents — always use PowerShell 7 (pwsh), never Windows PowerShell 5.1 (powershell.exe); use rg for text search and file enumeration; PowerShell-native syntax for quoting, here-strings, pipelines and exit-code checks. Trigger when generating or executing shell commands on Windows, or when the host shell is PowerShell / cmd / Git Bash / WSL and the agent might pick the wrong shell.
description_zh: Windows 命令执行指引 —— 强制 PowerShell 7（禁用 powershell.exe / cmd / Git Bash / WSL 嵌套），单双引号与 here-string 规范，文本检索优先 rg，附 PowerShell 7 与 ripgrep 安装引导
description_en: Windows PowerShell 7 command execution and rg-based search guide for coding agents
---

# Win PowerShell

Windows 上执行命令时遵循本 skill：强制 PowerShell 7、原生 PowerShell 语法、检索优先 `rg`。适用于生成或执行任何 Windows 侧 shell 命令的场景。

## 环境引导（首次使用时）

- **PowerShell 7 未安装** → 建议用户安装（winget）：

  ```powershell
  winget install --id Microsoft.PowerShell --source winget
  ```

  验证：`pwsh.exe -NoLogo -NoProfile -Command '$PSVersionTable.PSVersion.Major'` → 应输出 `7`。

- **rg（ripgrep）未安装** → 建议用户安装（先征求同意）：

  ```powershell
  winget install --id BurntSushi.ripgrep.MSVC --source winget
  ```

  验证：`rg --version`。

- 安装动作必须先征求用户同意，不擅自执行。

## Windows 与命令执行

- Windows 环境使用当前 PowerShell 7；不得调用 `powershell.exe`（当前机器会降级到 Windows PowerShell 5.1）。除非任务明确要求，不嵌套调用 `pwsh.exe`、`cmd.exe`、Git Bash、WSL 或其他 shell。
- 确需启动独立 PowerShell 7 进程时，使用 `pwsh.exe -NoLogo -NoProfile`，避免加载用户 profile 注入额外命令与配置。
- 直接使用 PowerShell 语法；避免 Unix 命令及 `cat`、`find`、`where` 等含义不明确的别名或同名程序。
- PowerShell 字符串和正则在无需变量展开时使用单引号；需要展开时使用双引号，相邻字符有歧义时写 `${name}`。双引号字符串内使用反引号而非反斜杠转义。
- 多行文本使用 PowerShell here-string，不使用 Bash heredoc（`<<EOF`）。
- 将 `foreach`、`if` 等语句块的输出接入管道时，使用 `& { ... } | ...` 包裹，不直接在 `}` 后接 `|`。
- 命令失败时先检查命令、路径、引号及退出码，不随意切换 shell。

## 检索

- 文本检索优先使用 `rg`，文件枚举优先使用 `rg --files`，按文件名查找使用 `rg --files | rg`。
- 不把含 `*` 的路径直接作为 `rg` 的 PATH 参数；文件类型筛选使用 `-g`，目录通配先用 `Get-ChildItem -Directory -Path` 展开为 `FullName` 后再传给 `rg`。
- `rg` 不可用时，使用 PowerShell 的 `Get-ChildItem` 和 `Select-String`；如需安装工具，先征求用户同意。
- `rg` 退出码 `1` 表示没有匹配结果，不视为执行错误。
