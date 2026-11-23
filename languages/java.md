# Java 工程规范

> 适用版本：Java 17+（LTS）｜ 生态：企业后端、Android、大数据 ｜ 识别信号：`.java`、`pom.xml`、`build.gradle`
> 通用规范见 `core/`；本文件只写 Java 特有的落地方式。

## 1. 语言定位与适用场景
强类型、生态成熟、企业级后端与大型系统主力；适合复杂领域建模、事务系统。启动与内存开销相对大，轻量场景可考虑 Go/Node。

## 2. 项目结构与模块划分
- Maven/Gradle 标准布局（`src/main/java`、`src/test/java`）；按领域/限界上下文分包。
- 用 module 系统（JPMS）或构建工具的多模块管理大项目；包边界用 package-info.java 说明。
- 依赖方向单向；领域层不依赖框架细节（DDD 分层 / 六边形架构）。

## 3. 命名规范
- 类/接口：PascalCase；接口不加 `I` 前缀；实现类可加 `Impl` 后缀。
- 方法/变量：camelCase；常量：`UPPER_SNAKE_CASE`；泛型类型参数单大写字母（`T`、`E`、`K`、`V`）。
- 包名全小写、域名反转（`com.company.project`）。
- 测试类 `XxxTest`；避免缩写歧义。

## 4. 类型系统与数据建模
- 优先不可变：字段 `final`、用 record（Java 16+）表达值对象；用 Optional 表达"可能为空"的返回值（不用于字段/参数）。
- 用 enum 表达有限状态、sealed class（Java 17+）表达受限继承，让非法状态不可表示。
- 校验集中边界（DTO/控制器），领域对象保证自身不变量。
- 避免 null：用注解（`@Nullable`/`@NotNull`）明确契约，或用 Optional。

## 5. 错误处理与异常策略
- 受检异常（checked）用于调用方可恢复的错误，非受检（RuntimeException）用于编程错误/不可恢复。
- 自定义领域异常继承 `Exception`/`RuntimeException`，保留 cause；禁止吞异常（空 catch）。
- 边界统一异常处理（`@ControllerAdvice` 等），错误结构一致（码/消息/上下文）。
- 用 try-with-resources 管理资源；不要在 finally 里吞掉原始异常。

## 6. 注释与文档
- **结构声明必须有 Javadoc**：公开类/接口/方法写 `/** */`，含 `@param`/`@return`/`@throws`/`@since`；公开 API 的契约（含线程安全条件）写清。
- 包级文档用 `package-info.java`。
- **函数体逻辑**：只写"为什么"；`//` 注释克制，代码能看懂的不写。
- 用 `javadoc` 生成 API 文档；复杂算法写实现说明文档而非堆注释。

## 7. 并发与异步模型
- 线程池用 `ExecutorService`/`ThreadPoolExecutor`，不用 `new Thread` 裸奔；明确队列/拒绝策略/命名。
- 共享状态用 `synchronized`/`Lock`/`AtomicX`/`ConcurrentHashMap`；理解 JMM（可见性、happens-before）。
- 取消用 `Future.cancel`/`interrupt`，超时用 `Future.get(timeout)`/`CompletableFuture.orTimeout`。
- 虚拟线程（Java 21+，JEP 444）适合高 IO 并发、**一律不要池化**；pinning 按版本区分：JDK 21–23 需注意 `synchronized` 内的 pinning，**JDK 24+（JEP 491）已基本消除**，仅 native/FFM 等场景仍需留意。
- 避免死锁：锁顺序一致、缩小锁范围、用 tryLock 超时。

## 8. 测试
- JUnit 5 + AssertJ/Mockito；测试名表达行为（`@DisplayName` 中文亦可）。
- `@BeforeEach` 建固件；Mockito 只 mock 外部依赖，不 mock 被测对象。
- 测试分层：单元（快）/集成（Spring Boot Test、Testcontainers）/端到端。
- JaCoCo 看覆盖率，重点覆盖新增逻辑；测试隔离、不依赖顺序。

## 9. 依赖管理与构建
- Maven（`pom.xml`）或 Gradle（`build.gradle.kts`）；依赖版本集中管理（BOM/dependencyManagement）；锁定版本、避免 SNAPSHOT 进入生产。
- 多模块用 parent/aggregate；构建目标统一（compile/test/package/verify）。
- 评估依赖许可证与安全漏洞（OWASP Dependency-Check）。

## 10. 工具链
Checkstyle/Spotless（格式）、SpotBugs/PMD（静态分析）、 JaCoCo（覆盖率）、Maven/Gradle 统一入口；CI 强制 check + test。

## 11. 性能惯用法与常见陷阱
- 度量先行：JFR（Flight Recorder）、async-profiler、JMH（微基准，别用循环计时充数）。
- 坑：字符串 `+` 在循环中的开销、装箱拆箱、伪共享、集合初始容量、GC 压力（短命对象/大对象）、线程池配置不当、内存泄漏（静态集合持有引用）。
- 优先算法/数据结构优化，再谈 JVM 调优；避免过早调优。

## 12. 代码评审检查清单
- [ ] 公开 API Javadoc 齐全（param/return/throws/线程安全）？
- [ ] 不可变性优先（final/record/Optional 合理）、null 契约明确？
- [ ] 异常有上下文、cause 保留、无空 catch、资源 try-with-resources？
- [ ] 线程池规范、取消/超时明确、无竞态死锁？
- [ ] 测试覆盖正常/边界/异常、mock 未盖被测对象？
- [ ] 格式/静态分析/覆盖率检查全过？
