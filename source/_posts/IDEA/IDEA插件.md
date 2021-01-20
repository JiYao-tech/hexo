---
title: IDEA常用插件
tags:
  - IDEA
  - 插件
copyright: true
abbrlink: 19821
date: 2020-06-07 23:37:13
---

## Lombok

> Lombok为Java项目提供了非常有趣的附加功能，使用它的注解可以有效的地解决那些繁琐又重复的代码，例如 Setter、Getter、toString、equals、hashCode 以及非空判断等。

- 举个例子，我们给一个类添加@Getter和@Setter注解：

```
/**
 * 修改订单费用信息参数
 * Created by macro on 2018/10/29.
 */
@Getter
@Setter
public class OmsMoneyInfoParam {
    private Long orderId;
    private BigDecimal freightAmount;
    private BigDecimal discountAmount;
    private Integer status;
}
```

- Lombok就会为我们自动生成所有属性的Getter和Setter方法。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235614-607778.png)

## Free MyBatis Plugin

> MyBatis扩展插件，可以在Mapper接口的方法和xml实现之间自由跳转，也可以用来一键生成某些xml实现。

- 我们可以通过Mapper接口中方法左侧的箭头直接跳转到对应的xml实现中去；

<!--more-->

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235616-561532.png)

- 也可以从xml中Statement左侧的箭头直接跳转到对应的Mapper接口方法中去；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235618-875445.png)

- 还可以通过`Alt+Enter`键组合直接生成新方法的xml实现，使用起来是不是很方便！

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235620-804429.png)

## MyBatis Log Plugin

> 有时候我们需要运行过程中产生的SQL语句来帮助我们排查某些问题，这款插件可以把Mybatis输出的SQL日志还原成完整的SQL语句，就不需要我们去手动转换了。

- 首先我们需要打开这款插件的窗口；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235622-230196.png)

- 当我们调用方法，控制台输出Mybatis的SQL日志时；

```
2020-04-28 15:52:20.455 DEBUG 13960 --- [nio-8081-exec-1] c.m.m.m.UmsAdminMapper.selectByExample   : ==>  Preparing: select id, username, password, icon, email, nick_name, note, create_time, login_time, status from ums_admin WHERE ( username = ? )
2020-04-28 15:52:20.456 DEBUG 13960 --- [nio-8081-exec-1] c.m.m.m.UmsAdminMapper.selectByExample   : ==> Parameters: admin(String)
2020-04-28 15:52:20.463 DEBUG 13960 --- [nio-8081-exec-1] c.m.m.m.UmsAdminMapper.selectByExample   : <==      Total: 1
```

- 该插件会自动帮我们转换成对应的SQL语句；

```
1  2020-04-28 15:50:40.487 DEBUG 9512 --- [nio-8081-exec-9] c.m.m.m.UmsAdminMapper.selectByExample   : ==>
select id, username, password, icon, email, nick_name, note, create_time, login_time, status
 FROM ums_admin
 WHERE ( username = 'admin' );
```

- 有的时候我们需要转换的日志并不在自己的控制台上，这时可以使用插件的`SQL Text`功能：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235624-459832.png)

- 直接复制我们需要转换的日志，然后点击`Restore Sql`按钮即可。

## RestfulToolkit

> 一套Restful服务开发辅助工具集，提供了项目中的接口概览信息，可以根据URL跳转到对应的接口方法中去，内置了HTTP请求工具，对请求方法做了一些增强功能，总之功能很强大！

- 可以通过右上角的`RestServices`按钮显示项目中接口的概览信息；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235626-807594.png)

- 可以通过搜索按钮，根据URL搜索对应接口；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235627-400642.png)

- 可以通过底部的HTTP请求工具来发起接口测试请求；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235629-549088.png)

- 通过在接口方法上右键可以生成查询参数、请求参数、请求URL；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235631-301146.png)

- 通过在实体类上右键可以直接生成实体类对应的JSON；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235633-588820.png)

## Translation

> 一款翻译插件，支持Google、有道、百度翻译，对我们看源码时看注释很有帮助！

- 直接选中需要翻译的内容，点击右键即可找到翻译按钮；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235634-661172.png)

- 直接使用`翻译文档`可以将整个文档都进行翻译；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235636-123706.png)

- 还可以通过右上角的翻译按钮直接翻译指定内容。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235637-990342.png)

## GsonFormat

> 这款插件可以把JSON格式的字符串转化为实体类，当我们要根据JSON字符串来创建实体类的时候用起来很方便。

- 首先我们需要先创建一个实体类，然后在类名上右键`Generate`，之后选择`GsonFormat`；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235639-401848.png)

- 输入我们需要转换的JSON字符串：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235641-340106.png)

