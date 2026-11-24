# languages 层导航（语言规范 · 约定式）

**硬规则：识别到项目主语言后、写任何代码前，必读对应语言文件。** 多语言项目按本次改动涉及的语言分别读。

**读取协议**：本目录每个 `.md`（除 `_TEMPLATE.md`）对应一门语言，**文件名即语言名**（`python.md` / `typescript.md` / `go.md` / `java.md` / `cpp.md` / `rust.md` / `bash.md` / `powershell.md` / `cuda.md`）。识别到主语言后直接读同名文件。**新增语言只新建文件，不改本导航。**

识别信号（帮助判断主语言，非登记清单）：`.py`→Python；`.ts/.js`→TypeScript；`.go`→Go；`.java`→Java；`.cpp/.h`→C++；`.rs`→Rust；`.sh`→Bash；`.ps1`→PowerShell；`.cu`→CUDA（CUDA 是 C++ 扩展，host 端遵循 `cpp.md`）。

新语言流程：复制 `_TEMPLATE.md` → 填 12 章 → **不改本导航**（文件名自描述即可发现）。语言文件只写"该语言特有的落地方式"；通用规范在 `core/`，语言文件引用而非复制。
