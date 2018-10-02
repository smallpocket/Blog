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

### 注解

####类注解

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