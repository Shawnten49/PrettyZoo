# Modifications

本 fork 在 [vran-dev/PrettyZoo](https://github.com/vran-dev/PrettyZoo)（Apache License 2.0，已归档）基础上修改，用于在 macOS 26（Apple Silicon）上本地编译、打包并正常运行 PrettyZoo 2.1.1。

## 改动清单

### 1. JavaFX 18.0.2 → 21.0.12（`app/build.gradle`）

上游固定的 JavaFX 18.0.2 在 macOS 14+ 上启动即崩溃（`-[NSWindow _setFrameCommon:display:fromServer:]`，官方在 17.0.10+/21+ 修复，18 为 EOL 无修复版）。升级到 21 LTS 最新补丁版：

```groovy
javafx {
    version = "21.0.12"
    modules = ['javafx.controls', 'javafx.fxml', 'javafx.graphics', 'javafx.base']
}
```

### 2. jlink 合并模块 requires 补齐（`app/build.gradle`）

打包使用 beryx jlink 插件（`org.beryx.jlink` 2.26.0）。在 `mergedModule {}` 中显式声明内容后，插件不再自动补全 jdeps 分析出的 `requires`，需手写全部依赖，否则运行期抛 `IllegalAccessError` / `NoClassDefFoundError`：

- JDK 模块：`java.logging`、`java.security.sasl`、`java.naming`、`java.management`、`java.prefs`、`java.rmi`、`java.sql`、`java.xml`、`java.desktop`、`java.security.jgss`、`jdk.unsupported`、`java.scripting`、`java.compiler`、`java.net.http`
- JavaFX 模块：`javafx.base`、`javafx.graphics`、`javafx.controls`、`javafx.fxml`（缺失时点“连接”加载节点视图 FXML 即抛异常，表现为一直转圈、无报错）
- 日志：`org.slf4j`

### 3. SLF4J / log4j2 服务声明（`app/build.gradle`）

jlink 后 SLF4J 找不到 provider，应用与 Curator 的日志全部丢失。重新声明服务实现：

- `provides org.slf4j.spi.SLF4JServiceProvider` → `org.apache.logging.slf4j.SLF4JServiceProvider`
- `provides`/`uses` log4j2 的 `Provider`、`ContextDataProvider`、`PropertySource`、`ThreadDumpMessage.ThreadInfoFactory`

### 4. macOS 跳过 AWT Taskbar（`PrettyZooApplication.java`）

`Taskbar.getTaskbar().setIconImage(...)` 在无图形/受限会话中会触发 AWT 原生 abort。macOS 的 Dock 图标由 bundle 内 `.icns` 提供，macOS 上直接跳过，Windows/Linux 保留原逻辑。

### 5. 连接链路诊断日志（`PrettyZooFacade.java`、`CuratorZookeeperConnectionFactory.java`）

在连接异步任务与 Curator 建连的关键步骤写入 `~/.prettyZoo/connect-debug.log`，便于排查“连接卡住”类问题。如需纯净版本，删除这两个文件中的 `debug(...)` 调用即可。

### 6. 构建工具链升级

- Gradle wrapper 7.6 → 8.14.3（8.5+ 才支持在 Java 21 上运行）；
- 使用 JDK 21 编译（`JAVA_HOME` 指向本机 Temurin 21），`sourceCompatibility` 保持 17：beryx jlink 插件 2.26.0 的模块检查器不支持 class file 65（Java 21 字节码）；内置运行时仍由 JDK 21 的 jlink 生成；
- Lombok 1.18.22 → 1.18.46（1.18.30+ 才支持 JDK 21）。

变更文件：`build.gradle`、`gradlew`、`gradlew.bat`、`gradle/wrapper/*`。

## 构建方式

```bash
# 使用本机 JDK 21（Gradle 8.14.3 已支持）
export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"

# 生成 app bundle（macOS arm64）
./gradlew app:jpackageImage --no-daemon -x test

# 生成 dmg 安装包（需在可挂载磁盘镜像的环境中执行）
./gradlew app:jpackage
```

产物位于 `app/build/jpackage/`。

## 许可证

遵循上游 Apache License 2.0，保留原 `LICENSE` 与版权声明；本仓库仅做兼容性修复，不声明为官方版本。
