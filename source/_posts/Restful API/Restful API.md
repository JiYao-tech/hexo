---
title: Restful API！整合RestTemplate
tags: Restful API
copyright: true
abbrlink: 1462
date: 2020-06-07 21:27:53
---
# 概述

针对目前项目中使用的前后端分离开发，越来越感觉到API设计的重要性，而不好的API设计通常让使用者通过URL无法明确知道这个URL到底是干什么用的，并且会显得设计混乱，在此将使用Restful风格设计API进行总结，并对在SpringBoot中具体实现Restful API的设计给出一定示例。

# Restful API设计

使用Restful API是一种面向资源的设计风格，因此我们每一个URL对应的都是一个资源。从而针对资源的操作我们就可以使用HTTP的相应的请求方式来进行体现。

## 版本

应该将API的版本号放入URL中，例如http://localhost:8080/v1/。

## URL定义为名词

在之前我们通常看见/addUser、/editUser这样的URL定义，那么如果我们使用这样的定义就相当于我们放弃了HTTP请求方式（GET,POST,PUT,PATCH,DELETE）等的表达作用。并且我们也知道Restful提倡面向资源的URL路径，所以URL使用名词来进行定义。
例如：

1. 操作用户资源：http://localhost:8080/v1/user，
2. 操作多个用户资源：http://localhost:8080/v1/users，
3. 操作财务部下的用户资源：http://localhost:8080/v1/accounting-department/user，从这里我们可以学到一点，针对某个资源需要两个单词来进行表达的时候我们可以使用“-”将两个单词连接起来。

## 使用HTTP请求方法来表达对资源的操作

上面我们知道了Restful API的URL定义是面向资源的，也就是每一个URL表示的是一种资源，那我我们要表达对资源的相关操作怎么办呢？而且Restful风格也强调了URL必须为名词构成。这里我们就可以使用HTTP的请求方式来表达对资源的操作了。

- GET：获取资源。
- POST:创建资源。
- PUT:修改资源。
- PATCH:修改资源，这里的修改资源和PUT的区别在于这里只向后台传递了修改资源的部分内容，而PUT传递的是全部内容。
- DELETE：删除资源。还有两个不常用的HTTP动词。HEAD：获取资源的元数据。OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

## 数据传递格式

JSON阅读性更高，扩展性更强，适合各种环境和语言进行解析，现在大的互联网公司，对外提供的API基本都使用JSON。

## 使用HTTP状态码

针对请求的响应，我们可以直接利用Http的状态码来表达错误信息，这样也可以很形象地得知每次请求的结果，而不是每次请求都是200，然后再去响应体中获取是否有错误信息，这样响应体没有结果封装，没有额外的传输负担。
针对Http状态码无法表示的异常信息，我们可以增加自定义异常码。增加自定义头字段，Error-Message 表示错误信息，只在出现异常的时候返回。下面的表格内容参考自菜鸟教程。

**HTTP状态码共分为5种类型：**

| 分类 | 分类描述                                       |
| ---- | ---------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

**HTTP状态码列表:**

