# Python 工程规范

> 适用版本：3.10+ ｜ 生态：AI/算法、数据、后端、脚本 ｜ 识别信号：`.py`、`pyproject.toml`、`requirements.txt`

## 1. 定位与场景
AI/算法工程主力；擅长数据处理、ML、胶水与快速迭代；不适合高性能计算密集核心（用 C/Rust 扩展）。

## 2. 项目结构与模块划分
- 包用 `__init__.py`（或 namespace package）；按领域分层，`src/` 布局优先。
- 模块职责单一，公开面用 `__all__` 明确；避免循环导入。

## 3. 命名规范
PEP 8：模块/函数/变量 snake_case，类 CapWords（即俗称 PascalCase，官方术语为 CapWords），常量 UPPER_SNAKE，私有前缀 `_`。类型注解完整。（注：CapWords 为 PEP 8 §Naming Conventions 官方术语；"PascalCase" 为通用俗称，非 PEP 原文。）

## 4. 类型系统与数据建模
- 完整类型注解（`list[str]`、`T | None`）；`@dataclass(frozen=True)` / `pydantic.BaseModel` 建模。
- 用 `Literal`/`Enum`/联合类型让非法状态不可表示；边界用 pydantic 校验一次成模型。
- 泛型与 Protocol 表达结构化契约；避免 `Any`，用 `object` + 收窄。

## 5. 错误处理与异常策略
- 自定义领域异常继承 `Exception`；`raise ... from err` 保留链；禁止裸 `except:` 和宽泛吞错。
- 用显式异常表达可恢复错误（Python 官方以异常为主要机制，见 PEP 8 §Programming Recommendations）；若项目引入 `Result` 返回类型属项目/库级约定（如 returns 库），非 Python 官方构造。边界把底层异常转成领域异常。

## 6. 注释与文档
- **结构声明必须有文档**：模块/类/公开函数用 docstring，公开 API 写清参数/返回/Raises（PEP 257 §Multi-line Docstrings 要求记录这些）。**docstring 的具体标记风格（Google/NumPy/Sphinx）是项目/社区约定，PEP 257 明确不涉及标记语法**——项目统一即可；相关工具 Sphinx/Google 风格文档生成器属第三方生态。
- **函数体逻辑**：只写"为什么"，不写"做什么"；用 `#` 注释，行内注释克制。
- 类型注解已表达的信息不在 docstring 重复。

## 7. 并发与异步模型
- `asyncio`：异步 IO 用 `async/await`；CPU 密集用 `ProcessPoolExecutor` 绕开 GIL；IO 密集用线程池。
- 取消用 `asyncio.CancelledError`；超时用 `asyncio.wait_for`；锁用 `asyncio.Lock`。
- 协程里不调用阻塞 IO；注意事件循环阻塞。

## 8. 测试
- `pytest` + fixtures；测试名 `test_<行为>`；参数化覆盖边界。
- mock 用 `unittest.mock`/`pytest-mock`，只隔离外部依赖（网络/时间/DB），不 mock 被测对象。
- coverage 看未覆盖关键路径，不追虚高数字。

## 9. 依赖管理与构建
`pyproject.toml` 统一管理；锁文件（`uv.lock`/`poetry.lock`/`requirements` 冻结）；venv 隔离；区分运行时与开发依赖。

## 10. 工具链
ruff（格式+lint）、mypy/pyright（类型）、pytest（测试）、bandit（安全）；配置写进 pyproject。

## 11. 性能惯用法与常见陷阱
- 度量先行：`cProfile`/`py-spy`/`timeit` 定位热点，不凭感觉优化。
- 坑：可变默认参数、浅拷贝、循环内字符串拼接、GIL 下的 CPU 并发、全局解释器状态。
- 向量化用 numpy/pandas，避免 Python 层逐元素循环。

## 12. 评审检查清单
- [ ] 类型注解完整、无 `Any` 滥用？
- [ ] 公开 API docstring 齐全（参数/返回/Raises）？
- [ ] 异常链保留、无静默吞错？
- [ ] 异步代码无阻塞调用、有取消与超时？
- [ ] 测试覆盖正常/边界/异常，mock 未盖被测对象？
