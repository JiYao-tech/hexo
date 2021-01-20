---
title: Spring AOP的使用
tags: Spring AOP
copyright: true
abbrlink: 22699
date: 2021-01-15 00:29:35
---
# Spring AOP的使用

# 理解AOP

## 什么是AOP

AOP（Aspect Oriented Programming），面向切面思想，是Spring的三大核心思想之一（两外两个：IOC-控制反转、DI-依赖注入）。

那么AOP为何那么重要呢？

在我们的程序中，经常存在一些系统性的需求，比如权限校验、日志记录、统计等，这些代码会散落穿插在各个业务逻辑中，非常冗余且不利于维护。例如下面这个示意图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPzmZTrBxHPoicZQMVqZibzjTXSg1XdulKGaIQtkI0ibUmlcwor6CicCQia5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有多少业务操作，就要写多少重复的校验和日志记录代码，这显然是无法接受的。当然，用面向对象的思想，我们可以把这些重复的代码抽离出来，写成公共方法，就是下面这样：

<!--more-->

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPYXw0VPYribRboWEIlahvV1fp24loE65eicV6FwUku15Fuu2JDRyuPSog/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样，代码冗余和可维护性的问题得到了解决，但每个业务方法中依然要依次手动调用这些公共方法，也是略显繁琐。有没有更好的方式呢？有的，那就是AOP，AOP将权限校验、日志记录等非业务代码完全提取出来，与业务代码分离，并寻找节点切入业务代码中：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPS5RdGyLIcdibNb2maf99fNYsEY8VIuU4qic8v7icfIDSd4icibCRjxGiaWZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## AOP体系与概念

简单地去理解，其实AOP要做三类事：

- 在哪里切入，也就是权限校验等非业务操作在哪些业务代码中执行。
- 在什么时候切入，是业务代码执行前还是执行后。
- 切入后做什么事，比如做权限校验、日志记录等。

因此，AOP的体系可以梳理为下图：
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPaWB7wECf4kiczeXm7Ek9CcUGyQ4NIDxVfurrnA7Oul3kn7gtU87MNYg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
一些概念详解：

- `Pointcut`：切点，决定处理如权限校验、日志记录等在何处切入业务代码中（即织入切面）。切点分为`execution`方式和`annotation`方式。前者可以用路径表达式指定哪些类织入切面，后者可以指定被哪些注解修饰的代码织入切面。
- `Advice`：处理，包括处理时机和处理内容。处理内容就是要做什么事，比如校验权限和记录日志。处理时机就是在什么时机执行处理内容，分为前置处理（即业务代码执行前）、后置处理（业务代码执行后）等。
- `Aspect`：切面，即`Pointcut`和`Advice`。
- `Joint point`：连接点，是程序执行的一个点。例如，一个方法的执行或者一个异常的处理。在 Spring AOP 中，一个连接点总是代表一个方法执行。
- `Weaving`：织入，就是通过动态代理，在目标对象方法中执行处理内容的过程。

网络上有张图，我觉得非常传神，贴在这里供大家观详：
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPtiaNHJkbx98YP330aLgLeFKWCJcDoksHovAibu95IHSw7P06smMsH3Nw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# AOP实例

实践出真知，接下来我们就撸代码来实现一下AOP。完成项目已上传至：

> https://github.com/ThinkMugz/aopDemo

