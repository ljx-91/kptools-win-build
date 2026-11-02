# 用 GitHub Actions 编译 kptools Windows 版

本仓库基于随用户提供的 `kptools-win-build-strict-0.13.10` 源码包整理。GitHub Actions 使用 Windows runner + MSYS2 UCRT64，自动编译 Windows x64 `kptools.exe`，检查版本号，并收集 UCRT64 运行时 DLL。

> 这只负责构建程序，不会上传、修补或刷写手机镜像。严格 Kallsyms 修改是本地实验性修改，不是 KernelPatch 官方发布，也没有证明适用于所有内核。

## 1. 创建 GitHub 仓库

1. 登录 GitHub，点右上角 `+` → `New repository`。
2. 仓库名可填 `kptools-win-build`。
3. 可见性建议选 `Private`，然后创建仓库。
4. 下载并解压本项目 ZIP。
5. 将解压目录里的**所有文件和文件夹**上传到仓库根目录，确保 `.github/workflows/build-windows.yml` 的路径没有多套一层目录。

仓库根目录应该类似：

```text
.github/
  workflows/
    build-windows.yml
kernel/
tools/
version
README_CN.md
README_GITHUB_CN.md
VALIDATION_CN_0.13.10.md
```

不要把个人的 `boot.img`、`new-boot.img`、`kpimg-android` 或任何 SuperKey 放进 GitHub 仓库。工作流不需要这些文件。

## 2. 启动云端编译

1. 打开仓库顶部的 `Actions`。
2. 如果 GitHub 提示启用 Actions，选择允许运行仓库工作流。
3. 左侧选 `Build kptools Windows (UCRT64)`。
4. 点 `Run workflow` → 再点绿色的 `Run workflow`。
5. 等工作流完成，打开这次运行记录。

以后每次 push 新提交或创建 Pull Request，也会自动触发编译。

## 3. 下载编译结果

打开成功的工作流运行记录，滚动到页面底部 `Artifacts` 区域，下载：

```text
kptools-win-ucrt64
```

解压后应包含：

- `kptools.exe`
- 运行所需的 UCRT64 DLL（按实际依赖收集）
- `version.txt`
- `help.txt`
- `dependencies.txt`
- `SHA256SUMS.txt`
- `README_RUNTIME.txt`

在 Windows 命令提示符中进入解压目录，运行：

```bat
kptools.exe --version
kptools.exe --help
```

版本检查期望输出：

```text
d0a
```

## 4. 本次 Workflow 做了什么

- 在 `windows-latest` 上运行。
- 使用 MSYS2 `UCRT64` 环境，而不是 MSYS POSIX GCC。
- 安装 UCRT64 GCC、CMake、Make 和 zlib。
- 编译 `tools` 目录中的源代码。
- 检查 `kptools.exe` 能启动、版本号为 `d0a`，并生成帮助文本。
- 递归收集 `/ucrt64/bin` 中被程序依赖的 DLL。
- 将结果作为 GitHub Actions Artifact 保存 14 天。

## 5. 重要限制

GitHub 上构建成功，只表示 Windows 可执行文件成功编译并通过了基础启动测试；它不表示 Kallsyms 修改已通过真实 Windows runner 的目标内核验证，也不证明生成的任何 boot 镜像可启动。第一次只下载、运行 `--version` 和 `--help` 即可。不要用新 EXE 直接刷写手机。
