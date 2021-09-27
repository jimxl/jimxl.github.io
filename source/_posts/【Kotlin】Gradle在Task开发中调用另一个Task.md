---
title: 【Kotlin】Gradle在Task开发中调用另一个Task
date: 2021-09-27 21:02:08
tags:
---

## 背景

原来Rails开发的时候，经常使用Guard的插件，好处非常多，其中一个好处就是当修改了代码的时候，能制动的跑对应的测试。然后输入回车的时候跑全部的测试。
最近在开发Migration的Kotlin项目的时候，由于现在已经不习惯通过手动试验来验证代码的正确性，完全是靠测试来保证。每次写完代码，都需要手动在Idea里面点运行测试，感觉不是很方便。
于是想，Java体系中是否有对应的插件，找了一会没发现有对应的。想了想，原理也很简单，就是监听文件变化，然后找到对应的测试文件运行，于是马上动手撸一个。

最近在开发的项目中，有一个叫Kotby的项目. 本来是想把ruby标准库中的一些方便的函数引入到kotlin里面，现在想想可以扩充到把ruby体系中的东西都引入进Kotlin的体系。


## 监控文件变化

第一步要实现，对文件的监控，Kotlin可以通过java的库，用下面的方式来完成。需要注意的是，Java的这个函数默认只会监控目录下直接的文件变化，并不会监控子目录的变化。所以需要自己遍历来实现。

```Kotlin
import mu.KotlinLogging
import java.io.File
import java.nio.file.*
import java.nio.file.StandardWatchEventKinds.*

class FileWatcher(private val watchPath: String) {

    private val logger  = KotlinLogging.logger {}
    private val watcher = FileSystems.getDefault().newWatchService()

    private var fileCreateAction: ((String) -> Unit)? = null
    private var fileUpdateAction: ((String) -> Unit)? = null
    private var fileDeleteAction: ((String) -> Unit)? = null

    fun onFileCreate(action: (String) -> Unit) {
        fileCreateAction = action
    }

    fun onFileModify(action: (String) -> Unit) {
        fileUpdateAction = action
    }

    fun onFileDelete(action: (String) -> Unit) {
        fileDeleteAction = action
    }


    private fun registerWatcher(dir: String) {
        logger.info{ "Start watch path [$dir]" }
        Paths.get(dir)
             .toAbsolutePath()
             .register(watcher, ENTRY_CREATE, ENTRY_MODIFY, ENTRY_DELETE)
    }

    private fun registerAllSubDir(dir: String) {
        File(dir).walk().forEach {
            if (it.isDirectory) {
                registerWatcher(it.path)
            }
        }
    }

    private fun registerSelfAndAllSubDir() {
        registerWatcher(watchPath)
        registerAllSubDir(watchPath)
    }

    private fun handleFileEvent(event: WatchEvent<*>, parentPath: File) {
        val kind = event.kind()
        val file = parentPath.resolve(event.context().toString())

        logger.info { """
            *********************************
            event:          [${event.kind()}]
            event.context() [${event.context()}]
            parentPath:     [$parentPath]
            filePath:       [${file.absolutePath}]
            isFile?:        [${file.isFile}]
            *********************************
        """.trimIndent() }

        if(file.isFile) {
            when(kind) {
                ENTRY_CREATE -> fileCreateAction?.invoke(file.absolutePath)
                ENTRY_MODIFY -> fileUpdateAction?.invoke(file.absolutePath)
                ENTRY_DELETE -> fileDeleteAction?.invoke(file.absolutePath)
            }
        }
    }

    private fun processWatch() {
        while(true) {
            val key = watcher.take()
            val dir = File(key.watchable().toString())
            key.pollEvents().forEach { handleFileEvent(it, dir) }
            key.reset()
        }
    }

    fun create() {
        registerSelfAndAllSubDir()
        processWatch()
    }

}
```

## 创建Guard任务

这一步，就是用Gradle标准的task开发方式创建一个任务，需要完成的就是启动文件监控，并在文件变化的回调中启动测试

## 启动测试

最开始就简单的在文件变化之后，运行命令行命令 "gradle test", 但是由于是在Guard任务中直接运行的"gradle test"命令，所以输出的并不是自己想要的，会有多个Gradle Task的输出。
而且，这样的调用方式也不优雅，心里觉得一定有更好的方式，可以直接通过函数调用运行测试。于是各做搜索之后，找到了如下的办法

```kotlin
val connection = GradleConnector.newConnector().forProjectDirectory(project.projectDir).connect()
connection.newBuild().forTasks("test").setStandardOutput(System.out).run()
```

利用了GradleConnector的能力，然后运行里面的test任务，最后完美得到了想要的结果。