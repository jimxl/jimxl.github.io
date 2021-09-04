---
title: 【Kotlin】Gradle Build 文件报错
date: 2021-09-05 02:07:41
tags: Kotlin, Gradle
---

## 背景

最近在用Kotlin重写 [Migration](https://github.com/JavaDream/migration) 项目，今天遇到了奇怪的现象：
写着写着项目,下面的所有 build.gradle.kts 文件Idea全部报错，说找不到 "org.gradle.kotlin.dsl.KotlinBuildScript"。但是在这些错误下，程序竟然还可以正常的跑测试和运行。
整个问题很奇怪，我什么都没改，有时候只是添加了一个依赖，就会出现整个问题。而且去掉以来之后还是不行。

期间试用了各种办法, 网上搜索比较靠谱的就是 [解决办法](https://stackoverflow.com/questions/65645510/cannot-access-script-base-class-org-gradle-kotlin-dsl-kotlinbuildscript)
里面最靠谱的方法是:

![微观管理](/images/image_5.png)

在我开始创建整个项目的时候，也遇到了这个问题，当时就是用Idea新建了项目，什么也不干就是有整个错误，最后就使用上图的办法解决的。但是这次反反复复折腾了几次还是不行。
此时内心是崩溃的，难道整个项目要换回Maven来管理？而且怎么便捷的更换过去呢？ 一堆问题的情况下，我决定还是再坚持坚持。

期间各种搜索，各种实验，结果是各种失败，就在要放弃的时候，尝试的看了一下竟然还有人这么说

![微观管理](/images/image_6.png)

抱着最后的希望试了一下，发现竟然可以了, 具体设置如下

![微观管理](/images/image_7.png)
![微观管理](/images/image_8.png)

虽然还没搞懂为啥，但是毕竟是有效果了，赶紧记下来先。

