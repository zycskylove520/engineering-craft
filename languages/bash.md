# Bash 工程规范（Linux / POSIX Shell）

> 适用版本：Bash 4+（当前官方手册 Edition 5.3）｜ 生态：构建脚本、部署、运维、CI、胶水 ｜ 识别信号：`.sh`、`shebang #!/bin/bash`、`Makefile`/CI 中的 shell
> 注：Google Shell Style Guide §1.1 明确以 Bash 为准、无需刻意追求 POSIX 兼容；关联数组等特性需 Bash 4+。
> 通用规范见 `core/`；本文件只写 Shell 特有的落地方式。Shell glue 常是破坏性操作高发区，安全条款要格外严格执行。

## 1. 语言定位与适用场景
擅长串接命令、文件/文本处理、自动化运维与胶水；不适合复杂数据结构、业务逻辑、错误处理密集的系统——超过 ~100 行或需要复杂控制流时，改用 Python/Go。

## 2. 项目结构与模块划分
- 脚本放 `scripts/` / `bin/`；可复用片段拆成 `lib/*.sh`，用 `source` 引入（先检查存在）。
- 可执行脚本首行统一用 `#!/bin/bash`（配最少 flags），shell 选项用 `set` 设置；**不要默认用 `#!/usr/bin/env bash`**（Google §1.1 明确要求 `#!/bin/bash`，且无需追求 POSIX 可移植）。库文件用 `.sh` 且不可执行（§2.1）。
- 用 `shellcheck` 可解析的结构；避免把大段逻辑塞进一行。
- 命名：脚本/库文件名用**小写加下划线**（`deploy_service.sh`、`common.sh`），**不用连字符 kebab-case**（Google §7.4 明确反例 `not make-template`）；入口与库职责分离。

## 3. 命名规范
- 变量/函数名 `snake_case` 小写；环境变量/常量 `UPPER_SNAKE`（`MAX_RETRY`）。
- 只读常量用 `readonly`；函数名用动词（`check_deps`、`deploy_service`）。
- 局部变量一律 `local`（函数内），避免污染全局。
- 私有辅助函数前缀 `_`；临时变量命名明确，不用 `a`/`b`/`tmp2`。

## 4. 类型系统与数据建模
- Shell 无类型：用约定表达——字符串是默认，数组用 `arr=(...)`, 关联数组 `declare -A`（Bash 4+）。
- 区分"未设置/空"：用 `${var:-default}`（未设或空取默认）、`${var-default}`（仅未设取默认）、`${var:?err}`（必须设置否则报错）。
- 用参数展开代替外部命令（`${var#prefix}`、`${var%suffix}`、`${var/old/new}`），更快更安全。
- 让非法状态不可表示：关键变量进入时校验，失败即退出。

## 5. 错误处理与异常策略
- **每个脚本开头**：`set -euo pipefail`（遇错退出/未定义变量报错/管道错误传播）；按需注释例外。
- 显式检查关键命令返回值；`if ! cmd; then ...` 处理预期失败，不要靠 `set -e` 覆盖全部。
- 管道里用 `|| true` 要写明理由；`set -e` 在 if/elif/while/until 条件、被 `!` 取反的命令、以及 `&&`/`||` 链中**非最后一条**命令处不触发——**链中最后一条仍会触发**（Bash Manual §4.3.1）；这些场景需手动检查。
- 错误信息输出到 stderr（`echo "err: ..." >&2`）；退出码有意义（0 成功，非 0 分类错误）。
- 用 `trap 'cleanup' EXIT` 做资源清理（临时文件/锁）；trap 里保留原退出码。
- 禁止静默吞错：`cmd >/dev/null 2>&1 || true` 只允许在明确知道可失败且无副作用的场景。

## 6. 注释与文档
- **文件头注释**：脚本用途、用法（参数）、依赖、作者/维护者、副作用说明。
- **函数声明注释**：结构声明层——每个函数说明用途、参数（`$1`/`$2` 含义）、返回/输出、前置条件。
- **函数体逻辑**：只写"为什么"——非常规写法、坑、平台差异的原因；`#` 注释克制，命令本身能看懂的不写。
- 用法信息用 `usage()` 函数 + `--help` 提供；长脚本配 README。

## 7. 并发与异步模型
- 需要并发用 `&` + `wait`；收集退出码用 `wait $pid` 逐个检查。
- 临时文件用 `mktemp -d`，退出时 trap 清理；避免固定路径导致并发冲突。
- 锁用 `flock`（文件锁）防并发实例竞争；cron 任务尤其要加锁。
- 后台任务的 stdout/stderr 显式重定向，避免悬挂；信号用 `trap` 处理（TERM/INT）。

## 8. 测试
- 用 **ShellCheck** 做静态检查（必过）；`bash -n` 检查语法。
- 测试框架：`bats-core`（Bash Automated Testing System）；测试用临时目录隔离，断言退出码与输出。
- 测试覆盖：正常路径、参数缺失/非法、命令失败、边界（空输入/特殊字符/空格路径）。
- 用 `set -euo pipefail` 在测试中确保失败可见。

## 9. 依赖管理与构建
- 外部命令依赖在脚本开头显式检查（`command -v foo >/dev/null || { echo ...; exit 1; }`）。
- 固定依赖版本/路径的假设写进注释；避免依赖 GNU 独有选项却在 macOS/BusyBox 跑（用 portable 写法或检测）。
- 做成 Makefile / CI step 的统一入口，避免散落的临时命令。

## 10. 工具链
shellcheck（静态检查，强制）、`bash -n`（语法）、shfmt（格式化）、bats（测试）、`shellspec`（可选）。
CI：shellcheck + `bash -n` + bats 全过。

## 11. 性能惯用法与常见陷阱
- 度量先行：`time`/`hyperfine` 测脚本耗时；避免循环里反复启动外部命令（用内建或批量处理）。
- 坑：**未加引号的变量导致分词与通配符展开**（一律 `"$var"`）、空格/换行路径、管道子 shell 变量丢失、`for` 遍历 `ls` 输出、`echo` 的 `-e`/反斜杠差异（用 `printf`）、`set -e` 的例外场景、`[[ ]]` vs `[ ]`、正则与 glob 混淆、命令替换吞掉尾随换行。
- 大文件用 `while read` 流式处理，不用 `cat | grep | awk` 堆叠。

## 12. 代码评审检查清单
- [ ] 开头 `set -euo pipefail`、有 shebang、有文件头用途/用法注释？
- [ ] 变量全部引号、用 `local`、未定义变量不裸用？
- [ ] 错误处理显式、退出码有意义、`|| true` 有理由、trap 清理？
- [ ] 不吞错、外部依赖有检查、错误信息到 stderr？
- [ ] 函数有声明注释、逻辑注释只写为什么？
- [ ] shellcheck / `bash -n` / bats 全过？
