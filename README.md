# PrettyZoo（macOS 本地构建版）

PrettyZoo 是一款基于 JavaFX 与 Apache Curator 的 ZooKeeper 图形化管理工具。本仓库是 [vran-dev/PrettyZoo](https://github.com/vran-dev/PrettyZoo)（Apache License 2.0，已归档）的 fork，在 2.1.1 版本基础上做了 macOS（Apple Silicon）兼容性修复与构建工具链升级，可直接在本机编译、打包并运行。

## 功能特性

- 多 ZooKeeper 服务器管理
- 节点实时同步，支持创建 / 搜索 / 更新 / 删除
- ACL 支持、SSH 隧道
- 配置导入 / 导出
- 内置终端操作
- JSON / XML / Properties 数据格式化与高亮
- 断线自动重连

## 与上游的差异

全部改动见 [MODIFICATIONS.md](./MODIFICATIONS.md)，主要包括：

- JavaFX 18.0.2 → 21.0.12（修复 macOS 14+ 启动即崩溃）
- jlink 合并模块显式声明全部所需模块（修复“连接一直转圈、无任何报错”的问题）
- 恢复 SLF4J / log4j2 日志输出（打包后日志不再丢失）
- macOS 跳过 AWT Taskbar 调用（避免受限会话中原生 abort）
- 构建工具链：Gradle 8.14.3 + JDK 21 编译（字节码目标 17），Lombok 1.18.46
- 新增连接链路诊断日志（`~/.prettyZoo/connect-debug.log`）
- 首页 GitHub 图标指向本 fork、移除赞赏图标；检查更新接口与页面同样指向本 fork
- 配置页顶部显示标题（新增配置 / Alias name）并带分割线

## 系统要求

- macOS（Apple Silicon / arm64 已验证）
- 无需安装 Java：应用自带 JDK 21 运行时

## 安装与运行

1. 获取 `PrettyZoo-2.1.1-mac-arm64.zip`（见 Releases 或自行构建）；
2. 解压后把 `PrettyZoo.app` 拖入「应用程序」，或直接双击运行；
3. 若 macOS 提示无法验证开发者 / 已损坏：

   ```bash
   xattr -cr "/Applications/PrettyZoo.app"
   ```

   然后右键（或按住 Control 点击）→ 打开。

### 数据与日志位置

```text
~/.prettyZoo/prettyZoo.cfg             # 服务器连接配置
~/.prettyZoo/log/prettyZoo.log         # 应用与 Curator 日志
~/.prettyZoo/connect-debug.log         # 连接链路诊断日志
```

## 从源码构建

环境要求：JDK 21（Temurin 21.0.12 已验证）、可访问网络（首次构建需下载依赖）。

```bash
git clone https://github.com/Shawnten49/PrettyZoo.git
cd PrettyZoo

export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"

# 生成 app bundle：app/build/jpackage/prettyZoo.app
./gradlew app:jpackageImage --no-daemon -x test

# 生成 dmg 安装包（需在可挂载磁盘镜像的环境中执行）
./gradlew app:jpackage
```

字节码目标保持 17 是为了兼容 beryx jlink 打包插件 2.26.0（不影响使用 JDK 21 编译，内置运行时仍由 JDK 21 的 jlink 生成），详见 [MODIFICATIONS.md](./MODIFICATIONS.md)。

## 常见问题

**连接 ZooKeeper 一直转圈、无任何报错**

该问题已在本 fork 修复，根因是 jlink 合并模块缺少 `javafx.*` / `java.*` 模块声明，导致连接前加载节点视图 FXML 时抛 `IllegalAccessError`。若仍出现异常，请查看：

```text
~/.prettyZoo/connect-debug.log
~/.prettyZoo/log/prettyZoo.log
```

**应用打不开 / 提示已损坏**

执行安装章节中的 `xattr -cr` 命令后重新打开。

## 许可证

Apache License 2.0。本仓库保留上游 `LICENSE` 与版权声明，全部改动见 [MODIFICATIONS.md](./MODIFICATIONS.md)。本项目为社区维护版本，并非官方发布。
