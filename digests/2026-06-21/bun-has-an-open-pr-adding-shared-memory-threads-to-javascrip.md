# Bun has an open PR adding shared-memory threads to JavaScriptCore

# JavaScriptCore 的共享内存线程支持（实验性，尚未可用）#249

Jarred-Sumner 希望将 151 个提交合并到主分支。

## 设计概述

JSC 共享堆线程支持的设计规格：堆服务器与每线程分配器、共享 VM 状态、TID/SW 标记与分段蝶形数组、N 个变异器下的 JIT 层级，以及 Thread/Lock/Condition/ThreadLocal API。包含 TSAN、竞态放大器和基准测试门控文档，以及 THREAD.md 中的设计概览。

## 主要变更

- **线程测试框架**：在 `--useThreads` 标志后实现 Thread/Lock/Condition/ThreadLocal 和属性上的 Atomics 操作，以 VM 的 JSLock 作为即将推出的共享堆实现的语义预言机进行序列化。包含 39 个测试用例、TSAN 无 JIT 构建目标（空闲时零数据竞态，空抑制列表）、随机化让步竞态放大器，以及带有记录基线的串行性能基准测试门控。还修复了 WTF 中 ICU 静态归档链接顺序和两个已存在的无 JIT 构建中断。

- **多线程对象模型、JIT 支持和线程 API**：具有每线程分配器和 N 线程安全点的多变异器堆、进程全局分片原子表和 StructureID 分配锁、每线程 VM-lite 执行状态、TID/共享写标记蝶形数组（带分段回退和 TTL 观察点省略）、每层级 TID/SW 检查（FTL 中使用处理器 IC 和基于纪元的 CodeBlock 回收），以及真实变异器线程（通过 Thread/Lock/Condition/ThreadLocal API 和对象属性上的 Atomics）。所有功能位于 `--useJSThreads` 标志后，保留 GIL 作为 `--useThreadGIL` 回退层。

- **门控驱动修复**：六轮针对线程测试集的门控驱动修复：每线程 CLoop 栈替代共享栈帧覆盖、LocalAllocator 和 Heap 共享模式竞态、退役 JIT 工件记账、等待者列表和条件唤醒修复、为生成线程初始化 LLInt 调用路径、慢路径中的蝶形数组模式分派，以及 `~CodeBlock` 中 JITData 泄漏前的观察点解除。测试集：81/85 通过；为运行器添加每测试超时使挂起报告为失败。

- **生成线程修复**：`trySpreadFast` 在数组上到达了仅限平面的 `butterfly()` 访问器，而该数组的蝶形数组在竞态的相同形状添加风暴下已分段；展开路径现在根据模式进行分派并回退到通用慢路径。从生成线程调用的基线编译被调用者读取了仅由 LLInt 入口初始化的每线程 JIT 状态；线程入口序列现在为所有层级具化该状态。线程测试集全绿。

- **SPEC-ungil 规格**：N 变异器执行模型 — JSLock GIL 关闭进入令牌模式、每线程微任务/任务队列（带 keepalive 生命周期）、停止世界指挥器协议（seq_cst 停止位/访问 Dekker 对）、线程拆卸状态机（在 lite 注册表锁下的 TEARDOWN/COLLECTED/DETACHED）、通过注册表条件等待的 `~VM` 完成围栏、haveABadTime class-4 停止、延迟初始化所有者重入契约、终止模型（仅限 VM 范围）。包含执行清单审计（K4/N7）、完整修订历史（含绑定附件）和展平的 18 任务实施清单。
