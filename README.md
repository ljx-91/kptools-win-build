# kptools Windows Build（严格 Kallsyms 验证版）

版本：0.13.10（基于上传的 0.13.9 源码）

## 本次修改

1. **精确符号数检查**：当工具无法在 `kallsyms_names` 前找到可信的 `kallsyms_num_syms` 时，不再用“近似数量减 10”继续解析，而是结束当前候选并进入下一轮重试。
2. **前三个地址一致性检查**：在 ARM64 的 vectors 回退定位阶段，要求候选地址表的前三项一致。该检查针对本次提供的 vivo 4.14.141 内核特征；对不满足此特征的其他内核会倾向于拒绝解析，而不是冒险继续补丁。
3. **重定位日志更明确**：将含糊的 `mismatch relo applied` 提示改为明确说明：候选必须继续通过精确符号数和 vectors 验证。
4. **重试状态清零**：每次查找符号数前清除上一次候选的结果，避免失败重试残留旧值。

这些修改参考了 KernelPatch PR #208 的严格 Kallsyms 思路，但不是官方已合并版本，也不是对所有内核的通用兼容性承诺。

## 已完成的验证

- 使用 GCC 14 / CMake 在 Linux 环境完成源码编译。
- 使用你提供的原厂 `boot.img` 中解压出的内核运行只读 `-d` 符号分析。
- 旧代码会在前两轮采用近似符号数并尝试 vectors；严格版本应在前两轮拒绝近似值，在第三轮找到精确符号数后继续。
- 最终符号表前三项均为 `0x1000`，与严格候选检查相符。

**尚未完成 Windows 实机编译，也没有对生成镜像进行刷机验证。不要把本构建视为已验证可启动的修补镜像。**

## Windows 构建

推荐使用仓库内 `.github/workflows/build-win.yml`：将此源码提交到 GitHub 仓库后，在 Actions 中运行 `Build kptools.exe (Windows)`，下载 `kptools-win` artifact。

也可以在 MSYS2 UCRT64 环境中：

```bash
cp -f kernel/include/preset.h tools/
cd tools
cmake .
make -j2
```

## 建议的使用方式

先只对原厂内核运行符号检查（`-d`），确认日志能找到精确 `kallsyms_num_syms` 并完成 vectors 校验。若日志提示严格检查失败，应停止修补并保留原厂 `boot.img`，不要绕过检查强制生成镜像。

原厂 `boot.img` 应始终单独备份。不要在测试前覆盖它。

## 使用 GitHub Actions 云端编译

完整步骤请看 [README_GITHUB_CN.md](README_GITHUB_CN.md)。仓库自带 `.github/workflows/build-windows.yml`，使用 Windows runner 和 MSYS2 UCRT64 构建，并上传 `kptools-win-ucrt64` Artifact。
