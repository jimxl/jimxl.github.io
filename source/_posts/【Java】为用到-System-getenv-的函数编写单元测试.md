---
title: 【Java】为用到 System.getenv 的函数编写单元测试
date: 2021-08-22 12:09:39
tags: Java
---

## 背景

最近在编写Java的[Migration](https://github.com/JavaDream/migration)项目, 数据库相关的配置有一种默认方式是从环境变量里面获取, 代码如下:

```java
package info.dreamcoder.config;

public class EnvConfig implements Config {
    private String dbUrl = System.getenv("MYSQL_URL");
    private String dbUsername = System.getenv("MYSQL_USERNAME");
    private String dbPassword = System.getenv("MYSQL_PASSWORD");

    @Override
    public String getDbUrl() {
        return dbUrl;
    }

    @Override
    public String getDbUserName() {
        return dbUsername;
    }

    @Override
    public String getPassword() {
        return dbPassword;
    }
}
```

然后为了测试他的时候，就想先要手动指定环境变量，然后再测试这个类读出来的对不对。原理很简单，但是java实现起来就是很麻烦。java就没有提供设置环境变量的办法，这一点非常蛋疼。想了各种办法，折腾了一天之后最终还是使用了如下的方案, 里面使用到了 [systemlambda](https://github.com/stefanbirkner/system-lambda)包终于解决了

## 解决办法

```java
package info.dreamcoder.config;


import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static com.github.stefanbirkner.systemlambda.SystemLambda.*;


class EnvConfigTest {
    private static Config config;

    @BeforeAll
    public static void createConfig() throws Exception {
        config = withEnvironmentVariable("MYSQL_URL", "MYSQL_URL_test")
                .and("MYSQL_USERNAME", "MYSQL_USERNAME_test")
                .and("MYSQL_PASSWORD", "MYSQL_PASSWORD_test")
                .execute(Factory::getConfig);
    }

    @Test
    void getDbUrl() throws Exception {
        assertThat(config.getDbUrl()).isEqualTo("MYSQL_URL_test");
    }

    @Test
    void getDbUserName() throws Exception {
        assertThat(config.getDbUserName()).isEqualTo("MYSQL_USERNAME_test");
    }

    @Test
    void getPassword() throws Exception {
        assertThat(config.getPassword()).isEqualTo("MYSQL_PASSWORD_test");
    }
}
```

## 遇到的问题1: opens java.util 错误

maven配置里面， maven-surefire-plugin还需要添加如下的参数, 否则java16会给出 module java.base does not "opens java.util" to unnamed module @xxx 的[错误](https://github.com/stefanbirkner/system-lambda/issues/23),原因是上面的withEnvironmentVariable 会要反射修改 java.util 里面的内容，需要明确声明出来，这儿为了找到如何给JVM加上参数，也搜索了半天，知道了用argLine的方法是可以的。不过坑爹的是这个参数还只能一行如果多个就只能认到的一个...网上也有很多人跟我有类似的问题，例如携程如下的

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.1</version>
    <configuration>
    <argLine>@{argLine}</argLine>
    <argLine>--illegal-access=deny</argLine>
    <argLine>--add-opens java.base/java.util=ALL-UNNAMED</argLine>
    </configuration>
</plugin>
```

要写在一个里面才行，下面展示的是正确的内容。

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.1</version>
    <configuration>
    <argLine>
        @{argLine}
        --illegal-access=deny
        --add-opens java.base/java.util=ALL-UNNAMED
    </argLine>
    </configuration>
</plugin>
```

## 遇到的问题2: jacoco监测不到代码覆盖率

这里面 [@{argLine}](https://dzone.com/articles/reporting-code-coverage-using-maven-and-jacoco-plu) 也非常关键，如果没有这一行，jacoco的代码覆盖率就会得不到，搜索了文章，应该是说jacoco会注入这个参数，但由于我们手动指定了，导致jacoco注入的参数失效，最终得不到覆盖率。 

期间还行是否可以通过mock的方式, 在mockito-inline中有一个mock静态代码的方法


## 遇到的问题3: 通过mock的方式优雅，但是走不通

```java
package info.dreamcoder.config;


import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.mockito.MockedStatic;
import org.mockito.Mockito;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;


class EnvConfigTest {
    private static Config config;

    @BeforeAll
    public static void createConfig() {
        MockEnvironment env = new MockEnvironment();
        env.setProperty("MYSQL_URL", "MYSQL_URL_test");
        MockedStatic<System> theMock = Mockito.mockStatic(System.class);
        theMock.when(() -> System.getenv("MYSQL_URL")).thenReturn("MYSQL_URL_test");
        theMock.when(() -> System.getenv("MYSQL_USERNAME")).thenReturn("MYSQL_USERNAME_test");
        theMock.when(() -> System.getenv("MYSQL_PASSWORD")).thenReturn("MYSQL_PASSWORD_test");
        config = Factory.getConfig();
    }

    @Test
    void getDbUrl() {
        assertThat(config.getDbUrl()).isEqualTo("MYSQL_URL_test");

    }

    @Test
    void getDbUserName() {
        assertThat(config.getDbUserName()).isEqualTo("MYSQL_USERNAME_test");
    }

    @Test
    void getPassword() {
        assertThat(config.getPassword()).isEqualTo("MYSQL_PASSWORD_test");
    }
}
```

这样语法是没有问题的, 实现的也很优雅，可惜 mockito 库不允许mock System这个class，在运行的时候会抛出错误。


通过周末两天的折腾，终于搞定了环境变量的单元测试，很有成就感。