---
layout:     post
title:      mybatis generator简易教程
subtitle:   mybatis generator简易教程
date:       2018-04-02
author:     dushenzhi
header-img: img/post-bg-debug.png
catalog: true
tags:
    - mysql
    - mybatis
    - java
---
## mybatis generator简易教程
mybatis generator jar包下载地址：
[http://repo1.maven.org/maven2/org/mybatis/generator/mybatis-generator-core/](http://repo1.maven.org/maven2/org/mybatis/generator/mybatis-generator-core/)

官网地址：[http://mybatis.org/generator/](http://mybatis.org/generator/)

github托管地址：[https://github.com/mybatis/generator](https://github.com/mybatis/generator)

在maven中导入mybatis generator：

```xml
<dependency>    
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-core</artifactId>	
	<version>1.3.2</version>
	<type>jar</type>
	<scope>test</scope>
</dependency>
```

## 配置文件的设置

`generatorConfig.xml`示例：
```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <classPathEntry location="/Users/dushenzhi/.m2/repository/mysql/mysql-connector-java/5.1.32/mysql-connector-java-5.1.32.jar"/>
    <context id="MysqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <commentGenerator>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test"
                        userId="root"
                        password="xxxx">
        </jdbcConnection>

        <javaModelGenerator targetPackage="test.model" targetProject="mofang/impl/src/test/java">
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="test.xml"  targetProject="mofang/impl/src/test/resources"/>

        <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="mofang/impl/src/test/java"/>

        <table tableName="%">
            <generatedKey column="id" sqlStatement="Mysql"/>
        </table>
    </context>
</generatorConfiguration>
```
generatorConfig.xml更多使用可以参考：[http://blog.csdn.net/isea533/article/details/42102297](http://blog.csdn.net/isea533/article/details/42102297)以及官方网站

## mybatis generator的使用方法

* 在命令行中执行：
```bash
java -jar mybatis-generator-core-1.3.2.jar -configfile ./generatorConfig.xml -overwrite
```

* 在Java中调用：

```java
    List<String> warnings = new ArrayList<String>();
    boolean overwrite = true;
    File configFile = new File("generatorConfig.xml");
    ConfigurationParser cp = new ConfigurationParser(warnings);
    Configuration config = cp.parseConfiguration(configFile);
    DefaultShellCallback callback = new DefaultShellCallback(overwrite);
    MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
    myBatisGenerator.generate(null);
```

* maven插件的方式：

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
    <configuration>
        <configurationFile>src/main/resources/mybatis-generator/generatorConfig.xml</configurationFile>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
    <executions>
        <execution>
            <id>Generate MyBatis Artifacts</id>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.2</version>
        </dependency>
    </dependencies>
</plugin>
```



## 使用建议

实际项目中我们是使用生成的DAO层的interface `java`文件已经对应的mybatis映射`xml`文件

mybatis generator相关jar、代码和配置文件可以不用放到实际项目中去。

缺点：

1. 每次数据表结构有变化需要重新生成代码，避免出现问题

2. 生成的代码可读性比较差，建议不要修改任何生成代码
3. 如果有不支持的自定义SQL操作，仍需要手动去写，建议自定义数据库表操作不要跟mybatis generator生成的代码放到一起，避免重新生成代码被覆盖掉。



## 其他

mybatis generator生成的代码不支持分页，关于limit分页可参考：[http://www.360doc.com/content/14/0321/08/11298474_362356724.shtml](http://www.360doc.com/content/14/0321/08/11298474_362356724.shtml)

中文帮助文档：[http://mbg.cndocs.tk/index.html](http://mbg.cndocs.tk/index.html)
