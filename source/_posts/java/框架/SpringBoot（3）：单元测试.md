---
title: test心得
date: 2018-09-23 00:02:45
tags:
---
# 进行模块测试 #
## 单元测试 ##
有责任的开发人员都应该写单元测试

对项目进行打包的时候会自动执行单元测试 mvn clean package
测试内容：
- service
- API

# 代码

## 最终效果

- 模拟mockMvc对API进行测试 
- 对spring-boot的Test进行简单的整理


# 实现

## 基础环境搭建

首先要对类有一个注解

``` java
//在JUnit中有很多个Runner，他们负责调用你的测试代码，每一个Runner都有各自的特殊功能，
@RunWith(SpringRunner.class)
@SpringBootTest
public class BaseControllerTest{
```

对一些常见的test注释的介绍
``` java
    /**
     * 注解详解:
     *
     * @BeforeClass :所有测试方法前执行一次，一般在其中写上整体初始化的代码
     * @AfterClass 在所有测试方法后执行一次，一般在其中写上销毁和释放资源的代码
     * @Test(timeout = 1000) 测试方法执行超过1000毫秒后算超时，测试将失败
     * @Test(expected = Exception.class) 测试方法期望得到的异常类，如果方法执行没有抛出指定的异常，则测试失败
     * @Ignore(“not ready yet”)        执行测试时将忽略掉此方法，如果用于修饰类，则忽略整个类
     * @Test 编写一般测试用例
     * @Transactional，test对数据库进行操作时，可以在测试过后回滚数据库状态，因此仅仅只是测试
     */ 
```

## 对service进行测试

这个是比较简单的，和常规的没有什么差别

有一个需要注意的点是事务方面，需要添加注释@Transactional，这样的话，如果进行了数据库的操作，那么就会进行回滚

## 对API进行测试

想要对API进行测试

首先需要对类加一个新的注释@WebAppConfiguration

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@WebAppConfiguration
public class BaseControllerTest {
```

要模拟创建一个mock的环境

```java
    @Autowired
    private WebApplicationContext context;
    private MockMvc mockMvc;
```

同时，有了变量之后，是没有初始化的，这个时候要用到@Before，在测试前进行初始

```java
    /**
     * 构造mockMvc,初始化mock
     *
     * @throws Exception
     */
    @Before
    public void setUp() throws Exception {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }
```

环境搭建好了，那么接下来进行一手mock，注释在里面写好了，这个方法将参数写在了里面

对参数的解析使用了json-path，需要添加pom

``` xml
        <!--用于检测JSON格式的响应数据-->
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
        </dependency>
```

接下来开始真正的代码

```java
    /**
     * 进行参数的请求,模拟mock
     *
     * @throws Exception
     */
    @Test
    public void getMap() throws Exception {
        //调用接口
        MvcResult result = (MvcResult) mockMvc
                //使用get方法
                .perform(get("/test/get")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                //传入参数
                .param("userId", "11").
                        param("userName", "henry")
                //接收的类型
                .accept(MediaType.APPLICATION_JSON))
                //判断接收到的状态是否是200
                .andExpect(status().isOk())
                //打印内容
                .andDo(print())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                //匹配返回值中的内容
                .andExpect(content().string(Matchers.containsString("OK")))
                //使用jsonPath解析返回值，判断具体的内容 
                //需要学习jsonpath
                .andExpect(jsonPath("$.errcode", is(0)));
        int statusCode = result.getResponse().getStatus();
        Assert.assertEquals(statusCode, 200);
    }

```

在实际中，有可能参数是一个类

```java
    /**
     * 测试添加用户接口
     *
     * @throws Exception
     */
    @Test
    public void testAddUser() throws Exception {
        //构造添加的用户信息
        UserInfo userInfo = new UserInfo();
        userInfo.setName("testuser2");
        userInfo.setAge(29);
        userInfo.setAddress("北京");
        ObjectMapper mapper = new ObjectMapper();
        //调用接口，传入添加的用户参数
        mockMvc.perform(post("/user/adduser")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                //将整个userInfo当做参数传入
                .content(mapper.writeValueAsString(userInfo)))
                .andExpect(status().isOk())
                //使用jsonPath解析返回值，判断具体的内容
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                //判断返回值，是否达到预期，
                //测试示例中的返回值的结构如下{"errcode":0,"errmsg":"OK","p2pdata":null}
                .andExpect(jsonPath("$.errcode", is(0)))
                .andExpect(jsonPath("$.p2pdata", notNullValue()))
                .andExpect(jsonPath("$.p2pdata.id", not(0)))
                .andExpect(jsonPath("$.p2pdata.name", is("testuser2")));
    }
```


### 注解

#### 类注解

 - @RunWith(SpringRunner.class)//底层junit测试工具,在测试环境跑
 - @SpringBootTest //启动整个Spring的工程

#### 方法注解

- @Test
- @Transactional，test对数据库进行操作时，可以在测试过后回滚数据库状态，因此仅仅只是测试

### 对service测试 ###

自己编写方法测试
>
- test文件夹下，新建用于测试的.java文件
- 添加注解
- 右键Run 该test类  或者 指定的方法 进行测试
- 显示test passed 则测试成功，test failed 则测试失败

使用IDE
>
- 选中测试的方法，点击GoTo，点击Test 点击create
- 默认会写好方法注释@Test,但是没类注释

### 对API测试 ###

- 可以测试的内容：
>
- URL
- 返回的内容

- 示例
>
- 添加类注释和方法注释
- 添加新的类注解@AutoConfigureMockMvc
- 注入MockMvc类
- 使用mvc.perform进行URL请求测试
- mvc.perform(MockMvcRequestBuilders.get("/grils"))  请求的路径
- .andExpect(MockMvcResultMathers.status().isOk())对返回的状态码进行判断
- .andExpect(MockMvcResultMathers.content().string("abc"));对返回的内容进行判断，是否等于abc

### 判断是否成功 ###

- 使用断言，如查询一个对象是否存在
`Assert.assertNotNull(result);`

# 参考

1. [springboot项目中使用MockMvc 进行测试](https://blog.csdn.net/qq_33996921/article/details/79076951)
2. [SpringMVC 测试 mockMVC](https://www.cnblogs.com/lyy-2016/p/6122144.html)