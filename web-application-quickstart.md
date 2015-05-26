# 快速开始 Web 应用

本章介绍了 Gradle 对 Web 应用的相关支持。 Gradle 为 Web 开发提供了两个主要插件,War 插件 和 Jetty 插件。 其中 War 插件继承自 Java 插件,可以用来生成 WAR 文件。Jetty 插件 继承自 War 插件 作为工程部署的容器。

## 构建 WAR 文件

应用 War 插件 来构建 WAR 文件：

```
build.gradle

	apply plugin: 'war'
```

>注意，完整的项目源码见[https://github.com/waylau/Gradle-2-User-Guide-Demos](https://github.com/waylau/Gradle-2-User-Guide-Demos) 中 webApplication/quickstart

同时应用 Java 插件,当你执行 `gradle build` 时,将会编译、测试、打包工程成为一个 WAR 文件。 Gradle 会在 WAR 中 src/main/webapp 下寻找 源文件。编译后的classes文件以及运行时依赖也都会被包含在 WAR  包中，分别在 WEB-INF/classes 和 WEB-INF/lib 目录下。

## 运行应用

需要应用 Jetty 插件来运行应用。

```
build.gradle

	apply plugin: 'jetty'
```

同样需要应用 WAR 插件，当你执行 `gradle jettyRun` 时,将会运行应用在一个内嵌的 Jetty Web 容器里。运行 `gradle jettyRunWar`将会构建成 WAR 文件，接着运行在内嵌 的 Web 容器。

TODO:url,端口,以及源文件位置都可以在脚本中进行指定修改并重载。

*Groovy web 应用*

*在一个项目中你可以采用多个插件。比如你可以在 web 项目中同时使用War 插件和 Groovy 插件来构建基于 web 应用的 Groovy。适当的 Groovy 库将被添加到 WAR 的文件中。*

## 总结

你可以在[https://github.com/waylau/Gradle-2-User-Guide-Demos](https://github.com/waylau/Gradle-2-User-Guide-Demos) 中 webApplication 下找到更多示例.