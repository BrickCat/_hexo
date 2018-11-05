---
title: SpringCloud系列（一）服务注册与发现
date: 2018-11-05 14:32:49
tags: [Spring Cloud]
categories: [SpringCloud系列]
---
### 写在前面
系列中使用的springboot为2.1.0。

### 服务注册与发现

#### maven依赖

##### 父模块通用依赖
父工程pom.xml
<!-- more -->
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.githink.thinkx</groupId>
    <artifactId>thinkx</artifactId>
    <version>${think.version}</version>
    <packaging>POM</packaging>

    <name>ThinkX</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <think.version>0.0.1-SNAPSHOT</think.version>
        <spring-boot.version>2.1.0.RELEASE</spring-boot.version>
        <spring-cloud.version>Greenwich.M1</spring-cloud.version>
        <spring-platform.version>Cairo-SR5</spring-platform.version>
        <monitor.version>2.0.4</monitor.version>
    </properties>

    <dependencies>
        <!--服务发现-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--监控客户端-->
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
            <version>${monitor.version}</version>
        </dependency>
        <!--断路器依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--Lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!--只是对版本进行管理，不会实际引入jar-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>io.spring.platform</groupId>
                <artifactId>platform-bom</artifactId>
                <version>${spring-platform.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <finalName>${project.name}</finalName>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>

```
##### 子模块依赖

注册中心pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.githink.thinkx</groupId>
    <artifactId>thinkx-eureka</artifactId>
    <version>${think.version}</version>
    <packaging>jar</packaging>

    <name>thinkx-eureka</name>
    <description>服务注册中心</description>

    <parent>
        <groupId>cn.githink.thinkx</groupId>
        <artifactId>thinkx</artifactId>
        <version>${think.version}</version>
    </parent>

    <dependencies>
        <!--服务注册-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!--加入服务认证(密码),需要引入security-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <!--web 模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <!--排除tomcat依赖-->
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--undertow容器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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
##### 服务配置文件
```yaml
# 启动端口号
server:
  port: 8001
eureka:
  client:
    # 不要向注册中心注册自己
    register-with-eureka: false
    # 禁止检索服务
    fetch-registry: false
    # 注册中心地址
    service-url:
      # 用户名:密码
      defaultZone: http://think:think@${eureka.instance.hostname}:${server.port}/eureka/
  instance:
    hostname: localhost
    prefer-ip-address: true
  server:
    # 清理间隔(单位毫秒,默认是60*1000)
    eviction-interval-timer-in-ms: 4000
    # 关闭自我保护机制
    enable-self-preservation: false
    # 自我保护系数
    renewal-percent-threshold: 0.9

# 配置注册中心的安全配置
spring:
  security:
    user:
      name: think
      password: think
# 服务实例名
  application:
    name: thinkx-eureka

# 监控跟踪，获取服务的监控信息
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

##### 开启服务注册
```java
// 开启注册服务
@EnableEurekaServer
@SpringBootApplication
public class ThinkxEurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(ThinkxEurekaApplication.class, args);
    }
}
```

##### 其他配置

自2.0开始security默认开启了CSRF，所以我们为了能够让其他服务成功的注册进来做如下配置
```java
@EnableWebSecurity
@Configuration
public class EurekaAppConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
                //关闭从csrf
                .disable()
                .authorizeRequests()
                //获取监控信息
                .antMatchers("/actuator/**").permitAll()
                //注意：为了可以使用 http://${user}:${password}@${host}:${port}/eureka/ 这种方式登录,所以必须是httpBasic,如果是form方式,不能使用url格式登录
                .anyRequest().authenticated().and().httpBasic();
    }
}
```
