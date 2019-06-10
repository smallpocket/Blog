---
title: controller心得
date: 2018-09-23 00:01:54
categories: "springBoot"
type: "tags"
tags: [java,spring boot]
---
# Controller #

### @Controller ###

- 处理HTTP请求 
- 需要配合模板使用,类似于JSP等，不常用
模板：
1. 导入包 spring-boot-starter-thymeleaf
2. 新建resoucres.templates.index.html
3. 方法当中return “index”

### @RestController ###

Spring4之后新加的注解，返回JSON

### @RequestMapping ###

- 配置URL映射
- 对应于两个URL，使用集合{“/hello”，"/hi"}
- 使用method确认提交方式（GET等）
- 使用GetMapping,PostMapping

### @PathVariable ###

- 获取URL中的数据
- URL路径为say/ID  
- 认为设定Mapping路径{ID}/say

### @RequestParam ###

- 获取请求参数的值
- /say?id=100
- value="id",required=false,defaultValue="100",不必然传值，默认值为100

### 可以写一个对象获取请求参数 ###

### @GetMapping ###

- 组合注解

## 最简单的HelloWorld ##
    /**
     * Spring Boot HelloWorld案例
     *
     * Created by bysocket on 16/4/26.
     */
    @RestController
    public class HelloWorldController {
     
    	@RequestMapping("/")
    	public String sayHello() {
    	return "Hello,World!";
    	}
    }
1. @RestController：提供实现了RESTAPI，可以服务JSON,XML或者其他。这里是以String的形式渲染出结果
2. @RequestMapping：提供路由信息，"/“路径的HTTP Request都会被映射到sayHello方法进行处理。 
---
