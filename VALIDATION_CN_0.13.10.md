# kptools 0.13.10 严格 Kallsyms 验证报告

## 输入

- 原厂 `boot.img`：从 Android Boot Image v2 中提取 GZIP 内核和附加 DTB，使用只读符号分析。
- 原修补版 `new-boot.img`：同样只读分析，没有写入、刷写或修改上传镜像。
- 源码：用户上传的 `kptools-win-build.zip`。

## 变更

- 找不到精确 `kallsyms_num_syms` 时，拒绝使用近似符号数量，令当前候选失败并进入后续重试。
- ARM64 vectors 回退定位时，增加前三个符号地址一致性检查。
- 清除每轮查找的旧符号数量状态。
- 将 `mismatch relo applied` 改成说明后续必须通过精确符号数和 vectors 校验的提示。
- 版本号更新至 0.13.10。

## 验证结果

| 检查 | 结果 |
|---|---|
| Linux GCC 14.2 + CMake Release 编译 | 通过 |
| 版本输出 | `d0a`（0.13.10） |
| 第一轮近似符号计数 | 被拒绝 |
| 第二轮近似符号计数 | 被拒绝 |
| 后续精确符号计数 | `0x1a5b9`，通过 |
| `pid_vnr` CFI stub / `sp_el0` 校验 | 通过 |
| `vectors` 解析 | index 18，offset `0x2800` |
| `kallsyms_addresses` 解析 | offset `0x01c1c700` |
| 原厂内核解析出的符号总数 | 107,961 |
| 原版与新版本的完整符号列表比较 | 完全一致 |
| 原修补版内核只读解析 | 通过，符号列表与原厂一致 |

## 限制

- 这次是 Kallsyms 解析器的离线验证，不是新 KPIMG 或新 boot 镜像的启动验证。
- 当前环境完成的是 Linux 原生编译，没有完成 Windows/MSYS2 实机编译。
- 上传的 PR #208 本身并未合并进上游；本地只采用了其严格计数与前三项校验思路，并进行了针对该设备内核的只读测试。
- 不包含对 PAC/SCS、Vivo 安全模块、CFI 初始化链或 AVB 启动链的实机验证。
- 不要将此包生成的镜像直接刷入；应先在 Windows/MSYS2 构建后使用原厂内核执行 `-d` 只读验证，并继续检查 KPIMG 与设备内核兼容性。