| 状态码 | 状态码英文名称                  | 中文描述                                                     |
| ------ | ------------------------------- | ------------------------------------------------------------ |
| 100    | Continue                        | 继续。客户端应继续其请求                                     |
| 101    | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
| 200    | OK                              | 请求成功。一般用于GET与POST请求                              |
| 201    | Created                         | 已创建。成功请求并创建了新的资源                             |
| 202    | Accepted                        | 已接受。已经接受请求，但未处理完成                           |
| 203    | Non-Authoritative Information   | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
| 204    | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205    | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206    | Partial Content                 | 部分内容。服务器成功处理了部分GET请求                        |
|        |                                 |                                                              |
| 300    | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301    | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302    | Found                           | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 303    | See Other                       | 查看其它地址。与301类似。使用GET和POST请求查看               |
| 304    | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305    | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                       |
| 306    | Unused                          | 已经被废弃的HTTP状态码                                       |
| 307    | Temporary Redirect              | 临时重定向。与302类似。使用GET请求重定向                     |
|        |                                 |                                                              |
| 400    | Bad Request                     | 客户端请求的语法错误，服务器无法理解                         |
| 401    | Unauthorized                    | 请求要求用户的身份认证                                       |
| 402    | Payment Required                | 保留，将来使用                                               |
| 403    | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404    | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 405    | Method Not Allowed              | 客户端请求中的方法被禁止                                     |
| 406    | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                   |
| 407    | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408    | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                     |
| 409    | Conflict                        | 服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突 |
| 410    | Gone                            | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
| 411    | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
| 412    | Precondition Failed             | 客户端请求信息的先决条件错误                                 |
| 413    | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414    | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理               |
| 415    | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                             |
| 416    | Requested range not satisfiable | 客户端请求的范围无效                                         |
| 417    | Expectation Failed              | 服务器无法满足Expect的请求头信息                             |
|        |                                 |                                                              |
| 500    | Internal Server Error           | 服务器内部错误，无法完成请求                                 |
| 501    | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                         |
| 502    | Bad Gateway                     | 充当网关或代理的服务器，从远端服务器接收到了一个无效的请求   |
| 503    | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
| 504    | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
| 505    | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理               |

# GET接口

get请求通常表达获取某种资源。

（1）获取多个资源。

针对多个资源的获取我们可以使用url中的名词为复数形式进行标记为获取多个，具体示例如下：

    //获取多个用户信息
    @GetMapping(value="/users")
    public List<User> getUsers(){
        ... ...
    }
（2）获取单个资源，

并且按照田间进行筛选。这里我们就需要使用到url参数来传递筛选条件。具体示例如下：

    //获取单个用户信息
    @GetMapping("/user")
    public User getUser(
            @RequestParam(value = "name",defaultValue = "liutao")String name,
            @RequestParam(value = "age",defaultValue = "10") int age) {
     
            ... ...
    }
上面例子的name和age就为筛选条件,具体url类似http://locahost:8888/api-demo/user?name=XXXX&age=10。

当我们想要参数的合法性，如果不合法的时候，我们就返回错误状态码及错误信息。这里需要注意，我们不直接在响应内容中返回错误信息，这样会加重对响应内容的封装负担。我们直接在响应头中添加错误信息，具体如下：

```java
if(age < 0){
            HttpServletResponse response = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getResponse();
            response.setStatus(10004);
            response.addHeader("Error-Message","Parameter is invalid");
            return null;
        
```

这个时候如果请求传入的age < 0，响应的结果就会如下：