使用 AOP，首先需要引入 AOP 的依赖。参数校验：[这么写参数校验(validator)就不会被劝退了~](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247489603&idx=2&sn=c7a766500122c02694860bd2470204cf&chksm=ebd6276fdca1ae7997a98d8c038ad3462c7ee9b492f28b4de8b7461ca0965fac55d94f96dd37&scene=21#wechat_redirect)

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 第一个实例

接下来，我们先看一个极简的例子：所有的`get`请求被调用前在控制台输出一句"get请求的advice触发了"。

具体实现如下：

1. 创建一个AOP切面类，只要在类上加个 `@Aspect` 注解即可。`@Aspect` 注解用来描述一个切面类，定义切面类的时候需要打上这个注解。`@Component` 注解将该类交给 Spring 来管理。在这个类里实现advice：

```
package com.mu.demo.advice;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LogAdvice {
    // 定义一个切点：所有被GetMapping注解修饰的方法会织入advice
    @Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
    private void logAdvicePointcut() {}

 // Before表示logAdvice将在目标方法执行前执行
    @Before("logAdvicePointcut()")
    public void logAdvice(){
     // 这里只是一个示例，你可以写任何处理逻辑
        System.out.println("get请求的advice触发了");
    }
}
```

1. 创建一个接口类，内部创建一个get请求：

```
package com.mu.demo.controller;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(value = "/aop")
public class AopController {
    @GetMapping(value = "/getTest")
    public JSONObject aopTest() {
        return JSON.parseObject("{\"message\":\"SUCCESS\",\"code\":200}");
    }
    
 @PostMapping(value = "/postTest")
    public JSONObject aopTest2(@RequestParam("id") String id) {
        return JSON.parseObject("{\"message\":\"SUCCESS\",\"code\":200}");
    }
}
```

项目启动后，请求`http://localhost:8085/aop/getTest`接口：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPXuxoOL8FLX1O1aKM1lu2ibrITibkHDuacqWPeAZ2V5Lftz5gYQGL2icAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

请求`http://localhost:8085/aop/postTest`接口，控制台无输出，证明切点确实是只针对被`GetMapping`修饰的方法。

## 第二个实例

下面我们将问题复杂化一些，该例的场景是：

1. 自定义一个注解`PermissionsAnnotation`
2. 创建一个切面类，切点设置为拦截所有标注`PermissionsAnnotation`的方法，截取到接口的参数，进行简单的权限校验
3. 将`PermissionsAnnotation`标注在测试接口类的测试接口`test`上

具体的实现步骤：

1. 使用`@Target、@Retention、@Documented`自定义一个注解：

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PermissionAnnotation{
}
```

1. 创建第一个AOP切面类，，只要在类上加个 `@Aspect` 注解即可。`@Aspect` 注解用来描述一个切面类，定义切面类的时候需要打上这个注解。`@Component` 注解将该类交给 Spring 来管理。在这个类里实现第一步权限校验逻辑：

```
package com.example.demo;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Order(1)
public class PermissionFirstAdvice {

 // 定义一个切面，括号内写入第1步中自定义注解的路径
    @Pointcut("@annotation(com.mu.demo.annotation.PermissionAnnotation)")
    private void permissionCheck() {
    }

    @Around("permissionCheck()")
    public Object permissionCheckFirst(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("===================第一个切面===================：" + System.currentTimeMillis());

        //获取请求参数，详见接口类
        Object[] objects = joinPoint.getArgs();
        Long id = ((JSONObject) objects[0]).getLong("id");
        String name = ((JSONObject) objects[0]).getString("name");
        System.out.println("id1->>>>>>>>>>>>>>>>>>>>>>" + id);
        System.out.println("name1->>>>>>>>>>>>>>>>>>>>>>" + name);

        // id小于0则抛出非法id的异常
        if (id < 0) {
            return JSON.parseObject("{\"message\":\"illegal id\",\"code\":403}");
        }
        return joinPoint.proceed();
    }
}
```

1. 创建接口类，并在目标方法上标注自定义注解 `PermissionsAnnotation`：

```
package com.example.demo;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(value = "/permission")
public class TestController {
    @RequestMapping(value = "/check", method = RequestMethod.POST)
    // 添加这个注解
    @PermissionsAnnotation()
    public JSONObject getGroupList(@RequestBody JSONObject request) {
        return JSON.parseObject("{\"message\":\"SUCCESS\",\"code\":200}");
    }
}
```

在这里，我们先进行一个测试。首先，填好请求地址和header：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPibrIoVGd8YYHgGR41VurkPvgiaYugicJpfMSVU6vS33J8qtXdejont4iaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其次，构造正常的参数：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPSsb30kLU0VF8yY3KsWHVGCT76WIbia1DEOua09Hkrvdrp3DtnmObMzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以拿到正常的响应结果：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPJNF1iatvlw7cIZpO6SD6zkhZng9jd5N2VGHBLaOo1K7kmu2VGOiaJvrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后，构造一个异常参数，再次请求：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPa6wPdfE4CFCKFrrloXq4kNMX2nicQArmnUYCSssibgAdz7Mn9fzKCrBg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

响应结果显示，切面类进行了判断，并返回相应结果：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPErZfpdvO48NhZ8Jwv9WiculZNGmPAhgOCOGjlSBiaRVQGRjJJ4htNoMA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有人会问，如果我一个接口想设置多个切面类进行校验怎么办？这些切面的执行顺序如何管理？

很简单，一个自定义的`AOP`注解可以对应多个切面类，这些切面类执行顺序由`@Order`注解管理，该注解后的数字越小，所在切面类越先执行。

下面在实例中进行演示：

创建第二个AOP切面类，在这个类里实现第二步权限校验：

```
package com.example.demo;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Order(0)
public class PermissionSecondAdvice {

   @Pointcut("@annotation(com.example.demo.PermissionsAnnotation)")
   private void permissionCheck() {
   }

   @Around("permissionCheck()")
   public Object permissionCheckSecond(ProceedingJoinPoint joinPoint) throws Throwable {
       System.out.println("===================第二个切面===================：" + System.currentTimeMillis());

       //获取请求参数，详见接口类
       Object[] objects = joinPoint.getArgs();
       Long id = ((JSONObject) objects[0]).getLong("id");
       String name = ((JSONObject) objects[0]).getString("name");
       System.out.println("id->>>>>>>>>>>>>>>>>>>>>>" + id);
       System.out.println("name->>>>>>>>>>>>>>>>>>>>>>" + name);

       // name不是管理员则抛出异常
       if (!name.equals("admin")) {
           return JSON.parseObject("{\"message\":\"not admin\",\"code\":403}");
       }
       return joinPoint.proceed();
   }
}
```

重启项目，继续测试，构造两个参数都异常的情况：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPRjUPPVYDwBO0iafenXZibicTe8INww417icATAbhVADnicy4Js2UnS5S7DA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

响应结果，表面第二个切面类执行顺序更靠前：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPr9ZoScibZqiaILzkhx2q9AavWJwVvqLu7p1fN2njOL8ZLlxZghGWElnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

# AOP相关注解

上面的案例中，用到了诸多注解，下面针对这些注解进行详解。

## @Pointcut

`@Pointcut` 注解，用来定义一个切面，即上文中所关注的某件事情的入口，切入点定义了事件触发时机。

```
@Aspect
@Component
public class LogAspectHandler {

    /**
     * 定义一个切面，拦截 com.itcodai.course09.controller 包和子包下的所有方法
     */
    @Pointcut("execution(* com.mutest.controller..*.*(..))")
    public void pointCut() {}
}
```

@Pointcut 注解指定一个切面，定义需要拦截的东西，这里介绍两个常用的表达式：一个是使用 `execution()`，另一个是使用 `annotation()`。

更多参考：[SpringBoot内容聚合](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247488811&idx=2&sn=0d054027651bef114ae6bec5a8f62359&chksm=ebd62a07dca1a3111269692dd9b0a5ab8af7d2038b8ec6600a057573364e76db1332adc5fd46&scene=21#wechat_redirect)

execution表达式：

以 `execution(* * com.mutest.controller..*.*(..)))` 表达式为例：

- 第一个 * 号的位置：表示返回值类型，* 表示所有类型。
- 包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，在本例中指 com.mutest.controller包、子包下所有类的方法。
- 第二个 * 号的位置：表示类名，* 表示所有类。
- *(..)：这个星号表示方法名，* 表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数。

annotation() 表达式：

`annotation()` 方式是针对某个注解来定义切面，比如我们对具有 @PostMapping 注解的方法做切面，可以如下定义切面：

```
@Pointcut("@annotation(org.springframework.web.bind.annotation.PostMapping)")
public void annotationPointcut() {}
```

然后使用该切面的话，就会切入注解是 `@PostMapping` 的所有方法。这种方式很适合处理 `@GetMapping、@PostMapping、@DeleteMapping`不同注解有各种特定处理逻辑的场景。

还有就是如上面案例所示，针对自定义注解来定义切面。

```
@Pointcut("@annotation(com.example.demo.PermissionsAnnotation)")
private void permissionCheck() {}
```

## @Around

`@Around`注解用于修饰`Around`增强处理，`Around`增强处理非常强大，表现在：

1. `@Around`可以自由选择增强动作与目标方法的执行顺序，也就是说可以在增强动作前后，甚至过程中执行目标方法。这个特性的实现在于，调用`ProceedingJoinPoint`参数的`procedd()`方法才会执行目标方法。
2. `@Around`可以改变执行目标方法的参数值，也可以改变执行目标方法之后的返回值。

`Around`增强处理有以下特点：

1. 当定义一个`Around`增强处理方法时，该方法的第一个形参必须是 `ProceedingJoinPoint` 类型（至少一个形参）。在增强处理方法体内，调用`ProceedingJoinPoint`的`proceed`方法才会执行目标方法：这就是`@Around`增强处理可以完全控制目标方法执行时机、如何执行的关键；如果程序没有调用`ProceedingJoinPoint`的`proceed`方法，则目标方法不会执行。
2. 调用`ProceedingJoinPoint`的`proceed`方法时，还可以传入一个`Object[ ]`对象，该数组中的值将被传入目标方法作为实参——这就是`Around`增强处理方法可以改变目标方法参数值的关键。这就是如果传入的`Object[ ]`数组长度与目标方法所需要的参数个数不相等，或者`Object[ ]`数组元素与目标方法所需参数的类型不匹配，程序就会出现异常。

`@Around`功能虽然强大，但通常需要在线程安全的环境下使用。因此，如果使用普通的`Before`、`AfterReturning`就能解决的问题，就没有必要使用`Around`了。如果需要目标方法执行之前和之后共享某种状态数据，则应该考虑使用`Around`。尤其是需要使用增强处理阻止目标的执行，或需要改变目标方法的返回值时，则只能使用`Around`增强处理了。

下面，在前面例子上做一些改造，来观察`@Around`的特点。

自定义注解类不变。首先，定义接口类：

```
package com.example.demo;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(value = "/permission")
public class TestController {
    @RequestMapping(value = "/check", method = RequestMethod.POST)
    @PermissionsAnnotation()
    public JSONObject getGroupList(@RequestBody JSONObject request) {
        return JSON.parseObject("{\"message\":\"SUCCESS\",\"code\":200,\"data\":" + request + "}");
    }
}
```

唯一切面类（前面案例有两个切面类，这里只需保留一个即可）：

```
package com.example.demo;

