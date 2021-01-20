---
title: SLF4J+logback进行日志记录
tags: Log
copyright: true
abbrlink: 26975
date: 2021-01-15 00:38:18
---

# SLF4J+logback进行日志记录

SpringBoot会默认使用logback作为日志框架，在生成springboot项目的时候可以直接勾选logback，那么就可以直接使用logback了。手动添加的话，建议使用slf4j+logback，后面项目更容易维护：

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.1.7</version>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.7</version>
</dependency>
1234567891011121314151617
```

> SLF4J 是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。大概意思是指你只需要按统一的方式写记录日志的代码，而无需关心日志是通过哪个日志系统，以什么风格输出的，因为它们取决于部署项目时绑定的日志系统。
> 例如，在项目中使用了 SLF4J 记录日志，并且绑定了 Log4j（即导入相应的依赖），则日志会以 Log4j 的风格输出；后期需要改为以 Logback 的风格输出日志，只需要将 Log4j 替换成 Logback 即可，不用修改项目中的代码。

<!--more-->

# 快速实现

假如我们需要实现这么一个需求：在文件中记录调用接口事件和传参，并在控制台显示。实现起来很简单，三步即可。

第一步，在resource目录下创建一个logback.xml文件，内部写入：

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!--日志配置-->
<configuration>
    <!--直接定义属性-->
    <property name="logFile" value="logs/mutest"/>
    <property name="maxFileSize" value="30MB"/>

    <!--控制台日志-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d [%thread] %-5level %logger{50} -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

	<!--滚动文件日志-->
    <appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logFile}.log</file>
        <encoder>
        	<!--日志输出格式-->
            <pattern>%d [%thread] %-5level -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logFile}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>${maxFileSize}</maxFileSize>
        </rollingPolicy>
    </appender>
    <!--创建一个具体的日志输出-->
    <logger name="com.mutest" level="info" additivity="true">
    	<!--可以有多个appender-ref，即将日志记录到不同的位置-->
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="fileLog"/>
    </logger>

    <!--基础的日志输出-->
    <root level="info">
    </root>
</configuration>
```

> 关于这段xml的详情，第二章节会详细讲解

第二步，在application.yml文件中配置项目要使用的日志配置文件路径：

```xml
logging:
  config: classpath:logback.xml
```

第三步，在接口添加日志记录：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
	// getLogger()的入参是当前类，否则输出日志的类名会是错误的
    private final Logger logger = LoggerFactory.getLogger(TestController.class);

    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public String logTest(String name, String age) {
        logger.info("logTest,name:{},age:{}", name, age);
        return "success";
    }
}
```

当然，如果你安装了lombok这个插件，就更简单了： [Lombok使用详解](https://blog.csdn.net/mu_wind/article/details/104844946)

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class TestController {

    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public String logTest(String name, String age) {
        log.info("logTest,name:{},age:{}", name, age);
        return "success";
    }
}
```

启动项目后调用接口，控制台输出如我们所期望：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831163917777.png)
同时，项目中增加了一个log目录，生成mutest.log文件，里面记录了日志：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020083116401172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L211X3dpbmQ=,size_16,color_FFFFFF,t_70#pic_center)
功能是实现了，但我们脑子里还是有很多小问号，不急，接下来就细细讲来。

# 配置xml

首先，在`resource`目录下创建一个文件，命名为`logback.xml`。现在先向里面写一些固定的内容，就是下面这个样子：

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!--日志配置-->
<configuration>
    <!--直接定义属性-->
    <property name="" value=""/>
    <!--通过配置文件定义属性-->
    <springProperty name="" source=""/>
    
    <!--定义并描述一个日志的输出属性-->
    <appender name="" class="">

    </appender>
    <!--创建一个具体的日志输出-->
    <logger name="" level="" additivity="">
        <appender-ref ref=""/>
    </logger>

    <!--基础的日志输出-->
    <root level="">
        <appender-ref ref=""/>
    </root>
</configuration>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831155516471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L211X3dpbmQ=,size_16,color_FFFFFF,t_70#pic_center)

## configuration

`<configuration>`是logback.xml这个xml文件的根节点，它包含以下属性：

1. `scan`：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
2. `scanPeriod`：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
3. `debug`：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

例如，下面这个configuration：

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <!-- 其他配置省略-->  
</configuration>  
```

## property和springProperty

这两个节点可以设置全局变量。

property可以直接设置，例如：

```xml
<property name="logFile" value="logs/mutest"/>
```

这样就设置了一个名为`logFile`的变量，后续通过`${logFile}`的方式就引用到了其值`logs/mutest`。

而springProperty则要配合配置文件，例如：

```xml
<springProperty name="logFile" source="log.file"/>
```

也是设置了一个名为`logFile`的变量，但没有直接赋值，而是通过source指向了配置文件的路径，配置文件中是这样的：

```xml
log:
  file: logs/mutest
