# C++ 工程规范（Google C++ Style Guide）

> 适用版本：C++20（Google 官方现行基线，target C++20、不用 C++23 特性；来源 google.github.io/styleguide/cppguide.html#C++_Version）｜ 生态：系统与高性能、游戏、基础设施 ｜ 识别信号：`.cc`/`.cpp`/`.h`/`.hpp`、`CMakeLists.txt`
> 通用规范见 `core/`；本文件按 **Google C++ Style Guide** 落地，风格冲突时以本文件为准。

## 1. 语言定位与适用场景
高性能、系统级、资源可控；适合引擎、基础设施、嵌入式、性能敏感核心。复杂度高、开发效率低于托管语言，避免用它写业务逻辑胶水。

## 2. 项目结构与模块划分
- 头文件（`.h`）与实现（`.cc`）分离；目录按模块/组件划分，一个模块一个目标（CMake target）。
- **`#include` 顺序**（Google）：相关头文件 → C 系统 → C++ 标准 → 其他库 → 项目内头文件，各组内按字母序。
- 头文件必须用 `#define` guard（格式 `<PROJECT>_<PATH>_<FILE>_H_`，基于源码树全路径），**不要用 `#pragma once`**（Google 官方明确禁止：cppguide.html#The__define_Guard / #Windows_Code）；尽量自包含（include 自身依赖），减少前置声明滥用。
- 内部符号放 `.cc` 匿名命名空间；公开面控制在头文件。

## 3. 命名规范（Google 风格）
- 类型（类/struct/enum/type alias）：`PascalCase`（`MyClass`）。
- **普通函数用 mixed case / UpperCamelCase**（首字母大写、每词首字母大写，如 `AddTableEntry()`）；**只有 accessor/mutator 可与变量同名用 snake_case**（如 `int count()`、`void set_count(int)`）。变量/局部用 `snake_case`（官方：cppguide.html#Function_Names）。
- 常量：`kConstantName`（`kMaxSize`）；宏：`MACRO_NAME`（尽量少用宏）。
- 成员变量：`foo_`（尾部下划线）；静态成员：`foo_`（同样）；结构体字段可不加。
- 命名空间：全小写、`snake_case`，不用 `using namespace` 污染全局（Header 中禁止）。
- 文件名全小写 + 下划线；枚举值用 `kEnumValue` 风格。

## 4. 类型系统与数据建模
- 优先值语义与 RAII；用 `std::unique_ptr`/`std::shared_ptr` 表达所有权（明确、少用 shared）。
- 用 `std::optional` 表达可选、`std::variant` 表达受限联合、`enum class` 表达强类型枚举（避免隐式转换）。
- const 正确：不变的参数/方法标 `const`；用 `constexpr`/`consteval` 表达编译期常量。
- 让非法状态不可表示：构造即不变量、private 构造 + 工厂、禁用拷贝/移动（`= delete`）按需。
- 避免裸 new/delete，用标准容器与智能指针；避免隐式转换（`explicit` 单参构造）。

## 5. 错误处理与异常策略
- Google 风格**不使用异常**（官方 #Exceptions，历史/兼容性考量）：用错误码或预期类型显式处理。**`absl::Status`/`StatusOr<T>` 是 Abseil 库实践，非 Google C++ Style Guide 规定**；项目可用等价的预期类型。
- 若项目允许异常：只抛不吞、catch by reference、按值抛；禁止空 catch；资源用 RAII 保证异常安全。
- 边界统一错误转换；不要在析构函数里抛异常。
- 断言：`DCHECK`（调试）/ `CHECK`（致命）按语义使用，不用于业务错误处理。**这些是 glog 日志库宏，非 Google C++ Style Guide 正文内容**，项目用其他断言库时按等价语义处理。

## 6. 注释与文档（Google 风格）
- **结构声明必须有文档**：公开类/函数/模板在头文件写说明——用途、参数、返回、前置条件、线程安全、所有权语义。注释语法按官方 #Comment_Style 用 `//` 或 `/* */` 保持一致即可；**Doxygen 的 `/** */` 或 `///` 标记及文档生成器是项目/社区约定，非 Google 官方要求**，项目统一即可。
- 文件头注释说明文件用途；版权/许可按项目约定。
- **函数体逻辑**：只写"为什么"——意图、权衡、算法出处；`//` 注释克制，代码能看懂的不写。
- 不写与代码重复的注释；TODO 格式统一（`// TODO(user): ...`）。

## 7. 并发与异步模型
- 用 `std::thread`/`std::jthread`（C++20）、`std::async`；同步用 `std::mutex`/`std::atomic`/条件变量。
- RAII 管理锁（`std::lock_guard`/`std::unique_lock`），避免手动 lock/unlock 遗漏。
- 理解内存序（memory_order）：默认 `seq_cst` 安全，优化时须有证据；避免数据竞争（UB）。
- 明确线程安全契约（写进类注释）；避免死锁（锁顺序、scoped_lock）；取消/超时用条件变量谓词或 future wait_for。

## 8. 测试
- GoogleTest + GoogleMock；测试名 `TestSuite.TestName`，表达行为。
- 用 fixture 管理公共 setup；mock 只隔离外部依赖，不 mock 被测对象。
- 测试分级：单元（快）/集成；`ctest`/构建系统集成；覆盖率用 gcov/lcov/llvm-cov。
- 开启 sanitizer（ASan/UBSan/TSan）跑测试，抓内存/未定义行为/竞态。

## 9. 依赖管理与构建
- CMake 为主：目标导向（`add_library`/`add_executable`），显式声明依赖与可见性（PUBLIC/PRIVATE）。
- 第三方依赖用包管理（vcpkg/Conan）或 FetchContent；版本锁定；减少依赖面。
- 构建类型区分 Debug/Release；统一编译选项（警告当错误 `-Werror` 按项目约定）。

## 10. 工具链
clang-format（Google 风格配置，强制）、clang-tidy（静态分析）、include-what-you-use、sanitizers、perf/VTune（性能）。
CI 跑格式检查 + 编译 + 测试 + sanitizer。

## 11. 性能惯用法与常见陷阱
- 度量先行：perf、VTune、benchmark（Google Benchmark），别凭直觉优化。
- 坑：未定义行为（越界、悬垂引用、数据竞争）、对象切片、隐式拷贝开销、虚调用与异常处理开销、false sharing、头文件膨胀导致编译慢。
- 优先算法与数据结构（复杂度），再谈微优化；移动语义与 `emplace` 有度；内联靠编译器判断。

## 12. 代码评审检查清单
- [ ] 命名符合 Google 风格（类 Pascal/函数变量 snake/常量 k/成员尾下划线）？
- [ ] #include 顺序与自包含正确、无 `using namespace` 污染？
- [ ] 所有权清晰（智能指针/RAII）、const 正确、无裸 new/delete？
- [ ] 错误用 Status/StatusOr 显式处理、无异常滥用/空 catch？
- [ ] 公开类/函数有文档（用途/参数/返回/前置条件/线程安全）？
- [ ] 线程安全契约明确、无数据竞争、锁用 RAII？
- [ ] clang-format/tidy、测试、sanitizer 全过？