import com.alibaba.fastjson.JSONObject;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;


@Aspect
@Component
@Order(1)
public class PermissionAdvice {

    @Pointcut("@annotation(com.example.demo.PermissionsAnnotation)")
    private void permissionCheck() {
    }


    @Around("permissionCheck()")
    public Object permissionCheck(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("===================开始增强处理===================");

        //获取请求参数，详见接口类
        Object[] objects = joinPoint.getArgs();
        Long id = ((JSONObject) objects[0]).getLong("id");
        String name = ((JSONObject) objects[0]).getString("name");
        System.out.println("id1->>>>>>>>>>>>>>>>>>>>>>" + id);
        System.out.println("name1->>>>>>>>>>>>>>>>>>>>>>" + name);

  // 修改入参
        JSONObject object = new JSONObject();
        object.put("id", 8);
        object.put("name", "lisi");
        objects[0] = object;
  
  // 将修改后的参数传入
        return joinPoint.proceed(objects);
    }
}
```

同样使用JMeter调用接口，传入参数：`{"id":-5,"name":"admin"}`，响应结果表明：`@Around`截取到了接口的入参，并使接口返回了切面类中的结果。
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbLsHvicPPQst11FJvVicm1woPo2hTYT8rI08VWKyNE8ib39pNicDw1NkyJutscMHh1sRTSwmibp5TjoIZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## @Before

`@Before` 注解指定的方法在切面切入目标方法之前执行，可以做一些 `Log` 处理，也可以做一些信息的统计，比如获取用户的请求 `URL` 以及用户的 `IP` 地址等等，这个在做个人站点的时候都能用得到，都是常用的方法。例如下面代码：

```
@Aspect
@Component
@Slf4j
public class LogAspectHandler {
    /**
     * 在上面定义的切面方法之前执行该方法
     * @param joinPoint jointPoint
     */
    @Before("pointCut()")
    public void doBefore(JoinPoint joinPoint) {
        log.info("====doBefore方法进入了====");

        // 获取签名
        Signature signature = joinPoint.getSignature();
        // 获取切入的包名
        String declaringTypeName = signature.getDeclaringTypeName();
        // 获取即将执行的方法名
        String funcName = signature.getName();
        log.info("即将执行方法为: {}，属于{}包", funcName, declaringTypeName);

        // 也可以用来记录一些信息，比如获取请求的 URL 和 IP
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        // 获取请求 URL
        String url = request.getRequestURL().toString();
        // 获取请求 IP
        String ip = request.getRemoteAddr();
        log.info("用户请求的url为：{}，ip地址为：{}", url, ip);
    }
}
```

`JointPoint` 对象很有用，可以用它来获取一个签名，利用签名可以获取请求的包名、方法名，包括参数（通过 `joinPoint.getArgs()` 获取）等。

## @After

`@After` 注解和 `@Before` 注解相对应，指定的方法在切面切入目标方法之后执行，也可以做一些完成某方法之后的 Log 处理。

```
@Aspect
@Component
@Slf4j
public class LogAspectHandler {
    /**
     * 定义一个切面，拦截 com.mutest.controller 包下的所有方法
     */
    @Pointcut("execution(* com.mutest.controller..*.*(..))")
    public void pointCut() {}

