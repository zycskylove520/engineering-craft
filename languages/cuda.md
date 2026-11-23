# CUDA C++ 工程规范（GPU 并行编程）

> 适用版本：CUDA 12.x+ / C++17–C++23（官方 Programming Guide §5.3 支持 C++03/11/14/17/20/23；C++20 需 CUDA 12.x 起）/ Compute Capability 6.0+ ｜ 生态：AI 训练推理、科学计算、高性能计算 ｜ 识别信号：`.cu`/`.cuh`、`nvcc`、`__global__`/`__device__`
> 通用规范见 `core/`。CUDA 是 **C++ 的扩展**（`nvcc` 编译 C++，`__global__` kernel 也支持模板/类等 C++ 特性）：**host 端代码整体遵循 `languages/cpp.md` 的 Google 风格**；本文件写 device 端与 CUDA 特有的落地方式，以及 CUDA C++ 相对普通 C++ 的限制与约定。（注：lambda 作为 kernel 实参须为带 `__device__`/`__host__ __device__` 的扩展 lambda 且需 `--extended-lambda`，见官方 §5.3.8。）

## 1. 语言定位与适用场景
GPU 大规模数据并行（SIMT）主力；适合计算密集、高度并行、数据局部性好的问题（矩阵、卷积、归约、蒙特卡洛）。不适合分支发散多、小规模、强串行依赖的任务——搬上 GPU 反而更慢。迁移前先问：并行度够不够、数据搬运成本能否摊薄。

## 2. 项目结构与模块划分
- 文件：`.cu`（host+device 混合）、`.cuh`（device 头/模板，**此扩展名为社区/项目约定，NVIDIA 官方未强制**，模板放头文件是 C++ 语言要求）、host 纯 C++ 用 `.cpp` 遵循 `languages/cpp.md`。
- 按 kernel 域分文件（如 `kernels/attention.cu`）；host 封装与 device kernel 分离：host 侧负责内存/启动/错误检查，device 侧只做计算。
- `__host__ __device__` 双版本函数放 `.cuh`；模板 kernel 必须在头文件可见。
- 用 CMake 的 `enable_language(CUDA)` / `FindCUDAToolkit` 管理，指定 `CMAKE_CUDA_ARCHITECTURES`（目标算力）。

## 3. 命名规范
- 继承 host 端 Google 风格（见 `languages/cpp.md`）：类型 PascalCase、函数变量 snake_case、常量 k 前缀、成员尾下划线。
- 命名约定（**NVIDIA 官方 CUDA 文档不规定命名风格，以下为项目/团队约定**）：kernel 函数以 `_kernel` 后缀（`reduce_sum_kernel`）与 host 函数区分；device 辅助函数加 `__device__` 并可用 `device_` 前缀；共享内存/寄存器密集变量用 `smem_`/`reg_` 等前缀体现用途。

## 4. 类型系统与数据建模（CUDA C++）
- 这是 C++：用 C++ 类型系统——模板、`constexpr`、类/结构体、RAII、STL 风格容器（host 侧）；device 侧支持 C++11–C++23 特性（按 CUDA Toolkit 版本，`--std=c++20`/`--std=c++23`，见官方 Programming Guide §5.3）。
- 固定宽度类型（`int32_t`/`float`/`__half`/`nv_bfloat16`）；向量类型（`float4`/`int4`）用于对齐访问与带宽。
- 用 `__restrict__` 标注无别名指针；const 指针表达只读（`const float* __restrict__ in`）。
- **`__host__ __device__` 双版本函数**：同一函数在 host 与 device 都能调用（如数学工具、类型 convert），加 `__forceinline__` 促内联；注意函数体两边都能编译（不含 host-only API）。
- **模板 kernel**：`template<typename T> __global__` 支持泛型 kernel；模板与 `__host__ __device__` 函数模板放 `.cuh` 头文件；用 `if constexpr` 按类型分支。
- **STL  equivalents**：device 端没有完整 STL，用 Thrust（`thrust::device_vector`、`thrust::sort/transform/reduce`）、CUB（`cub::BlockReduce` 等 block 级原语）、libcudacxx（`cuda::atomic` 等 C++ 标准库风格设施）。
- device 端**不支持/慎用**：C++ 异常（官方 §5.3 明确不支持异常传播，用错误码/断言）、RTTI（`dynamic_cast`/`typeid`，官方新版指南未明确其 device 支持状态，按"不支持"保守处理）、虚函数（有开销且需谨慎）、`std::` 容器默认内存（device 不可用，用 Thrust/自定义 allocator）。
- 结构体在 device 使用注意对齐（`#pragma pack` 谨慎）；让非法状态不可表示：kernel 参数校验放 host 侧，device 假设输入合法。

## 5. 错误处理与异常策略
- **host 侧每个 CUDA API 调用都要查错**：用宏包装 `cudaError_t err = cudaXxx(); if (err != cudaSuccess) ...`，含 `__FILE__/__LINE__`。
- kernel 启动后查 `cudaGetLastError()`（捕获启动配置错误）与 `cudaDeviceSynchronize()`（捕获异步执行错误，调试用）。
- 释放资源用 RAII（智能指针 + 自定义 deleter 或封装类），保证异常/早退不泄漏 device 内存。
- device 代码**不能用 C++ 异常**（CUDA 平台不支持在 device 端 throw/catch，即使 host 编译器开启异常）；用返回错误码、`assert()`（device 端可用，触发则 kernel 中止并报 assert 失败）或在 host 侧校验前置条件。
- 异步错误定位困难：用 `cudaGetErrorString` 输出具体错误；用 **Compute Sanitizer**（`memcheck`/`racecheck`/`initcheck`/`synccheck`）查非法访问——`cuda-memcheck` 已被其取代、停止维护。
- 禁止静默吞错：不要忽略 `cudaError_t`；release 至少记录，debug 直接失败。