![](https://myblog-1259323290.cos.ap-nanjing.myqcloud.com/Typora/202006/07/211822-661225.png)

我们就可以根据响应状态码来获取请求状态，如果请求错误，我们就可以从请求头获取Error-Message来得到错误信息。

（3）如果我们想要限定用户所属部门，但是这个时候我们又不想要使用路径参数，咋办呢？这个时候就应该能想到Spring中有一个注解PathVariable，具体示例如下：

    /**
     * 获取某个部门下的某个用户
     * 这里针对部门的限制使用了路径参数，使用PathVariable标签来获取部门信息
     *
     * @param dept
     * @param name
     * @return
     */
    @GetMapping("/{dept}/user")
    public User getUserOfDept(
            @PathVariable("dept") String dept,
            @RequestParam(value = "name")String name,
            HttpServletRequest request
    ){
            ... ...
     }
这个时候请求路径可以是这样的 http://locahost:8888/api-demo/finance/user?name=XXXX。

然后我们就可以获取到部门信息（dept）为finance。

在我们通常的API中涉及到权限认证token的问题，我们可以直接将token存放在请求头里面，然后在接口里面获取请求头里面的token，进行权限认证，如下：

String token = request.getHeader("token");
当然针对上面这一步建议直接在拦截器里面做处理。

# POST接口

post请求表达对某种资源的创建。

（1）针对Restful API我们建议数据传递使用json格式。示例如下：

    //添加用户信息
    @PostMapping("/json/user")
    public int addUserToSpecialDepart(
            @RequestParam("dept") String dept,
            @RequestBody User user
    ){
        ... ... 
    }
RequestBody标签会自动将json格式的数据转换成对象。

（2）上面我们看了post接口如何接收json格式，那么如果我们要接收form表单数据呢？这个时候就需要将RequestBody注解换成ModelAttribute,示例如下：

    /**
     * 添加用户信息
     * 请求参数支持表单数据
     * @param user
     * @return
     */
    @PostMapping("/form/user")
    public int addUserByForm(@ModelAttribute User user){
        ... ... 
    }
# PUT接口

put请求表示对某种资源的修改，但是我们需要注意put的修改与patch相比，put需要传入修改对象的全部数据，二patch仅仅需要传递部分数据。


    //修改用户信息（传入被修改对象的全部）
    @PutMapping("user")
    public int updateUser(@RequestBody User user){
        ... ...
    }
# PATCH接口

patch接口表示修改，需要传入修改对象的部分内容。

    /**
     * 修改用户名（传入被修改对象的部分信息）
     * @param id
     * @param name
     * @return
     */
    @PatchMapping("user/{id}")
    public int updateUserName(
            @PathVariable("id")String id,
            @RequestParam("name") String name
    ){
        ... ...
    }
# DELETE接口

delete请求表示删除某个资源，其实delete请求和get请求的使用方式是相同的，不同的仅仅是请求动词。

    /**
     * 删除用户信息
     * @return
     */
    @DeleteMapping("user")
    public int getUser(
            @RequestParam(value = "name",defaultValue = "liutao")String name,
            @RequestParam("age") int age) {
        ... ...
    }

# RestTemplate

## 创建RestTemplate

在项目中我们通常在项目启动的时候就在Spring容器中创建一个RestTemplate的bean，在需要的时候我们直接注入就行了。

    @Configuration
    public class RestTemplateConfig {
        @Value("${restTemplate.connectionRequestTimeout}")
        private int connectionRequestTimeout; //连接请求超时时间
    
        @Value("${restTemplate.connectionTimeout}")
        private int connectionTimeout;        //连接超时时间
    
        @Value("${restTemplate.readTimeout}")
        private int readTimeout;              //读取超时时间
    
        @Bean
        @Primary
        public RestTemplate customRestTemplate(){
            HttpComponentsClientHttpRequestFactory httpRequestFactory = new 			HttpComponentsClientHttpRequestFactory();
            httpRequestFactory.setConnectionRequestTimeout(connectionRequestTimeout);
            httpRequestFactory.setConnectTimeout(connectionTimeout);
            httpRequestFactory.setReadTimeout(readTimeout);
            return new RestTemplate(httpRequestFactory);
    	}
    }

这里我们需要在配置文件中添加相应的配置参数：

```yml
restTemplate.connectionRequestTimeout = 300000
restTemplate.connectionTimeout = 300000
restTemplate.readTimeout = 300000
```

## RestTemplate使用

### RestTemplate之get请求

#### (1) getForObject

```java
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables)throws  RestClientException
```

**参数意义：**

**url：**url地址

**responseType：**响应实体类型。

**uriVariables：**url地址参数，如果没有url参数的时候直接传空Map(new HashMap())。
示例：

    @Test
    public void testGetForObject_one(){
        String url = HOST +"/api-demo/user";
        Map<String,Object> paramMap = new HashMap<>();
        paramMap.put("name","rose");
        paramMap.put("age",22);
        User user = restTemplate.getForObject(url+"?name={name}&age={age}",User.class,paramMap);
        logger.debug("user:"+user);
    }
#### (2) getForObject

```java
 public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables)
     throws RestClientException
```

**参数意义：**

**url：**url地址

**responseType：**响应实体类型。

**uriVariables：**url地址参数，如果url地址上没有参数的，这个参数可以不填。
示例：

    @Test
    public void testGetForObject_two(){
        String url = HOST +"/api-demo/user";
        Object[] arr = new Object[]{"rose", 22};
     
        //注意这种调用方式针对URL中没有参数的情况可以不传入最后的一个参数
        User user = restTemplate.getForObject(url+"?name={name}&age={age}",User.class,arr);
        logger.debug("user:"+user);
    }
#### (3)getForEntity

```java
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables)
     throws RestClientException
```

此方法与**(1) getForObject**方法的使用相同，不同的是这个方法的返回结果为ResponseEntity，可以从返回结果获取响应状态及响应体等信息。

处理响应体的代码如下：

    /**
     * 处理响应responseEntity
     * @param responseEntity
     */
    private void disposeResponseEntity(ResponseEntity<User> responseEntity) {
        //获取响应状态code
        int code = responseEntity.getStatusCodeValue();
        logger.debug("httpstatus code:"+code);
     
        //获取响应头相关信息
        if(code > 1000){
            String errorMessage = String.valueOf(responseEntity.getHeaders().get("Error-Message"));
            logger.debug("Error-Message:"+errorMessage);
        }
     
        //获取响应体
        User user = responseEntity.getBody();
        logger.debug("user:"+user);
    }
示例：

    @Test
    public void testGetFoEntity_one(){
        String url = HOST +"/api-demo/user";
        Map<String,Object> paramMap = new HashMap<>();
        paramMap.put("name","rose");
        paramMap.put("age",22);
        ResponseEntity<User> responseEntity = restTemplate.getForEntity(url+"?name={name}&age={age}",User.class,paramMap);
        disposeResponseEntity(responseEntity);
    }
#### (4)getForEntity

```java
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables)
     throws RestClientException
```

此方法与**(2) getForObject**方法的使用相同，不同的是这个方法的返回结果为ResponseEntity，可以从返回结果获取响应状态及响应体等信息。

示例：

    @Test
    public void testGetFoEntity_two(){
        String url = HOST +"/api-demo/user";
        Object[] arr = new Object[]{"rose", -10};
        ResponseEntity<User> responseEntity = restTemplate.getForEntity(url+"?name={name}&age={age}",User.class,arr);
        disposeResponseEntity(responseEntity);
    }
#### (5)getForEntity

```java
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType)
     throws RestClientException
```

**参数意义：**

**参数意义：**url为URI对象URI uri = new URI("http://localhost:8080/api/demo?name=zhangsan")，如果有url参数需要拼接在地址后面。

**responseType：**响应实体类型。
示例：

    @Test
    public void testGetFoEntity_three(){
        String url = HOST +"/api-demo/user";
        url = url+"?name=rose&age=22";
        URI uri = null;
        try {
            uri = new URI(url);
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        ResponseEntity<User> responseEntity = restTemplate.getForEntity(uri,User.class);
     
        disposeResponseEntity(responseEntity);
    }
### RestTemplate之post请求

#### (1)postForObject

```java
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType,
     Object... uriVariables) throws RestClientException
```

**参数意义：****

**url：**url地址

**request：**请求实体对象

**responseType：**响应类型

**uriVariables：**url地址参数，如果url地址上没有参数的，这个参数可以不填，使用和**(1)getForObject**相同。

**类似方法1：****

```java
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException
```

方法和**(1)getForObject**基本相同，使用也相同，不同的是此方法最后一个url参数的传值使用的是Map,因此如果没有url参数的时候直接传空Map(new HashMap())。

**类似方法2：****

```java
public <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType) throws RestClientException

```

方法和**(5)getForEntity**基本相同，使用也相同，不同的事此方法的url参数是一个URI对象，并且没有最后一个参数，因此如果url有参数的时候， 需要我们直接拼接在url后面。例如：URI uri = new URI("http://localhost:8080/api/demo?name=zhangsan")。

示例：

    @Test
    public void testPostForObject_one(){
        String url = HOST +"/api-demo/json/user?dept={dept}";
        Object[] arr = new Object[]{"finance"};
        int result = restTemplate.postForObject(url,new User("liutao",12,"liutao123"),int.class,arr);
        logger.debug("result:"+result);
    }
#### (2)postForEntity

```java
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request,Class<T> responseType,
     Object... uriVariables) throws RestClientException
```

**参数意义：**

**url：**url地址

**request：**请求实体对象

**responseType：**响应类型

**uriVariables：**url地址参数：，如果url地址上没有参数的，这个参数可以不填，使用和**(4)getForEntity**相同。同样，当url中没有参数的时候，最后一个参数uriVariables可以不填。

**类似方法1：**

```java
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request,Class<T> responseType,
Map<String, ?> uriVariables) throws RestClientException
```

方法和**(3)getForEntity**基本相同，使用也相同，不同的是此方法最后一个url参数的传值使用的是Map,因此如果没有url参数的时候直接传空Map(new HashMap())。

**类似方法2：**

```java
public <T> ResponseEntity<T> postForEntity(URI url, @Nullable Object request, Class<T> responseType)throws
                    RestClientException
```

方法和**(5)getForEntity**基本相同仅仅需要将第一个参数换作URI即可。例如第一个参数为：URI uri = new URI("http://localhost:8080/api/demo");

示例：

    @Test
    public void testPostForEntity(){
        String url = HOST +"/api-demo/json/user?dept={dept}";
        Object[] arr = new Object[]{"finance"};
        ResponseEntity<Integer> responseEntity = restTemplate.postForEntity(url,new User("liutao",12,
                        "liutao123"),
                Integer.class,arr);
        logger.debug("responseEntity:"+responseEntity);
    }
（3）演示数据传递以form表单的格式进行

    @Test
    public void testPostForm(){
        String url = HOST +"/api-demo/form/user";
     
        //设置请求数据的格式
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
     
        //封装参数
        MultiValueMap<String, String> params= new LinkedMultiValueMap<>();
        params.add("name", "liutao");
        params.add("age", "12");
        params.add("password", "liutao123");
     
        //封装请求内容
        HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(params,headers);
        ResponseEntity<Integer> responseEntity = restTemplate.postForEntity(url,requestEntity,Integer.class);
        logger.debug("responseEntity:"+responseEntity);
    }
### RestTemplate之put请求

#### (1)put

```java
public void put(String url, @Nullable Object request, Object... uriVariables)
     throws RestClientException
```

**参数意义：**

**url：**url地址

**request：**请求实体对象

**uriVariables：**url地址参数：，如果url地址上没有参数的，这个参数可以不填，的使用和 **(2) getForObject**相同。

RestTemplate 的其余两个put请求方法和前面的get和post的对应方法使用类似，我们可以查看源码和参考前面的get、post方法的相应方法进行学习使用。
注意：这里的put方法没有获取任何响应，那么如果我们要获取响应咋个办呢？那就只有直用exchange方法来实现put请求。

示例：

    @Test
    public void testPut(){
        String url = HOST +"/api-demo/user";
        User user = new User("liutao",12,"liutao123");
        restTemplate.put(url,user);
    }
### RestTemplate之patch请求

#### (1)patchForObject

```java
public <T> T patchForObject(String url, @Nullable Object request, Class<T> responseType,
     Object... uriVariables) throws RestClientException
```

**参数意义：**

**url：**url地址

**request：**请求实体对象

**uriVariables：**url地址参数：，如果url地址上没有参数的，这个参数可以不填，的使用和**(2) getForObject** 相同。
     
**类似方法1：**

```java
public <T> T patchForObject(String url, @Nullable Object request, Class<T> responseType,
                    Map<String, ?> uriVariables) throws RestClientException
```

方法和**(1) getForObject**基本相同，使用也相同，不同的是此方法最后一个url参数的传值使用的是Map,因此如果没有url参数的时候直接传空Map(new HashMap())。

**类似方法2：**

```java
public <T> T patchForObject(URI url, @Nullable Object request, Class<T> responseType)
                    throws RestClientException

```

这个方法和**(5)getForEntity**使用基本相同，不同的是url。例如第一个参数为：URI uri = new URI("http://localhost:8080/api/demo");

示例：

    @Test
    public void testPatch(){
        String url = HOST +"/api-demo/user/1212?name={name}";
        Object[] arr = new Object[]{"rose"};
        Integer result = restTemplate.patchForObject(url,null,Integer.class,arr);
        logger.debug("result:"+result);
    }
### RestTemplate之DELETE请求

#### (1)delete

```java
public void delete(String url, Object... uriVariables)
```

**参数意义：**

**url：**url地址

**uriVariables：**url地址参数：，如果url地址上没有参数的，这个参数可以不填，uriVariables的传递和get、post的类似方法相同。
RestTemplate的其余两个delete请求方法和前面的get和post的对应方法使用类似，我们可以查看源码和前面的get、post方法的相应方法进行学习使用。

注意：这里的delete方法没有获取任何响应，那么如果我们要获取响应咋个办呢？那就只有直用exchange方法来实现delete请求。

示例：

    @Test
    public void testDelete(){
        String url = HOST +"/api-demo/user?name={name}&age={age}";
        Object[] arr = new Object[]{"rose", 10};
        restTemplate.delete(url,arr);
    }
### RestTemplate之EXCHANGE请求

#### (1)exchange

```java
public <T> ResponseEntity<T> exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, Class<T> responseType, Object... uriVariables) throws RestClientException
```



**参数意义：**

**url：**url地址

**method：**http请求方法，通过HttpMethod枚举类获取

**requestEntity**：请求实体，可以包含请求头和请求体的信息

**responseType：**响应类型

**uriVariables：**url地址参数，如果url地址上没有参数的，这个参数可以不填
此方法可以从返回结果ResponseEntity中获取到响应状态码以及响应头和响应体等信息。

**类似方法1：**

```java
public <T> ResponseEntity<T> exchange(String url, HttpMethod method,@Nullable HttpEntity<?> requestEntity,                         Class<T> responseType, Map<String, ?> uriVariables)throws RestClientException
```

 方法和(1)基本相同，使用也相同，不同的是此方法最后一个url参数的传值使用的是Map,因此如果没有url参数的时候直接传空Map(new HashMap())。

**类似方法2：**

```java
public <T> ResponseEntity<T> exchange(URI url, HttpMethod method, @Nullable HttpEntity<?> requestEntity,
                    Class<T> responseType) throws RestClientException
```

方法和(1)基本相同，使用也相同，不同的事此方法的url参数是一个URI对象，并且没有最后一个参数，因此如果url有参数的时候，需要我们直接拼接在url后面。例如：URI uri = new URI("http://localhost:8080/api/demo?name=zhangsan")

示例：

    @Test
    public void testExchange_one(){
     
        //封装请求头
        MultiValueMap<String,String> headers = new LinkedMultiValueMap<String,String>();
        headers.add("Accept","application/json");
        headers.add("Content-Type","application/json");
     
        //封装请求内容
        User user = new User("dasf",12,"liutao123");
        HttpEntity<User> requestEntity = new HttpEntity(user,headers);
     
        String url = HOST +"/api-demo/user";
        ResponseEntity<Integer> responseEntity = restTemplate.exchange(url,
                HttpMethod.PUT,requestEntity,Integer.class);
        logger.debug("responseEntity:"+responseEntity);
    }
(2)演示将token放置再请求头中传递至后台进行验证,通过exchange方法提交get请求，并在参数中通过指定requestHeaders来设置请求头参数

示例：

    @Test
    public void howToSendToken(){
        String url = HOST +"/api-demo/finance/user?name={name}";
        Map<String,Object> paramMap = new HashMap<>();
        paramMap.put("name","rose");
        //设置请求头数据
        HttpHeaders requestHeaders = new HttpHeaders();
        requestHeaders.set("token",UUID.randomUUID().toString());
        HttpEntity<String> requestEntity = new HttpEntity<>(null, requestHeaders);
        ResponseEntity<User> responseEntity = restTemplate.exchange(url,
                HttpMethod.GET,requestEntity,User.class,paramMap);
        logger.debug(responseEntity.getBody().toString());
    }