    /**
     * 在上面定义的切面方法之后执行该方法
     * @param joinPoint jointPoint
     */
    @After("pointCut()")
    public void doAfter(JoinPoint joinPoint) {

        log.info("==== doAfter 方法进入了====");
        Signature signature = joinPoint.getSignature();
        String method = signature.getName();
        log.info("方法{}已经执行完", method);
    }
}
```

到这里，我们来写个 Controller 测试一下执行结果，新建一个 AopController 如下：

```
@RestController
@RequestMapping("/aop")
public class AopController {

    @GetMapping("/{name}")
    public String testAop(@PathVariable String name) {
        return "Hello " + name;
    }
}
```

启动项目，在浏览器中输入：localhost:8080/aop/csdn，观察一下控制台的输出信息：

```
====doBefore 方法进入了====  
即将执行方法为: testAop，属于com.itcodai.mutest.AopController包  
用户请求的 url 为：http://localhost:8080/aop/name，ip地址为：0:0:0:0:0:0:0:1  
==== doAfter 方法进入了====  
方法 testAop 已经执行完
```

从打印出来的 `Log` 中可以看出程序执行的逻辑与顺序，可以很直观的掌握 `@Before` 和 `@After` 两个注解的实际作用。

## @AfterReturning

`@AfterReturning` 注解和 `@After` 有些类似，区别在于 `@AfterReturning` 注解可以用来捕获切入方法执行完之后的返回值，对返回值进行业务逻辑上的增强处理，例如：

```
@Aspect
@Component
@Slf4j
public class LogAspectHandler {
    /**
     * 在上面定义的切面方法返回后执行该方法，可以捕获返回对象或者对返回对象进行增强
     * @param joinPoint joinPoint
     * @param result result
     */
    @AfterReturning(pointcut = "pointCut()", returning = "result")
    public void doAfterReturning(JoinPoint joinPoint, Object result) {

        Signature signature = joinPoint.getSignature();
        String classMethod = signature.getName();
        log.info("方法{}执行完毕，返回参数为：{}", classMethod, result);
        // 实际项目中可以根据业务做具体的返回值增强
        log.info("对返回参数进行业务上的增强：{}", result + "增强版");
    }
}
```

需要注意的是，在 `@AfterReturning` 注解 中，属性 `returning` 的值必须要和参数保持一致，否则会检测不到。该方法中的第二个入参就是被切方法的返回值，在 `doAfterReturning` 方法中可以对返回值进行增强，可以根据业务需要做相应的封装。我们重启一下服务，再测试一下：

```
方法 testAop 执行完毕，返回参数为：Hello CSDN  
对返回参数进行业务上的增强：Hello CSDN 增强版
```

## @AfterThrowing

当被切方法执行过程中抛出异常时，会进入 `@AfterThrowing` 注解的方法中执行，在该方法中可以做一些异常的处理逻辑。要注意的是 `throwing` 属性的值必须要和参数一致，否则会报错。该方法中的第二个入参即为抛出的异常。

```
@Aspect
@Component
@Slf4j
public class LogAspectHandler {
    /**
     * 在上面定义的切面方法执行抛异常时，执行该方法
     * @param joinPoint jointPoint
     * @param ex ex
     */
    @AfterThrowing(pointcut = "pointCut()", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Throwable ex) {
        Signature signature = joinPoint.getSignature();
        String method = signature.getName();
        // 处理异常的逻辑
        log.info("执行方法{}出错，异常为：{}", method, ex);
    }
}
```