```

## root

`root`节点，必选节点，用来指定最基础的日志输出级别并指定`<appender>`，可以理解为根`logger`。

一个典型的root节点如下：

```xml
<root level="debug">
	<appender-ref ref="console" />
	<appender-ref ref="file" />
</root>
```

## appender

`appender`节点是非常关键的一个节点，负责格式化一个日志输出节点（也就是描述日志存储类型、位置、滚动规则等属性）。**我个人理解，appender作用类似于构造一个日志模板，而logger是真正的日志输出者，使用某个`appender`作为模板去写日志。**

appender有三种类型，分别是`ConsoleAppender`（控制台日志）、`FileAppender`（文件日志）、`RollingFileAppender`（滚动文件日志）。

### ConsoleAppender

`ConsoleAppender`的作用是将日志输出到控制台，一般在本地调试时使用，它的配置非常简单，一个典型的`ConsoleAppender`如下：

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	<encoder>
        <pattern>%d [%thread] %-5level %logger{50} -[%file:%line]- %msg%n</pattern>
        <charset>UTF-8</charset>
    </encoder>
</appender>
```

`appender` 有name和class两个属性：

- `name`：`appender`节点的名称，在后文中被`logger`节点引用。一个logback配置文件中不能有重复的`appender name`。
- `class`：使用何种日志输出策略，分别是`ConsoleAppender`（控制台日志）、`FileAppender`（文件日志）、`RollingFileAppender`（滚动文件日志）。

### FileAppender

`FileAppender`用于把日志添加到文件。一个典型的`FileAppender`如下：

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender"> 
	<file>testFile.log</file> 
	<append>true</append> 
	<encoder> 
		<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
	</encoder>
</appender> 
```

相对于`ConsoleAppender`，它多了一些子节点，让我们一一来看：

- `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
- `<append>`：如果是 `true`，日志被追加到文件结尾，如果是 `false`，清空现存文件，默认是`true`。
- `<encoder>`：对记录事件进行格式化。（具体参数稍后讲解 ）
- `<prudent>`：如果是 `true`，日志会被安全的写入文件，即使其他的`FileAppender`也在向此文件做写入操作，效率低，默认是 `false`。
- `<pattern>`：日志的输出格式。

`pattern`定义了日志的输出格式，我们以`<pattern>%d [%thread] %-5level -[%file:%line]- %msg%n</pattern>`为例，分解开来：

1. `%date`：表示日期
2. `%thread`：表示线程名
3. `%-5level`：表示级别从左显示 5 个字符宽度
4. `%logger{50}`：表示 Logger 名字最长 50 个字符
5. `%msg`：表示日志消息
6. `%n`：换行符

### RollingFileAppender

`RollingFileAppender`用于滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。一个典型的`RollingFileAppender`节点如下：

```xml
<configuration>
	<!--直接定义属性-->
    <property name="logFile" value="logs/mutest"/>
    <property name="maxFileSize" value="30MB"/>

	<appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!--日志文件存储路径，来自property设置-->
        <file>${logFile}.log</file>
        <encoder>
            <pattern>%d [%thread] %-5level -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        	<!--每天生成一个新的活动日志文件，旧的日志归档，后缀名为2019.08.12这种格式-->
            <fileNamePattern>${logFile}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!--活动日志文件最大值，超过这个值将产生新日志文件-->
            <maxFileSize>${maxFileSize}</maxFileSize>
            <!--只保留最近30天的日志-->
        	<maxHistory>30</maxHistory>
        	<!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
        	<totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
</configuration>
```

- 另外，`RollingFileAppender`节点下有一些常用的子节点：

  - `<rollingPolicy>`：当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。

  - `<filter>`：日志输出拦截器，可以自定义拦截器也可以用系统一些定义好的拦截器。

  - `<rollingPolicy>`：当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类。
1. `SizeAndTimeBasedRollingPolicy`：根据日志文件大小和时间周期作为切分条件，满足其中任意一个就要做切分。maxFileSize的值决定了当天的日志文件大小上限，超过这个上限，同一天将会有多个日志文件，因此`<fileNamePattern>${logFile}.%d{yyyy-MM-dd}.%i</fileNamePattern>`中有一个`%i`，就是为应对同一天生成多个日志文件而写，在日志量很大的情况下，会出现`mutest.log.2020-07-28.0.log、mutest.2020-07-28.1.log`这种情况。
    2. `TimeBasedRollingPolicy`：只以时间周期为切分条件，在这种策略下，存档日志名称格式设置为`<fileNamePattern>${logFile}.%d{yyyy-MM-dd}.log</fileNamePattern>`即可。
    3. `SizeBasedTriggeringPolicy`：只以文件大小为切分条件，在这种策略下，`<maxFileSize>`日志滚动的唯一触发条件。
    
