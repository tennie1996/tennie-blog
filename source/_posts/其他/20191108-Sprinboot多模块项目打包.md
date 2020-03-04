---
title: Springboot多模块Maven项目打包
date: 2019-11-08 22:27:28
urlname: 2019110801
categories: Java
tags:
  - Java
author: foochane
toc: true
mathjax: false
top: false
cover: false
---


## 1 项目结构

```
─ web-api
    ├─common
    ├─controller
    ├─dao
    ├─model
    └─service
```

 web-api 是父模块，其下面有五个模块：

pom文件中的模块设置为：
```xml
    <modules>
        <module>common</module>
        <module>dao</module>
        <module>service</module>
        <module>model</module>
        <module>controller</module>
    </modules>
```

## 2 打成jar包运行
### 2.1 修改pom文件配置

#### 2.1.1 设置packaging属性
父模块web-api中的pom.xml文件中的packaging设为 pom
```xml
<packaging>pom</packaging> 

```

其他模块中的pom.xml文件中的packaging设为全部设置为 jar

```xml
<packaging>jar</packaging> 

```

#### 2.2.2  设置build属性

只需要打包 web-api和controller模块，所以只需要在这两个模块下添加build熟悉

web-api下的pom.xml配置：

```xml
    <!--指定使用maven打包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <skipTests>true</skipTests>  <!--默认关掉单元测试 -->
                </configuration>
            </plugin>
        </plugins>
    </build>
```

controller 模块下pom.xml文件配置:

**注意修改mainClass**

```xml
	<!--spring boot打包的话需要指定一个唯一的入门-->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<!-- 指定该Main Class为全局的唯一入口 -->
					<mainClass>ControllerApplication</mainClass>
					<layout>ZIP</layout>
				</configuration>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

### 2.3 打包

在 IDEA 中点击 package 打包

### 2.4 运行

```bash
java -jar xxxx.jar

```

## 3 打成war包运行

### 3.1 修改pom文件

打成war运行在tomcat中，所以打包时要排除tomcat

所以在依赖中添加
```xml
		<!--在IDE中运行时使用内置tomcat，打包时排除tomcat-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
```

然后报controller下面的packaging改成jar

```xml
<packaging>war</packaging>

``` 

其他的不用修改，前面一样

### 3.2 修改controller

修改controller，继承SpringBootServletInitializer 重写 SpringApplicationBuilder

```java
package com.uestc;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 实现 WebMvcConfigurer 重写addCorsMappings方法解决前后端分离时的跨域问题
 * 继承SpringBootServletInitializer 重写 SpringApplicationBuilder 可打包成war包放在tomcat下运行
 */
@SpringBootApplication
@EnableTransactionManagement//开启事务管理
@MapperScan("com.uestc.dao")
public class WebApiApplication extends SpringBootServletInitializer implements WebMvcConfigurer {

    public static void main(String[] args) {
        SpringApplication.run(WebApiApplication.class, args);
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/**")
                .allowCredentials(true)
                .allowedHeaders("*")
                .allowedOrigins("*")
                .allowedMethods("*");

    }

    	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application)
	{
		return application.sources(WebApiApplication.class);
	}

}
```

### 3.3 打包运行

使用maven工具打包后放在，tomcat的webapps目录下，启动tomcat即可运行。