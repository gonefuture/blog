---
title: Netty, SpringBoot, MyBatis搭建游戏服务器 - 1
tags: 
- Netty
- SpringBoot
- MyBatis
categories:
- 游戏开发
---

# Netty, SpringBoot, MyBatis搭建游戏服务器 - 1 

---

## 各框架简介

Netty是步的、事件驱动的网络应用程序框架和工具，,很适合做游戏服务器。

使用SpringBoot可以抛弃传统Spirng框架的繁琐配置和依赖版本管理，更方便地打包与部署。这里我们主要是使用SpringBoot的依赖注入功能。

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。因为我们的数据库可能因为业务需求不断地变化，所以我们可以用MyBaits的逆向工程实时根据数据库的表生成实体类、Mapper接口，Mapper  XML。或者利用注解注解写SQL语句查询。

---

## 目的

我们要使用Netty构建一个游戏服务端，她可以接受客户端的信息，并根据业务逻辑做出响应。
![架构图](https://img-blog.csdn.net/20180919161809161?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNTA0MDE2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Spring负责依赖注入，管理对象（游戏业务对象另外由缓存模块管理），使得项目代码直之间解耦合。

MyBatis负责依据业务需求持久化各类数据。

---

## 依赖配置 


项目使用Maven来管理依赖，依赖结构初步想法是

* mmorpg
	* gameserver
		* mysql
		* 缓存模块
		* 游戏配置管理模块
		* ....
	*  gameclient


下面是当前版本项目的pom文件

总项目定义文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.wan37</groupId>
    <artifactId>mmorpg</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>mmorpg</name>
    <description>Demo project for Spring Boot</description>

    <modules>
        <module>gameserver</module>
        <module>mysql</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-data-redis</artifactId>-->
        <!--</dependency>-->

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>

```

持久化模块定义文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>mmorpg</artifactId>
        <groupId>com.wan37</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mysql</artifactId>
    <packaging>jar</packaging>


    <dependencies>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>


        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.26</version>
        </dependency>


        <!-- 分页插件 -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>4.1.6</version>
        </dependency>

        <!--Json Support-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.43</version>
        </dependency>

        <!--mybatis逆向工程-->
        <!--<dependency>-->
            <!--<groupId>org.mybatis.generator</groupId>-->
            <!--<artifactId>mybatis-generator-core</artifactId>-->
            <!--<version>1.3.2</version>-->
        <!--</dependency>-->

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <!--mybatis插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <configurationFile>src\main\resources\mybatis\generatorConfig.xml</configurationFile>
                    <!--允许移动生成的文件-->
                    <verbose>true</verbose>
                    <!--允许覆盖生成的文件-->
                    <overwrite>true</overwrite>
                </configuration>
                <executions>
                    <execution>
                        <id>GenerateMyBatis Artifacts</id>
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

        </plugins>
    </build>

</project>
```

服务端模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>


    <parent>
        <artifactId>mmorpg</artifactId>
        <groupId>com.wan37</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>



    <artifactId>gameserver</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gameserver</name>
    <description>A game server project for Spring Boot</description>


    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- Netty -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.29.Final</version>
        </dependency>

        <!-- 引用基于MySQL和-->
        <dependency>
            <groupId>com.wan37</groupId>
            <artifactId>mysql</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
---

## Spring和myBatis的配置文件

gameserver模块中的application.yml
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/game?useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

mybatis:
  mapper-locations: classpath*:mapper/*.xml
  check-config-location: true
  type-aliases-package: com.wan37.mysql.pojo.entity
  
debug: true
```
MyBitis配置类（分页助手不知是否能用到）

```
package com.wan37.gameServer.config;

import com.github.pagehelper.PageHelper;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Properties;


/**
 * @author 钱伟健 gonefutre
 * @date 2017/9/12 22:27.
 * @E-mail gonefuture@qq.com
 */

/*
 * 注册MyBatis分页插件PageHelper
 */

@Configuration
@MapperScan(basePackages = {"com.wan37"})
public class MybatisConfig {

    @Bean
    public PageHelper pageHelper() {
        System.out.println("MyBatisConfiguration.pageHelper()");
        PageHelper pageHelper = new PageHelper();
        Properties p = new Properties();
        p.setProperty("offsetAsPageNum", "true");
        p.setProperty("rowBoundsWithCount", "true");
        p.setProperty("reasonable", "true");
        pageHelper.setProperties(p);
        return pageHelper;
    }
}
```



MyBatis逆向工程文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration >
    <!-- 引入配置文件 -->
    <!-- 	<properties resource="db.properties"/> -->
    <!-- 指定数据连接驱动jar地址 -->
    <classPathEntry location="C:\project\mmorpg\mysql\mysql-connector-java-5.1.47.jar"/>
    <!-- 一个数据库一个context -->
    <context id="msql" targetRuntime="MyBatis3" >

        <!-- generate entity时，生成hashcode和equals方法 -->
        <!--<plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin" />-->
        <!-- generate entity时，生成serialVersionUID -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <!-- 这个插件只会增加字符串字段映射到一个JDBC字符的方法 -->
        <!--<plugin type="org.mybatis.generator.plugins.CaseInsensitiveLikePlugin" />-->
        <!-- genenat entity时,生成toString -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />


        <!-- 注释 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/><!-- 是否取消注释 -->
            <property name="supperssDate" value="false"/><!-- 是否生成注释代码时间戳 -->
        </commentGenerator>
        <!-- jdbc连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/game"
                        userId="root"
                        password="123456" />
        <!-- 类型转换 -->
        <javaTypeResolver>
            <!-- 是否使用bigDecimals,false可自动转化以下类型(Long,Integer,Short,ets..) -->
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 生成实体类地址 -->
        <javaModelGenerator targetPackage="com.wan37.mysql.pojo.entity" targetProject="src/main/java">
            <!-- 是否在当前路径下新加一层schema,eg:
            fase路径：com.sky.ssm.po ； true路径：com.sky.ssm.po.[shemaName]
             -->
            <property name="enableSubPackages" value="true"/>
            <!-- 是否针对string类型的字段在set的时候进行trim调用 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- 生成mapxml文件 -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <!-- 是否在当前路径下新加一层schema,eg:
            fase路径：com.sky.ssm.mapper ； true路径：com.sky.ssm.mapper.[shemaName]
             -->
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 生成mapxml对应client，也就是接口dao -->
        <javaClientGenerator targetPackage="com.wan37.mysql.pojo.mapper" targetProject="src/main/java"
                             type="XMLMAPPER" >
            <!-- 是否在当前路径下新加一层schema,eg:
            fase路径：com.sky.ssm.mapper ； true路径：com.sky.ssm.mapper.[shemaName]
             -->
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 配置表信息 -->
        <!-- schema即为数据库名； tableName为对应的数据库表 ；domainObjectName是要生成的实体类 ；enable*ByExample是否生成 example类 -->
        <!-- <table schema="store" tableName="user"
            domainObjectName="User" enableCountByExample="true"
            enableDeleteByExample="true" enableSelectByExample="true"
            enableUpdateByExample="ture">
            忽略列，不生成bean字段
             <ignoreColumn column="FRED"/>
                 指定列的java数据类型
           <columnOverride column="PRICE" javaType="double" />
          </table> -->
        <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
        <table tableName="game_role"/>
        <table tableName="player"/>
        <table tableName="player_roles"/>

    </context>
</generatorConfiguration>
```

##  整合Netty

我使用的办法是把Netty服务的Handler，启动类都都变成由Spring管理的类。通过在类定义上加@Component注解，SpringBoot启动时会自动扫描注解并将Bean注入到容器中。使用   @PostConstruct修饰启动方法来启动Netty服务。

```java
import com.wan37.gameServer.server.handler.ServerHandler;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;


/**
 * @author gonefuture  gonefuture@qq.com
 * time 2018/9/10 11:36
 * @version 1.00
 * Description: 游戏服务端
 */
@Slf4j
@Component
@ChannelHandler.Sharable
public class GameServer {

    @Resource
    private ServerHandler serverHandler;

    //绑定端口
    private void bind(int port) throws Exception {
        // 逻辑线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        // 工作线程
        EventLoopGroup workGroup = new NioEventLoopGroup();

        ServerBootstrap bootstrap = new ServerBootstrap(); // 启动器
        bootstrap.group(bossGroup, workGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024) // 最大客户端连接数为1024
                //是否启用心跳保活机制
                .childOption(ChannelOption.SO_KEEPALIVE, true)
        .childHandler(new ChannelInitializer<SocketChannel>() {

            @Override
            protected void initChannel(SocketChannel ch) throws Exception {
                // 这里添加业务处理handler
                ch.pipeline().addLast( serverHandler);
            }
        });

        try {
            ChannelFuture future = bootstrap.bind(port).sync();
            if (future.isSuccess()) {
                System.out.println("Server starts success at port:"+port);
            }
            future.channel().closeFuture().sync();
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 8000;
        new GameServer().bind(port);
    }


    @PostConstruct
    public void start() {
        int port = 8000;
        try {
          bind(port);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}

```

## 期间遇到的各种问题

整合的时候遇到过很多问题

1. 整合Netty时，由于使用new关键字创建Handler实例，导致在Handle中装配的服务实例为空，不能使用。
2. MyBatis配置类写在了mysql模块而没有写在gameserver模块,导致mapper文件扫描不到。
3. 逆向工程时产生了同名的mapper xml文件未及时删除，导致出现mapper result map 重复的错误。

## 结语

当前只整合了持久化模块，接下来还要编写请求分发的部分和整合缓存模块。