- `<fileNamePattern>`：必要节点。以`${logFile}.%d{yyyy-MM-dd}.%i.log`为例`（mutest.2019-07-28.0.log）`，有这么几个部分：
    1. `${logFile}`：固定文件名称前缀，这里是引用了`<property>`中设置的变量。
    2. `%d{yyyy-MM}`：指定日志名称中间日期的格式，如果只有`%d`，将默认使用`yyyy-MM-dd`格式。
  3. `%i`：当日志量过大，导致同一天生成两个及以上日志文件时，这个属性将为日志名称加一个索引作为后缀，以加以区分。
  4. `.log.zip`：指定存档日志文件的压缩格式。
  

还有几个属性，要根据滚动策略去添加：

  - `<maxFileSize>`：这是活动文件的大小，`SizeAndTimeBasedRollingPolicy`策略和`SizeBasedTriggeringPolicy`策略下必须有。默认值是10MB。超过这个大小，就要生成新的活动文件了。
  - `<maxHistory>`：可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且`<maxHistory>`是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删。
- `<totalSizeCap>`：可选节点，表示日志文件总大小超过1GB将删除存档日志文件。

## logger

`logger`节点，可选节点，作用是指明具体的包或类的日志输出级别，以及要使用的`<appender>`（可以把`<appender>`理解为一个日志模板）。

一个典型的`logger`节点如下：

```xml
<!-- name 属性表示匹配的logger类型前缀 -->  
<logger name="com.mutest.demo">  
    <level value="INFO" />  
    <!-- 引用的appender，类似于spring的ref -->  
    <appender-ref ref="fileLog" />  
    <appender-ref ref="STDOUT" /> 
</logger>  
```

1. `name`：必写属性，指定具体包或类，被指定的包或类中的日志输出将遵从该`logger`规定配置。
2. `level`：非必写属性，指定日志输出级别，该级别将覆盖`root`配置的输出级别。
3. `addtivity`：非必写属性，是否向上级loger传递打印信息。默认是true。
4. `appender-ref`：引用的`appender`，引用后将实现`appender`中定义的行为，例如上面示例中引用了`fileLog`这个`appender`，那么`com.mutest.demo`中打印的日志将按`fileLog`的配置进行记录。一个`logger`可以有多个引用，互不影响。

# 更多情形

## 日志级别

logback有5种级别，分别是`TRACE < DEBUG < INFO < WARN < ERROR`，定义于ch.qos.logback.classic.Level类中。

- Trace:是追踪，就是程序推进一下，你就可以写个`trace`输出，所以`trace`应该会特别多，一般不会设置到这个级别。
- `Debug`：指出细粒度信息事件对调试应用程序是非常有帮助的。
- `Info`：消息在粗粒度级别上突出强调应用程序的运行过程。
- Warn：输出警告及warn以上级别的日志。
- `Error`：输出错误信息日志.

此外OFF表示关闭全部日志，ALL表示开启全部日志。

那么，在logback中，日志级别如何设置呢？

首先，`<root>`中可以设置日志级别，如果不设置，`root logger`默认级别是`DEBUG`。

```xml
 <root level="info"></root>
```

其次，`logger`中可以设置日志级别，设置后将覆盖`<root>`的设置，不设置将继承`<root>`的日志级别

```xml
<logger name="com.mutest" level="error" additivity="false">
    <appender-ref ref="STDOUT"/>
	<appender-ref ref="fileLog"/>
</logger>
```

另外，还可以在配置文件中设置更加具体的日志级别，例如将`com.mutest.controller`包下所有的日志输出级别设置为info，那么即使`logger`中，设置为error级别，日志仍然输出。

```xml
logging:
  config: classpath:logback.xml
  level:
    com.mutest.controller: info
```

## 日志滚动

如果不设置日志滚动策略，那么会一直向一个文件中追加日志，日志文件会越来越大，想要查找有用信息会很慢，而且有占满磁盘的风险。所以，我们要设置滚动策略，即满足一定条件，生成一个新文件，而旧日志文件进行归档。

### 时间策略

以时间周期为切分条件，`<rollingPolicy>`的class要设置为`ch.qos.logback.core.rolling.TimeBasedRollingPolicy`，一个典型示例（每天生成一个日志文件，保存30天的日志文件）如下：

```xml
<configuration> 
　　<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
　　　　<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
　　　　　　<fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern> 
　　　　　　<maxHistory>30</maxHistory>
　　　　</rollingPolicy> 
　　　　<encoder> 
　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　</encoder> 
　　</appender> 
 
　　<root level="DEBUG"> 
　　　　<appender-ref ref="FILE" /> 
　　</root> 
</configuration>
```

