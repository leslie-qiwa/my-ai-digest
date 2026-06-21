# Linux eliminates the strncpy API after six years of work, 360 patches

# Linux 历经六年、360 余个补丁，终于移除 strncpy API

Linux 7.2 终于从 Linux 内核中移除了 strncpy API。strncpy() 函数用于复制指定字节数的数据，长期以来一直被标记为弃用，经过六年的工作和数百个补丁，内核中已不再有 strncpy 接口的使用者，该接口现已被正式移除。

Linux 内核中的 strncpy 函数多年来一直是"持续产生 bug 的来源"，原因在于其围绕 NUL 终止的反直觉语义和行为，以及对目标缓冲区进行冗余零填充所带来的性能问题。在过去六年中，开发者通过约 362 次提交逐步消除了内核中 strncpy 代码的使用，如今终于在 Linux 7.2 中到达了终点线。

周五的这次合并移除了 strncpy API 以及最后的各 CPU 架构 strncpy 实现。

取代 strncpy，Linux 内核代码应使用：strscpy() 用于 NUL 终止的目标缓冲区，strscpy_pad() 用于带零填充的 NUL 终止目标缓冲区，strtomem_pad() 用于非 NUL 终止的固定宽度字段，memcpy_and_pad() 用于带显式填充的有界复制，或 memcpy() 用于已知长度的内存复制。