## 6. 注释与文档（整体遵循 cpp.md 的 Google 风格）
- **结构声明必须有文档**：kernel 在 `.cuh`/`.cu` 头文件处说明——功能、grid/block 配置要求、输入输出指针的形状与对齐、线程映射（哪个线程处理哪些元素）、边界处理；公开 host 封装函数按 Google 风格写用途/参数/返回/前置条件。
- **函数体逻辑**：只写"为什么"——为何这样划分 tile、`__syncthreads` 的位置为何必要、bank conflict 规避的原因、内存访问模式的选择。
- kernel  comment 里写清线程到数据的映射公式（如 `idx = blockIdx.x * blockDim.x + threadIdx.x`），这是 device 代码最重要的契约。

## 7. 并发与异步模型
- **层次**：grid → block → warp(32线程) → thread；用 `threadIdx`/`blockIdx`/`blockDim`/`gridDim` 计算索引，处理越界（`if (idx < n)`）。
- **同步**：`__syncthreads()`（block 内屏障，所有分支都要到达否则死锁）；warp 级用 `__syncwarp()`；host-device 同步用 `cudaDeviceSynchronize()` / stream 同步。
- **Streams**：用 stream 做并发执行与数据传输重叠；`cudaStreamCreate`/`cudaMemcpyAsync`（配 pinned memory）/事件（`cudaEvent`）计时。
- **原子操作**：`atomicAdd` 等用于竞争更新（注意性能与浮点确定性）；归约优先用 warp shuffle（`__shfl_down_sync`）而非共享内存+原子。
- 避免 warp 分支发散：同一 warp 内分支路径尽量一致。

## 8. 测试
- **正确性**：先写 CPU 参考实现（ground truth），小规模输入比对 GPU 输出（逐元素/容差）；随机种子固定。
- 边界测试：非整除 block 大小、空输入、最大尺寸、对齐/非对齐指针。
- 用 **Compute Sanitizer**（`memcheck`/`racecheck`/`initcheck`/`synccheck`，官方子工具名）查非法访问、竞态与未初始化。
- 单元测试 host 封装（内存分配/参数校验）；kernel 测试通过 host wrapper 调用。
- 覆盖率在这里意义有限，重点是正确性与性能回归基准。

## 9. 依赖管理与构建
- CMake + CUDA Toolkit；`CMAKE_CUDA_ARCHITECTURES` 明确目标算力（多架构用 `;` 分隔或 `native` 仅本地）。
- 依赖库（cuBLAS/cuDNN/CUTLASS/Thrust）版本锁定，记录 CUDA/driver 最低版本。
- 编译选项：`-arch=sm_XX` 与目标匹配；`-lineinfo`（调试）、`-G`（device debug，仅调试）。

## 10. 工具链
`nvcc`（编译）、`nsys`（NVIDIA Nsight Systems CLI，系统级时间线）、`ncu`（NVIDIA Nsight Compute CLI，kernel 级分析）、Compute Sanitizer（内存/竞态检查）。**`nvprof` 与 Visual Profiler 已弃用**（官方 Profiler Guide v12.8+），`cuda-memcheck` 已被 Compute Sanitizer 取代；二者不要再列入现役工具链。`nvdisasm`/`cuobjdump`（汇编检查）。
CI：编译 + CPU 参考比对 + compute-sanitizer + 关键 kernel 性能基准（有 GPU 的 runner）。

## 11. 性能惯用法与常见陷阱
- **度量先行**：用 nsys 看时间线（kernel/传输/同步占比）、ncu 看 kernel 指标（带宽/占用率/branch efficiency）；先找瓶颈再优化。
- **内存为王**：全局内存合并访问（coalesced）——相邻线程访问相邻地址；用共享内存做 tile 复用；用向量化（`float4`）提高带宽；常量/纹理内存用于只读广播数据。
- **占用率**：register 与 shared memory 用量决定每 SM 活跃 block 数；用 `__launch_bounds__` 指导；occupancy 不是越高越好，够隐藏延迟即可。
- 常见坑：非合并访问（stride 大）、共享内存 bank conflict（`__shared__` 数组布局）、`__syncthreads()` 缺失或位置错误导致竞态/死锁、host-device 频繁同步打断流水线、小块 kernel 启动开销、pinned vs pageable 内存传输差异、原子操作争用、warp 发散、未检查的异步错误延迟暴露。
- 优化顺序：算法/并行度 → 内存访问模式 → 占用率与资源 → 指令级；避免为微优化牺牲正确性。

## 12. 代码评审检查清单
- [ ] kernel 有文档（功能/grid-block 配置/线程映射公式/边界处理）？
- [ ] 索引计算正确、越界检查有、block 大小合理？
- [ ] 每个 CUDA API 查错、kernel 后有 cudaGetLastError、无静默吞错？
- [ ] host 内存用 RAII、无泄漏、有异常安全？
- [ ] `__syncthreads()` 位置正确且所有线程可达、无 warp 发散死锁风险？
- [ ] 全局内存合并访问、共享内存无 bank conflict、无原子争用滥用？
- [ ] 有 CPU 参考比对测试、compute-sanitizer 干净、性能基准无回归？
