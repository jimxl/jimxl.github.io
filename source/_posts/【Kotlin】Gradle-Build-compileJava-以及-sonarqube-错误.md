---
title: 【Kotlin】Gradle Build compileJava 以及 sonarqube 错误
date: 2021-09-05 21:57:28
tags: Kotlin, Gradle
---


## 背景

最近写Kotlin代码，发现文件函数还是不如ruby方便，例如获取目录下后缀为 .kt 的文件，例如创建文件的时候，如果目录不存在就把目录先创建了，于是想Kotlin的DSL语法也是很强大的，应该能较好的实现ruby的各种特性。
所以，开始了[Kotby](https://github.com/KotlinDream/kotby)项目, 始构建了一个简单的gradle项目，添加了sonarqube插件结果在 github action里面build步骤一直报错。

![错误截图](/images/image_9.png)

于是我在本地开始debug gradle build 命令，看看错误出现在哪儿，结果本地一直给出如下错误

![错误截图](/images/image_10.png)

而且在IDEA里面无论是build还是跑测试都正常，就是命令行里面执行 ./gradlew build。检查了IDEA里面的各种设置都是Java16啊, build.gradle.kts 文件里面也是写的16。

![build.gradle.kts](/images/image_11.png)

就奇怪了，哪儿也没有制定过 14。 还想是不是 build.gradle.kts 需要设置，但是google了半天也没找到如何设置。想来想去，那就只能是命令行里面Java的版本问题了，一看果然是的。
原来gradle运行的时候，是这样的逻辑啊。马上用 sdkman 安装了16，设置成了默认。一切ok，以为都解决了结果 sonarqube 又报错.

![build.gradle.kts](/images/image_12.png)

说什么手动，自动收集都开启了，需要关闭一个。问题我从来没开过什么自动收集。最重要的看错误也不知道是 gradle的某个什么收集，还是sonarqube的某个收集。又是两个小时的搜索和研究，终于找到了问题.

![sonarcloud](/images/image_13.png)

不知道是什么时候打开了，还是创建的时候是默认打开的，总之关闭之后终于正常了。

辛苦的一天