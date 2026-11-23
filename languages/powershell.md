# PowerShell 工程规范（Windows）

> 适用版本：PowerShell 7+（跨平台）/ 5.1（Windows 内置）｜ 生态：Windows 自动化、运维、CI、配置管理 ｜ 识别信号：`.ps1`/`.psm1`、`pwsh`、`PowerShell`
> 通用规范见 `core/`；本文件只写 PowerShell 特有的落地方式。

## 1. 语言定位与适用场景
Windows 系统自动化、AD/注册表/WMI/进程管理、CI 脚本主力；对象管道（传 .NET 对象而非文本）是核心优势。不适合复杂算法与大规模业务逻辑（用 C#/Python）。

## 2. 项目结构与模块划分
- 脚本 `.ps1`，模块 `.psm1` + 清单 `.psd1`；函数库做成模块，用 `Import-Module` 引入。
- 普通脚本顶部声明 `#requires -Version 7.0`（或 5.1 按目标）；模块导出面用 `Export-ModuleMember` 控制。
- 按功能分模块文件，避免单个巨型脚本；脚本参数用 `param()` 块声明。
- 命名：脚本/模块 `PascalCase` 或 `kebab-case`（项目统一），动词-名词命名函数（`Get-User`、`Set-Config`）。

## 3. 命名规范
- 函数/ cmdlet：**动词-名词**（`Verb-Noun`），动词用 approved verbs（`Get-`/`Set-`/`New-`/`Remove-`/`Test-`/`Invoke-`），用 `Get-Verb` 查。
- 参数 PascalCase（`ComputerName`）；变量 `$camelCase`；常量/脚本级配置 `$UPPER_SNAKE` 或 `$script:Var`。
- 私有函数用 `function Get-Thing { ... }` 不导出，或前缀 `_`；别名只用于交互，脚本里一律全名（别名不保证存在）。
- 布尔参数用形容词（`-Enabled`），避免 `-NoX` 双重否定。

## 4. 类型系统与数据建模
- 强类型变量声明：`[string]$name`、`[int]$count`、`[datetime]$when`；参数加 `[Parameter(Mandatory)]` 与类型。
- 用 PSCustomObject / `[pscustomobject]@{...}` 或 class（5+）建模数据，管道传对象而非字符串拼接。
- 空值用 `$null` 显式判断（`$null -eq $x`，把 `$null` 放左边避免数组比较陷阱）；需要可空值类型用 `[System.Nullable[T]]`（如 `[System.Nullable[int]]`），注意 PowerShell 无 C# 式可空引用类型 `[nullable]`。
- 让非法状态不可表示：参数用 `ValidateSet`/`ValidateRange`/`ValidatePattern` 约束，入口即校验。

## 5. 错误处理与异常策略
- 脚本顶部 `$ErrorActionPreference = 'Stop'`（把非终止错误转为终止错误）；`Set-StrictMode -Version Latest` 抓未定义变量。
- 用 `try/catch/finally` 处理终止错误；catch 里 `$_` 是 ErrorRecord，保留 `$_.Exception`。
- 区分终止/非终止错误：`-ErrorAction Stop` 显式转换；`Write-Error` 产生非终止错误，配合 `$ErrorActionPreference`。
- **禁止静默吞错**：空 catch、`-ErrorAction SilentlyContinue` 要有理由；用 `throw` 抛出有意义的错误。
- 退出码：`exit 0` 成功、非 0 失败；CI 里检查 `$LASTEXITCODE`（外部命令）与 `$?`（PowerShell 命令）。
- 资源清理用 `finally` 或 `try/finally`；COM 对象显式释放（`[System.Runtime.InteropServices.Marshal]::ReleaseComObject()` / `FinalReleaseComObject()`）。

