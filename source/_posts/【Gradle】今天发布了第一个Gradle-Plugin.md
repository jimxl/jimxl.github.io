---
title: 【Gradle】今天发布了第一个Gradle Plugin [info.dreamcoder.devtools]
date: 2021-10-06 18:49:21
tags: Gradle, Kotlin, Plugin
---

今天是值得纪念的，在之前成功的发布了 kotby代码库到官方的 maven仓库的基础上，今天有把 [devtools](https://plugins.gradle.org/plugin/info.dreamcoder.devtools) 的gradle plugin发布到了Gradle官方仓库

## 目的

这个插件就是希望在开发Gradle的项目的时候，能更自动的做一些事情，帮助我们高效开发。

目前能做的就是根据文件的变化，自动执行对应的测试。我们可以想象开发过程有如下场景:

1. 我边写了Kotlin代码，然后编写了对应的测试，想跑测试的时候。
2. 当我们修改了某个代码文件，想去跑对应的测试的时候。
3. 当我们修改了对应的测试，想跑测试的时候。

这个三种情况，原来要么在IDE里面点击，要么在命令行手动输入命令 gradle test --tests="AbcClass"
这些都会打断我们的开发和思路, 实际上最好的过程就是有一个守护程序，当发现文件变动后就自动执行对应的测试。
Rails开发中就有，[Guard](https://github.com/guard/guard) 插件能完成这个能力，开发过程非常方便。
Java体系的项目中好像一直就还没有找到对应的工具，所以我自己开发了一个。

如同 [Guard](https://github.com/guard/guard) 能完成很多功能一样，未来[devtools](https://plugins.gradle.org/plugin/info.dreamcoder.devtools)还会有更多的功能。

## 使用

1. 启用插件

```kotlin
id("info.dreamcoder.devtools") version "1.3"
```

2. 启动守护进程

```Gradle
gradle guard
```

于是，上面方便的提升测试效率的工具就有了。