- 选择性更改属性名称和类型：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235643-34635.png)

- 点击确定后直接生成实体类。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235644-703945.png)

## Grep Console

> 一款帮你分析控制台日志的插件，可以对不同级别的日志进行不同颜色的高亮显示，还可以用来按关键字搜索日志内容。

- 当项目打印日志的时候，可以发现不同日志级别的日志会以不同颜色来显示；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235646-313131.png)

- 如果你需要修改配色方案的话，可以通过`Tools`打开该插件的配置菜单；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235648-390994.png)

- 然后通过配置菜单修改配色方案；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235649-553840.png)

- 可以通过在控制台右键并使用`Grep`按钮来调出日志分析的窗口：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235651-426753.png)

- 然后直接通过关键字来搜索即可。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235652-946619.png)

## Alibaba Java Coding Guidelines

> 阿里巴巴《Java 开发手册》配套插件，可以实时检测代码中不符合手册规约的地方，助你码出高效，码出质量。

- 比如说手册里有这么一条；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235654-977056.png)

- 当我们违反手册规约时，该插件会自动检测并进行提示；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235656-38211.png)

- 同时提供了一键检测所有代码规约情况和切换语言的功能；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235657-305575.png)

- 如果你想修改某条规约的检测规则的话，可以通过设置的`Editor->Inspections`进行修改。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235659-496428.png)

## Maven Helper

> 解决Maven依赖冲突的好帮手，可以快速查找项目中的依赖冲突，并予以解决！

- 我们可以通过`pom.xml`文件底部的`依赖分析`标签页查看当前项目中的所有依赖；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235700-713618.png)

- 通过`冲突`按钮我们可以筛选出所有冲突的依赖，当前项目`guava`依赖有冲突，目前使用的是`18.0`版本；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235703-347413.png)

- 选中有冲突的依赖，点击`Exclude`按钮可以直接排除该依赖；

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235705-352565.png)

- 同时`pom.xml`中也会对该依赖添加`<exclusion>`标签，是不是很方便啊！

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235707-91168.png)

## EasyCode

在这个里面找到你想生成的表，然后右键，就会出现如下所示的截面。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235708-514870.jpeg)

点击1所示的位置，选择你要将生成的代码放入哪个文件夹中，选择完以后点击OK即可。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235711-476614.jpeg)

勾选你需要生成的代码，点击OK。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235712-76835.jpeg)

这样的话就完成了代码的生成了，生成的代码如下图所示：

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235713-673134.jpeg)

### pom.xml

```
<dependency>
<groupId>
org.springframework.boot
</groupId>
<artifactId>
spring-boot-starter
</artifactId>
</dependency>

<dependency>
<groupId>
org.springframework.boot
</groupId>
<artifactId>
spring-boot-starter-web
</artifactId>
</dependency>

<dependency>
<groupId>
org.projectlombok
</groupId>
<artifactId>
lombok
</artifactId>
<optional>
true
</optional>
</dependency>

<!--热部署-->
<dependency>
<groupId>
org.springframework.boot
</groupId>
<artifactId>
spring-boot-devtools
</artifactId>
<optional>
true
</optional>
<!-- 这个需要为 true 热部署才有效 -->
</dependency>

<!--mybatis-->
<dependency>
<groupId>
org.mybatis.spring.boot
</groupId>
<artifactId>
mybatis-spring-boot-starter
</artifactId>
<version>
1.3.2
</version>
</dependency>

<!-- mysql -->
<dependency>
<groupId>
mysql
</groupId>
<artifactId>
mysql-connector-java
</artifactId>
<version>
5.1.47
</version>
</dependency>

<!--阿里巴巴连接池-->
<dependency>
<groupId>
com.alibaba
</groupId>
<artifactId>
druid
</artifactId>
<version>
1.0.9
</version>
</dependency>
```

### Application.yml

```
server:
  port: 
8089
spring:
  datasource:
    url: jdbc:mysql:
//127.0.0.1:3306/database?useUnicode=true&characterEncoding=UTF-8
    username: root
    password: 
123456
    type: com.alibaba.druid.pool.
DruidDataSource
    driver-
class
-name: com.mysql.jdbc.
Driver

mybatis:
  mapper-locations: classpath:
/mapper/
*
Dao
.xml
  typeAliasesPackage: com.vue.demo.entity
```

### 启动项目

在启动项目之前，我们需要先修改两个地方。

在dao层加上@mapper注解

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235721-106006.webp)

在启动类里面加上@MapperScan("com.vue.demo.dao")注解。

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235722-134745.jpeg)

启动项目

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235724-198067.webp)

测试一下

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235726-543647.webp)

![img](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/235728-476400.webp)

