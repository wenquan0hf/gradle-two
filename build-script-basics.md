# 构建脚本的基本

在 Gradle 中两个顶级概念：project（项目）和 task （任务）

所有 Gradle 都有一个或多个 project 构成。project 的展现取决于 Gradle 所做的工作。举例。 project 可以是一个 JAR 库 或者是 web 应用。它可以是由项目生产 JAR 组成发布的 ZIP。一个 project 不一定
代表一个东西要构建。它可能是一件要做的事，如将应用程序部署到工作台
或生产环境。如果这看起来有点模糊，现在不要担心。Gradle 基于约定的构建支持增加一个 更具体的定义的 project。

每个项目都是由一个或多个 task。一个 task 代表了一个构建生成的原子的作品。这可能是编写一些类，创建一个 JAR ，生成 Javadoc，或发布一些库。

现在，我们将看看在构建一个 project 时定义一些简单的 task 。后面的章节将介绍多个 project 和更多的 task 。

## Hello world

运行 Gradle 是使用 gradle 命令行。命令行会寻找项目的根目录下 build.gradle 的文件（有关命令行，详见 [Appendix D. Gradle Command Line 命令行](https://github.com/waylau/Gradle-2-User-Guide/blob/master/Appendix%20D.%20Gradle%20Command%20Line%20%E5%91%BD%E4%BB%A4%E8%A1%8C.md)），这个就是构建的脚本，或者严格说是构建的配置脚本。他定义了project（项目）和 task 任务）。

尝试输出，创建一个 `build.gradle` 命名的文件：

```
build.gradle

	task hello {
		doLast {
			println 'Hello world!'
		}
	}
```

命令行切换到包含 build.gradle 文件的目录，执行 `gradle -q hello`

![](images/gradle601.jpg)


输出为：

```
	> gradle -q hello
	Hello world!
```

![](images/gradle602.jpg)

这个脚本定义了一个 名字是`hello` 的 task,并且添加了动作。当运行 `gradle hello` ,Gradle 执行这个 `hello` task，接着执行里面的动作。这里的动作只是简单的包含了一些可以执行的 Groovy 代码。

看上去很像 Ant　，不错，Gradle task 是相当于 Ant 的 target，但是你将看到，他们更强大。我们使用了跟 Ant 不同的术语，因为 task 比 target 更富表现力。

不幸的是，这一术语与 Ant 有冲突，Ant 调用它的命令行，如 javac 或copy 称之为 task 。所以当我们谈论的 task ，默认说的是 Gradle task ，这是相当 Ant 的 target。如果我们谈论的 Ant 的 task （Ant 命令），我们明确地说的 Ant task。

### 命令行加中 `-q` 的作用

q 是 quiet  的简写，意思是要安静、干净的输出。如果不加 `-q` 则会输出日志。详见[日志](logging.md).下面是对比


![](images/gradle604.jpg)

## 快捷 task 定义

定义 task 可以使用快捷方式，这样更简明。

```
build.gradle

	task hello << {
		println 'Hello world!'
	}
```

再次执行，得到相同的输出。在下面的文章中，我们都会采用这种定义方式。

![](images/gradle605.jpg)

##6 构建的脚本都是代码

工具的构建脚本给你完整的 Groovy 的功能。作为开胃菜，看看这个： 

```
build.gradle

	task upper << {
		String someString = 'mY_nAmE'
		println "Original: " + someString
		println "Upper case: " + someString.toUpperCase()
	}
```

执行 `gradle -q upper` 输出

```
	> gradle -q upper
	Original: mY_nAmE
	Upper case: MY_NAME
```

![](images/gradle606.jpg)

或者

```
build.gradle

	task count << {
		4.times { print "$it " }
	}
```

执行 `gradle -q count` 输出

```
	> gradle -q count
	0 1 2 3
```
![](images/gradle607.jpg)

## 依赖

可以声明 task 与 其他 task  的依赖

```
build.gradle

	task hello << {
		println 'Hello world!'
	}
	task intro(dependsOn: hello) << {
		println "I'm Gradle"
	}
```

执行 `gradle -q intro` 输出

```
	> gradle -q intro
	Hello world!
	I'm Gradle
```

![](images/gradle608.jpg)

添加一个依赖，相应的 task 不需要存在

```
build.gradle

	task taskX(dependsOn: 'taskY') << {
		println 'taskX'
	}
	task taskY << {
		println 'taskY'
	}
```

执行 `gradle -q taskX`输出

```
	> gradle -q taskX
	taskY
	taskX
```

![](images/gradle609.jpg)

taskX 的依赖 taskY 是在 taskY 定义之前 声明的。这个在多 project 构建时很重要。关于 task  的依赖详见 [更多关于任务](more-about-tasks.md)

请注意不要使用快捷符号，当引用的 task 还没有定义的情况下。

## 动态 task

Groovy 的能力不仅仅是定义一个 task。例如，你也可以用它来动态创建的 task。

```
build.gradle

	4.times { counter ->
		task "task$counter" << {
			println "I'm task number $counter"
		}
	}
```

执行 `gradle -q task1` 输出

```
	> gradle -q task1
	I'm task number 1
```

![](images/gradle610.jpg)

## 利用现有的任务

一旦 task 创建，他们可以通过一个 API 访问。例如，在运行时您可以使用此动态添加依赖到 task 。Ant 不允许这样的事情。

```
build.gradle

	4.times { counter ->
		task "task$counter" << {
			println "I'm task number $counter"
		}
	}
	task0.dependsOn task2, task3
```

执行 `gradle -q task0` 输出

```
	> gradle -q task0
	I'm task number 2
	I'm task number 3
	I'm task number 0
```

或者 可以添加行为到一个已经存在 task 中

```
build.gradle

	task hello << {
		println 'Hello Earth'
	}
	hello.doFirst {
		println 'Hello Venus'
	}
	hello.doLast {
		println 'Hello Mars'
	}
	hello << {
		println 'Hello Jupiter'
	}
```

执行 `gradle -q hello` 输出

```
	> gradle -q hello
	Hello Venus
	Hello Earth
	Hello Mars
	Hello Jupiter
```
![](images/gradle611.jpg)

doFirst 和 doLast 可以多次执行调用。他们在开始或结束的 task 动作清单中添加动作。task 执行时，按动作列表的顺序执行的动作。操作符 `<<` 仅仅是 doLast 的别名。

## 快捷符号

在前面的示例中已经注意到，有一个方便的符号访问现有的
task 。每个 task 可以作为构建脚本的一个属性：

```
build.gradle

	task hello << {
		println 'Hello world!'
	}
	hello.doLast {
		println "Greetings from the $hello.name task."
	}
```

执行 `gradle -q hello`输出

```
	> gradle -q hello
	Hello world!
	Greetings from the hello task.
```

![](images/gradle612.jpg)

这使得代码可读性增强，尤其是当使用的插件提供的 task ，如 compile task

## 额外 task 属性

可以添加自己属性到 task ,添加 `myProperty`属性，设置 、`ext.myProperty` 初始值,从这一点上，该属性可以读取和设置就像一个预定义的任务属性。

```
build.gradle

	task myTask {
	    ext.myProperty = "myValue"
	}
	
	task printTaskProperties << {
	    println myTask.myProperty
	}
```

执行 `gradle -q printTaskProperties`输出

```
	> gradle -q printTaskProperties
	myValue
```

![](images/gradle613.jpg)

task 不对额外属性做限制，更多详见[编写构建脚本](writing-build-scripts.md) 

## 使用 Ant task

Ant task 是 Gradle 一等公民。 Gradle 给 Ant task 提供了不错的整合通过简单依靠于 Gradle 。Groovy 被奇异的 AntBuilder 装载。从 Gradle 使用 Ant task 比使用 build.xml 文件更方便和更强大。从下面的例子中，你可以学习如何执行 Ant task 和如何访问 Ant 属性：

```
build.gradle

	task loadfile << {
	    def files = file('antLoadfileResources').listFiles().sort()
	    files.each { File file ->
	        if (file.isFile()) {
	            ant.loadfile(srcFile: file, property: file.name)
	            println " *** $file.name ***"
	            println "${ant.properties[file.name]}"
	        }
	    }
	}
```

执行 `gradle -q loadfile`输出

```
	> gradle -q loadfile
	*** agile.manifesto.txt ***
	Individuals and interactions over processes and tools
	Working software over comprehensive documentation
	Customer collaboration  over contract negotiation
	Responding to change over following a plan
	 *** gradle.manifesto.txt ***
	Make the impossible possible, make the possible easy and make the easy elegant.
	(inspired by Moshe Feldenkrais)
```

![](images/gradle614.jpg)

更多关于 构建脚本中使用 Ant ,详见 [从 Gradle 使用 Ant](using-ant-from-gradle.md)

## 使用方法

Gradle 延伸取决你如何组织的建造逻辑。上面的例子中的第一级别的组织你的构建逻辑，是提取方法。

```
build.gradle

	task checksum << {
	    fileList('../antLoadfileResources').each {File file ->
	        ant.checksum(file: file, property: "cs_$file.name")
	        println "$file.name Checksum: ${ant.properties["cs_$file.name"]}"
	    }
	}
	
	task loadfile << {
	    fileList('../antLoadfileResources').each {File file ->
	        ant.loadfile(srcFile: file, property: file.name)
	        println "I'm fond of $file.name"
	    }
	}
	
	File[] fileList(String dir) {
	    file(dir).listFiles({file -> file.isFile() } as FileFilter).sort()
	}
```

执行 `gradle -q loadfile`输出

```
	> gradle -q loadfile
	I'm fond of agile.manifesto.txt
	I'm fond of gradle.manifesto.txt
```

![](images/gradle615.jpg)

以后你会发现这样的方法可以在多 project 构建的子 project 之间共享。如果你建立逻辑变得越来越复杂，Gradle 为您提供其他工具很方便的方式来组织它。

## 默认 task

Gradle 允许你定义一个或多个默认 task 给你的构建


```
build.gradle
	
	defaultTasks 'clean', 'run'
	
	task clean << {
	    println 'Default Cleaning!'
	}
	
	task run << {
	    println 'Default Running!'
	}
	
	task other << {
	    println "I'm not a default task!"
	}
```

执行 `gradle -q` 输出

```	
	> gradle -q
	Default Cleaning!
	Default Running!
```
![](images/gradle617.jpg)

这个等于执行了 `gradle clean run` ，在多 project 中构建所有的子 project 都可以有自己具体的默认 task 。如果 子 project 没有明确的默认 task，则执行父 project 的默认 task（如果定义的话）

## 通过 DAG 配置

以后会详细描述，Gradle 有配置阶段和执行阶段。配置阶段后，Gradle 知道所有的 task 应该执行。Gradle 提供给你一个钩子来利用这些信息。这个用例将检查发布 的 task 是否是要执行的 task。基于此，你可以赋予不同的值到一些变量。

在下面的例子中，在不同 version 变量中的 distribution 和 release task 执行结果不同。

```
build.gradle

	task distribution << {
	    println "We build the zip with version=$version"
	}
	
	task release(dependsOn: 'distribution') << {
	    println 'We release now'
	}
	
	gradle.taskGraph.whenReady {taskGraph ->
	    if (taskGraph.hasTask(release)) {
	        version = '1.0'
	    } else {
	        version = '1.0-SNAPSHOT'
	    }
	}
```

执行 `gradle -q distribution` 输出

```
	> gradle -q distribution
	We build the zip with version=1.0-SNAPSHOT
```

执行 `gradle -q release` 输出

```	
	> gradle -q release
	We build the zip with version=1.0
	We release now
```

![](images/gradle618.jpg)

`whenReady `影响了 release task 在 release task 被执行之前。同样适用于 release task 不是 主 task 的情况（比如，task 被 gradle 命令通过了）

## 下一步工作

本章，我们大概浏览了下 task ,但这不是 task 的全部，可以详见[更多关于任务](more-about-tasks.md)

另外，继续教程 [快速开始 Java](java-quickstart.md) 和 [依赖管理的基础知识.md](dependency-management-basics.md)