### 文件大小策略

以文件大小为切分条件，`<rollingPolicy>`的class要设置为`ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy`，一个典型示例（活动日志文件大小超过30M则生成新的活动日志文件）如下：

```xml
<configuration> 
　　<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
　　　　<rollingPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"> 
　　　　　　<fileNamePattern>logFile.%d{yyyy-MM-dd}.%i.log</fileNamePattern>  
　　　　　　<maxFileSize>30MB</maxFileSize>
　　　　</rollingPolicy> 
　　　　<encoder> 
　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　</encoder> 
　　</appender> 
 
　　<root level="DEBUG"> 
　　　　<appender-ref ref="FILE" /> 
　　</root> 
</configuration>
```

**要注意的是，`<fileNamePattern>`中，`%i`必须要有，如果同一天产生多个归档日志文件，`%i`会产生一个后缀加以区分。例如`mutest.2019-07-28.0.log` 和 `mutest.2019-07-28.1.log`。**

### 时间与文件大小策略

根据日志文件大小和时间周期作为切分条件，满足其中任意一个就要做切分。`<rollingPolicy>`的class要设置为`ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy`，一个典型示例如下：

```xml
<configuration> 
　　<appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>mutest.log</file>
        <encoder>
            <pattern>%d [%thread] %-5level -[%file:%line]- %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logFile}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>${maxFileSize}</maxFileSize>
        </rollingPolicy>
    </appender>
 
　　<root level="DEBUG"> 
　　　　<appender-ref ref="FILE" /> 
　　</root> 
</configuration>
```

同样，`<fileNamePattern>`中必须有`%i`。

## 日志过滤

首先，了解一下，日志级别从低到高分为：

```
TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```

有时候，我们需要对日志进行过滤，logback提供了多种过滤规则的实现。

### LevelFilter

比如说，日志级别为info以上，但我们不想打印warn类型的日志，那么按照下面的配置做：

```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">
    <level>warn</level>
    <onMatch>DENY</onMatch>
    <onMismatch>ACCEPT</onMismatch>
</filter>
```

几个参数的含义：

- `ch.qos.logback.classic.filter.LevelFilter`：过滤规则。这里是根据日志级别进行匹配。
- `level`：要匹配的日志级别。
- `<onMatch>DENY</onMatch>`：匹配到的日志会被拒绝。
- `<onMismatch>ACCEPT</onMismatch>`：未匹配到的日志会被打印。

### ThresholdFilter

除了 `ch.qos.logback.classic.filter.LevelFilter`外，还有一种过滤策略：`ThresholdFilter`。即临界值过滤器，过滤掉低于指定临界值的日志。当日志级别等于或高于临界值时，过滤器返回NEUTRAL；当日志级别低于临界值时，日志会被拒绝。

比如，设置只打印info级别以上的日志：

```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>info</level>
</filter>
```

### EvaluatorFilter

EvaluatorFilter是求值过滤器，评估、鉴别日志是否符合指定条件。

```xml
<filter class="ch.qos.logback.core.filter.EvaluatorFilter">         
  <evaluator> <!-- 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator -->   
	<expression>return message.contains("success");</expression>
  </evaluator>   
  <OnMatch>ACCEPT</OnMatch>  
  <OnMismatch>DENY</OnMismatch>  
</filter>   
```

属性释义：

- `<evaluator>`： 鉴别器，常用的鉴别器是`JaninoEventEvaluato`，也是默认的鉴别器，它以任意的Java布尔值表达式作为求值条件，求值条件在配置文件解释过成功被动态编译，布尔值表达式返回true就表示符合过滤条件。evaluator有个子标签`<expression>`，用于配置求值条件。
- `<onMatch>`：用于配置符合过滤条件的操作，ACCEPT或DENY
- `<onMismatch>`：用于配置不符合过滤条件的操作，ACCEPT或DENY

## 与配置文件结合

Spring Boot项目的配置文件有 application.properties 文件和 application.yml 文件两种，而我个人比较喜欢用 yml 文件。

application.yml 文件中对日志的配置：

```javascript
logging:
  config: logback.xml
  level:
    com.example.demo.dao: trace
```

1. logging.config 用来指定项目启动的时候，读取哪个配置文件，这里指定的日志配置文件是根路径下的 logback.xml 文件。关于日志的相关配置信息，都放在了 logback.xml 文件中。
2. logging.level 用来指定具体的 Mapper 中日志的输出级别，上面的配置表示 com.example.demo.dao 包下的所有 Mapper 日志输出级别为 Trace，会将操作数据库的 SQL 打印出来。开发时设置成 trace 方便定位问题，在生产环境上，将这个日志级别再设置成 error 级别即可。
3. **常用的日志级别按照从高到低依次为：ERROR、WARN、INFO、DEBUG。**