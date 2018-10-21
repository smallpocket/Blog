---
title: Hexo搭建
date: 2018-04-19 08:22:29
type: "tags"
tags: 
	- 建站
	- start
categories: 笔记
description: hexo+next进行博客的搭建
---

# 新博客 #

## 新建页面 ##

1. git bash下，定位到 Hexo 站点目录下。
2. 使用 hexo new page 新建一个页面，命名为 tags ：即 hexo new page tags
3. 或者hexo new posts "" 新建一个文章 ，默认为posts,即在post文件夹下面新建
4. 或者hexo new draft 新建一个草稿
5. 在source/_posts/路径下即可见

文章的头部：

	title: postName #文章页面上的显示名称，一般是中文
	date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
	updated: 2017-09-05 20:18:54 #手动添加更新时间
	type: "tags" #类型设置为 tags ，主题将自动为这个页面显示标签云
	categories: 默认分类 #分类
	tags:        #文章标签，可空，多标签请用格式，注意:后面有个空格
		- tag1
		- tag2
		- tag3 
	description: 附加一段文章摘要，字数最好在140字以内，会出现在meta的description里面
	comments: false #如果集成了评论，则关闭评论

### 使文章显示部分内容 ###

### 部署 ### 

- hexo generate 简写hexo g 生成静态页面
- hexo clean 清除缓存文件 (db.json) 和已生成的静态文件 (public)。
- hexo deploy 简写hexo d 将内容部署到网站
- hexo publish 发布内容，实际上是将内容从drafts（草稿）文件夹移到posts（文章）文件夹。
- hexo server 简写hexo s 启动服务器，默认情况下，访问网站为http://localhost:4000/
- 步骤： n c s g d

## 内容标签 ##

### Bootstrap Callout ###

- 这些样式出现在[http://getbootstrap.com/](http://getbootstrap.com/ " Bootstrap 的官方文档") 中。

#### 使用方式 ####

- {% note class_name %} Content (md partial supported) {% endnote %}
- class_name 可以是以下列表中的一个值：
- default
- primary
- success
- info
- warning
- danger

### 文本居中的引用 ###

- 此标签将生成一个带上下分割线的引用，同时引用内文本将自动居中。 
- 文本居中时，多行文本若长度不等，视觉上会显得不对称，因此建议在引用单行文本的场景下使用。 
- 例如作为文章开篇引用 或者 结束语之前的总结引用。

#### 使用方式 ####

- HTML方式：使用这种方式时，给 img 添加属性 class="blockquote-center" 即可。
- 标签方式：使用 centerquote 或者 简写 cq。
	
	<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
	<!-- 其中 class="blockquote-center" 是必须的 -->
	<blockquote class="blockquote-center">blah blah blah</blockquote>
	
	<!-- 标签 方式，要求版本在0.4.5或以上 -->
	{% centerquote %}blah blah blah{% endcenterquote %}
	
	<!-- 标签别名 -->
	{% cq %} blah blah blah {% endcq %}

### 突破容器宽度限制的图片 ###

- 当使用此标签引用图片时，图片将自动扩大 26%，并突破文章容器的宽度。 
- 此标签使用于需要突出显示的图片, 图片的扩大与容器的偏差从视觉上提升图片的吸引力。

#### 使用方式 ####

- HTML方式：使用这种方式时，为 img 添加属性 class="full-image"即可。
- 标签方式：使用 fullimage 或者 简写 fi， 并传递图片地址、 alt 和 title 属性即可。 属性之间以逗号分隔。

	<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
	<!-- 其中 class="full-image" 是必须的 -->
	<img src="/image-url" class="full-image" />
	
	<!-- 标签 方式，要求版本在0.4.5或以上 -->
	{% fullimage /image-url, alt, title %}
	
	<!-- 别名 -->
	{% fi /image-url, alt, title %}

# 插件 #

## Hexo-admin ##

Hexo-admin插件允许我们直接在本地页面上修改文章内容。
- 下载
	npm i hexo-admin --save
- 查看，登录http://localhost:4000/admin即可看到我们所有的文章内容，并且在可视化界面中操作文章内容

# 参考
1. [为你的Hexo加上评论系统-Valine](https://blog.csdn.net/blue_zy/article/details/79071414)




