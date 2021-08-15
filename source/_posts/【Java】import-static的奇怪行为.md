---
title: 【Java】import static的奇怪行为
date: 2021-08-15 16:12:52
tags: Java
---

## 背景

最近在编写Java的[Migration](https://github.com/JavaDream/migration)项目，想使用Assertj来作为测试编写的工具。在编写如下测试的时候，给人的感觉会比直接用Junit来得好. 例如:

1. Junit

```java
assertFalse(database.execute("show tables 123"));
```

2. Assertj

```java
assertThat(database.execute("show tables 123")).isFalse();
```

按照官网的例子，很容易写出如下赏心悦目的断言:

```java
assertThat("The Lord of the Rings").isNotNull()
                                   .startsWith("The")
                                   .contains("Lord")
                                   .endsWith("Rings");
```

## 测试数据库

因为[Migration](https://github.com/JavaDream/migration)是一个数据库操作相关的库，所以需要对数据库里面的内容进行判断, 例如我想判断表是否创建成功， 在Junit里面是这么写的:

```java
ResultSet rs = database.query("show tables like '%test_table%'");
assertDoesNotThrow(rs::next);
assertDoesNotThrow(() -> {
   String tableName = rs.getString(1);
   assertEquals("test_table", tableName);
});
```
这段代码是从系统的tables表里面查询出来，然后判断结果是否存在。看起来不是很优雅。在看Assertj文档的时候发现他也有数据库测试相关的例子:

```java
Table table = new Table(dataSource, "members");

// Check column "name" values
assertThat(table).column("name")
        .value().isEqualTo("Hewson")
        .value().isEqualTo("Evans")
        .value().isEqualTo("Clayton")
        .value().isEqualTo("Mullen");

// Check row at index 1 (the second row) values
assertThat(table).row(1)
        .value().isEqualTo(2)
        .value().isEqualTo("Evans")
        .value().isEqualTo("David Howell")
        .value().isEqualTo("The Edge")
        .value().isEqualTo(DateValue.of(1961, 8, 8))
        .value().isEqualTo(1.77);
```
看起来舒服多了，而且很简洁，于是我也想修改我上面的代码使用这种方式来编写测试, 于是改成了:

```java
Request request = new Request(source, "show tables like '%test_table%'");
assertThat(request).row(0)
       .value().isEqualTo("test_table");
```

奇怪，竟然报错了，说assertThat没有row方法。但是看官方文档就是这么写的。最后跟文档仔细对比发现原来是引入的包不对。
前面为了使用 assertThat 我引入了 import static org.assertj.core.api.Assertions.* , 以为已经有了assertThat方法。
后来才发现原来数据库相关的需要引入 import static org.assertj.db.api.Assertions.* 最开始Idea自动给我导入的是 import static org.assertj.db.api.Assertions.assertThat 结果导致原来的测试都出现了语法错误，原因是两个 assertThat 冲突了。

## 怎么办？

google了半天也没人说，于是各种尝试，首先改成了(Idea的自动帮助下)

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatNoException;
import static org.assertj.db.api.Assertions.assertThat;
```

于是代码

```java
assertThat(database.execute("show tables")).isTrue();

Request request = new Request(source, "show tables like '%test_table%'");

assertThat(request).row(0)
       .value().isEqualTo("test_table");
```

可以同时使用，不冲突了.

换成下面的样子，同样可以:

```java
import static org.assertj.core.api.Assertions.*;
import static org.assertj.db.api.Assertions.*;
```

但是下面这样就不行:

```java
import static org.assertj.core.api.Assertions.*;
import static org.assertj.db.api.Assertions.assertThat;
```

很是奇怪，不太了解他的冲突解决机制。看网上文章有讲导入的优先级，但是也解释不通上面的原因。

继续写代码，以后搞清楚了，再来分享。 