## 6. 注释与文档
- **文件头注释**：脚本用途、参数、依赖、权限要求、副作用（改系统状态要写明）。
- **函数声明注释**：用 comment-based help（` <# .SYNOPSIS ... .DESCRIPTION ... .PARAMETER ... .EXAMPLE ... #>`），公开函数/模块函数必须有——这是结构声明的文档层，可被 `Get-Help` 读取。
- **函数体逻辑**：只写"为什么"——平台差异、非常规写法、权限坑的原因；`#` 注释克制。
- 参数用 `[Parameter(HelpMessage='...')]` 补充；模块配 about 主题文档。

## 7. 并发与异步模型
- 后台任务：`Start-ThreadJob`（快、轻量）或 `Start-Job`（独立进程）；用 `Wait-Job`/`Receive-Job` 收集结果与错误，Job 要 `Remove-Job` 清理。
- ForEach-Object -Parallel（7+）并行处理，注意 `$using:` 传变量与线程安全。
- **PowerShell 没有 C# 的 `await` 关键字、`|=` 也不是 PowerShell 运算符**（官方 issue #10043 确认无 async/await）。调用返回 Task 的 .NET 异步方法时用 `$task.GetAwaiter().GetResult()`（或 `.Wait()`/`.Result`）同步取结果；并发优先用 `Start-ThreadJob` / `ForEach-Object -Parallel`；PowerShell Job 超时用 `Wait-Job -Timeout`。
- 共享状态/文件写用互斥（Mutex/文件锁）防并发冲突；COM 对象注意线程单元（STA）。

## 8. 测试
- **Pester**（PowerShell 测试框架）：测试名描述行为，`Describe`/`It`/`BeforeAll`；用 `Should -Be` 断言。
- 用 TestDrive/TestRegistry 提供隔离的临时文件系统/注册表；mock 外部依赖用 `Mock -CommandName`，不 mock 被测函数本身。
- 测试覆盖：正常/参数缺失非法/错误分支/边界（空数组/特殊字符/权限不足）。
- CI：`Invoke-Pester` + PSScriptAnalyzer 全过。

## 9. 依赖管理与构建
- 模块依赖在 `.psd1` 的 `RequiredModules`/`NestedModules` 声明；外部模块用 `Install-Module -Scope CurrentUser` 明确范围。
- 固定模块版本（`RequiredVersion`），避免自动升级破坏脚本；内部模块放私有仓库。
- 执行策略（`Set-ExecutionPolicy`）与签名要求写进部署文档；不要靠 `-ExecutionPolicy Bypass` 长期绕过。

## 10. 工具链
PSScriptAnalyzer（静态分析，强制）、Pester（测试）、`Get-Help`（文档）、VS Code + PSScriptAnalyzer 扩展；CI 跑分析 + Pester。

## 11. 性能惯用法与常见陷阱
- 度量先行：`Measure-Command`、Stopwatch；避免管道里逐对象调用高开销 cmdlet。
- 坑：**`$null` 比较方向**（`$null -eq $arr`）、数组单元素被展开（用 `@()` 强制数组）、`-contains` vs `-in`、`+=` 拼接数组造成 O(n²)（用 `List<T>` 或收集到数组）、`$?` 与 `$LASTEXITCODE` 混用、字符串插值与变量作用域（`script:`/`global:`）、路径用 `Join-Path` 而非字符串拼、通配符 `-like` vs `-match`、大小写不敏感的默认比较。
- 大集合用流式管道或 .NET 集合，避免反复物化。

## 12. 代码评审检查清单
- [ ] `param()` 声明完整、有类型与 Validate 约束、Mandatory 明确？
- [ ] `$ErrorActionPreference='Stop'` + StrictMode、try/catch 保留错误、无空 catch/静默忽略？
- [ ] 函数动词-名词命名、公开函数有 comment-based help？
- [ ] `$null` 比较正确、数组用 `@()`、路径用 Join-Path、退出码检查？
- [ ] 权限要求与副作用写明、无 Bypass 长期绕过？
- [ ] PSScriptAnalyzer / Pester 全过？
