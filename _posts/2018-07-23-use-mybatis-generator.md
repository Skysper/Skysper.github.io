---
layout: post
tId: 1807001
title: "使用mybatis generator生成代码"
date: 2018-07-23 18:00:00 +0800
categories: spring boot,mybatis,generator
codelang: java
desc: "mybatis generator build工具操作指南"
---
>请首先创建你的springboot项目

#### 在项目添加 mybatis-generator
1.在pom中添加plugin
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.5</version>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
        </dependency>
     </dependencies>
</plugin>
```

2.在resource目录下创建generatorConfig.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="MySqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/demo?serverTimezone=Asia/Shanghai"
                        userId="root"
                        password="123456">
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>

        <javaModelGenerator targetPackage="com.example.demo.model" targetProject="C:\...\project\project-module\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
		<!-- mappers生成在targetProject(即resources)目录下 -->
        <sqlMapGenerator targetPackage="mappers"  targetProject="C:\...\project\project-module\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="com.example.demo.mapper"  targetProject="C:\...\project\project-module\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <table tableName="%">
        </table>
    </context>
</generatorConfiguration>
```

3.创建 maven Run/Debug 配置
![Run/Debug配置](https://i.stack.imgur.com/UtjKJ.png) 

working directory为项目目录（src的父级）,command line如下：
-Dmybatis.generator.overwrite=true mybatis-generator:generate

开发中对生成的mapper以及接口添加了自定义的查询以后，建议将overwrite参数设置为false，否则生成时项目中的实体和mapper类以及mapper xml文件都将被重写，造成属性、函数和自定义查询的丢失

#### 在项目中引用 mybatis
1.在pom中添加mybatis依赖
```xml
<dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.1</version>
</dependency>
```

2.在resource目录下添加mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J" />
    </settings>

    <typeAliases>
        <package name="com.example.demo.model" />
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="" value="" />
            </transactionManager>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/demo" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="com.example.demo.mapper" />
    </mappers>
</configuration>
```

3.Application main类上添加MapperScan
```java
@SpringBootApplication
@MapperScan({"com.example.demo.mapper"})
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

4.添加mapper依赖注入
```java
@Autowired
private DemoMapper demoMapper;

@GetMapping("/demos")
public List<Demo> GetList() {

    try {
        return demo.selectAll();
    } catch (Exception ex) {
        return null;
    }
}
```

参考翻译：[how-to-build-project-by-spring-boot-mybatis-mybatis-generator](https://stackoverflow.com/questions/48976087/how-to-build-project-by-spring-boot-mybatis-mybatis-generator/48976088#48976088)