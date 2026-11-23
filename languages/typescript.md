# TypeScript / JavaScript 工程规范

> 适用版本：TS 5.x+（项目基线；官方当前已发布 7.0，本规范以 5.x 稳定特性为准）/ Node 20+ ｜ 生态：前端、Node 后端、工具链 ｜ 识别信号：`.ts`/`.tsx`/`.js`、`package.json`、`tsconfig.json`

## 1. 定位与场景
前端与 Node 全栈；强类型优先 TS；适合 UI、API 服务、CLI、工具库。

## 2. 项目结构与模块划分
- ESM 优先；按特性/领域分目录；入口与公开面集中在 `index.ts`。
- 用 barrel 导出控制公开 API；避免深层相对路径（配路径别名）。

## 3. 命名规范
文件/模块 kebab-case 或 PascalCase（组件）；类/类型 PascalCase；函数/变量 camelCase；常量 UPPER_SNAKE；接口不加 `I` 前缀（**TypeScript 官方无强制命名标准，以上均为社区/项目约定**——原文"项目统一即可"即为约定属性说明）。

## 4. 类型系统与数据建模
- `strict: true`；禁用 `any`，用 `unknown` + 收窄；判别联合（discriminated union）表达状态机。
- `zod`/`io-ts` 在边界校验一次成类型；`readonly`/`as const` 表达不可变。
- `never` 做穷尽检查（default 分支）。

## 5. 错误处理与异常策略
- 区分 throw（异常）与返回 `Result`/联合类型（可恢复错误）；边界把底层错误转成领域错误。
- Promise 拒绝必须处理（`catch` 或 `await` 包裹）；禁止空 catch。
- 自定义 Error 子类保留 `cause`。

## 6. 注释与文档
- **结构声明必须有文档**：导出模块/类/公开函数/类型用 TSDoc（`/** */`）；公开 API 写清 `@param`/`@returns`/`@throws`；生成文档用 TypeDoc。
- **函数体逻辑**：只写"为什么"；`//` 行内注释克制，代码能看懂的不写。
- 类型已表达的信息不在注释重复。

## 7. 并发与异步模型
- 单线程事件循环 + Promise/async-await；CPU 密集用 Worker Threads 或子进程。
- 超时用 `AbortController`；并发限制用 `p-limit` 等；注意未处理的 Promise rejection。
- 共享状态用原子操作/锁库，避免竞态。

## 8. 测试
- Vitest / Jest；测试名 `describe/it` 描述行为；MSW 隔离网络。
- mock 只隔离外部依赖，不 mock 被测对象；测试隔离，不依赖顺序。
- 覆盖率看关键路径，不追虚高。

## 9. 依赖管理与构建
npm/pnpm/yarn 锁文件提交；monorepo 用 workspace；构建目标（build/typecheck/test/lint）在 package.json scripts 统一。

## 10. 工具链
TypeScript（`tsc --noEmit`）、ESLint、Prettier、Vitest；配置统一，提交前跑一遍。

## 11. 性能惯用法与常见陷阱
- 度量先行：Chrome DevTools / Node `--prof` / `clinic.js`。
- 坑：闭包与循环引用、事件循环阻塞、`this` 绑定、隐式类型转换、依赖体积膨胀。
- 大列表用虚拟滚动/分页，避免不必要渲染与重复计算（memo 有度）。

## 12. 评审检查清单
- [ ] `strict` 开启、无 `any`、判别联合表达状态？
- [ ] 导出 API TSDoc 齐全？
- [ ] Promise rejection 全处理、无空 catch？
- [ ] 异步有超时/取消、无竞态？
- [ ] 测试覆盖关键路径、mock 未盖被测对象？
