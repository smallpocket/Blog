# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
列表
- 我是第一行
- 我是第二行
* 我是第三行
* 我是第四行
引用
> 我一直在使用引用。
> 从开始到现在。
图片为：![]()
链接为：[]()
**粗体**
*斜体*
横线***
\\ 反斜杠 
\` 反引号 
\* 星号 
\_ 下划线 
\{} 大括号 
\[] 中括号 
\() 小括号 
\# 井号 
\+ 加号 
\- 减号 
\. 英文句号 
\! 感叹号

ctrl+1 一级标题 
ctrl+2 二级标题 
····
ctrl+shift+o 有序列表
ctrl+u 无序列表
ctrl+g 插入图片
ctrl+l 插入超链接
Ctrl+B　粗体 
Ctrl+I　斜体
Ctrl+Q　引用 
Ctrl+K　代码块 
#jar包
**包的版本不一致会导致启动失败**
##先贴一个pom
<dependencies>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
        <!--日志log4j配置-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <!--jstl jsp引擎-->
        <dependency>
            <groupId>org.apache.taglibs</groupId>
            <artifactId>taglibs-standard-impl</artifactId>
            <version>1.2.5</version>
        </dependency>

        <dependency>
            <groupId>org.apache.taglibs</groupId>
            <artifactId>taglibs-standard-spec</artifactId>
            <version>1.2.5</version>
        </dependency>

        <dependency>
            <groupId>org.apache.taglibs</groupId>
            <artifactId>taglibs-standard-jstlel</artifactId>
            <version>1.2.5</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp.jstl</groupId>
            <artifactId>jstl-api</artifactId>
            <version>1.2</version>
        </dependency>

        <!--nutz包-->
        <dependency>
            <groupId>org.nutz</groupId>
            <artifactId>nutz</artifactId>
            <version>1.r.60</version>
        </dependency>

        <!--druid监控 web端监控-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.26</version>
            <exclusions>
                <exclusion>
                    <artifactId>jconsole</artifactId>
                    <groupId>com.alibaba</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>tools</artifactId>
                    <groupId>com.alibaba</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--mysql链接-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.40</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
            <version>1.5.2</version>
        </dependency>

        <dependency>
            <groupId>com.liferay</groupId>
            <artifactId>nl.captcha.simplecaptcha</artifactId>
            <version>1.1.1</version>
        </dependency>

        <!--quartz  好像是时间调动-->
        <dependency>
            <groupId>org.nutz</groupId>
            <artifactId>nutz-integration-quartz</artifactId>
            <version>1.r.60.r2</version>
        </dependency>

        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>

        <!--shiro  身份验证、授权、密码学和会话管理-->
        <dependency>
            <groupId>org.nutz</groupId>
            <artifactId>nutz-integration-shiro</artifactId>
            <version>1.r.60.r2</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.12</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.12</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-all</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-email</artifactId>
            <version>1.3.3</version>
        </dependency>

        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.9.2</version>
        </dependency>

        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>class_system</finalName>
        <plugins>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>9.4.3.v20170317</version>
            </plugin>
        </plugins>
    </build>
>[原文链接](http://nutzbook.wendal.net/dev_prepare/dev_prepare.html)
