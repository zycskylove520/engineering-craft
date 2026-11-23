# languages 层导航（语言规范）

**硬规则：识别到项目主语言后、写任何代码前，必读对应语言文件。** 多语言项目按本次改动涉及的语言分别读。

| 语言 | 识别信号 | 文件 | 状态 |
|---|---|---|---|
| Python | `.py`、`pyproject.toml`、`requirements.txt`、`setup.py` | `python.md` | 已填 |
| TypeScript / JavaScript | `.ts`/`.tsx`/`.js`/`.jsx`、`package.json`、`tsconfig.json` | `typescript.md` | 已填 |
| Go | `.go`、`go.mod` | `go.md` | 已填 |
| Java | `.java`、`pom.xml`、`build.gradle` | `java.md` | 已填 |
| C++ | `.cpp`/`.cc`/`.hpp`/`.h`、`CMakeLists.txt` | `cpp.md`（Google 风格） | 已填 |
| Rust | `.rs`、`Cargo.toml` | `rust.md` | 已填 |
| Bash（Linux/POSIX） | `.sh`、`#!/bin/bash`、`#!/bin/sh`、CI 中的 shell | `bash.md` | 已填 |
| PowerShell（Windows） | `.ps1`/`.psm1`/`.psd1`、`pwsh` | `powershell.md` | 已填 |
| CUDA C++（GPU） | `.cu`/`.cuh`、`nvcc`、`__global__` | `cuda.md` | 已填（是 C++ 扩展，host 端遵循 `cpp.md` 的 Google 风格） |
| 其他 | — | 复制 `_TEMPLATE.md` | 待填 |

新语言流程：复制 `_TEMPLATE.md` → 填 12 章 → 更新本表识别信号 → 不动 core。
语言文件只写"该语言特有的落地方式"；通用规范在 `core/`，语言文件引用而非复制。
