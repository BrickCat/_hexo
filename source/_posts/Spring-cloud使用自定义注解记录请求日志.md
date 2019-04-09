---
title: Spring Cloud使用自定义注解记录请求日志
date: 2019-02-19 14:46:22
tags: [Spring Cloud]
categories: [Spring Cloud]
---
### 写在前面
 我们做程序的，有的时候需要知道用户做了哪些操作，什么时间操作等等，方便我们定位问题、分析用户活动、以及一些其他问题定位。
 为了解决这些问题我们就需要有这样一个功能，记录用户操作并且持久化保存这些数据。这个功能基于Spring Aop开发，自定义注解对系统侵入小。<!-- more -->
 
#### 环境说明
- Spring Cloud Greenwich GA
- Spring Boot 2.1.2
- Rabbitmq 3.7.8
- MySql 5.7 
 
### 几个概念
#### AOP
AOP称为面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待。
- Aspect(切面):通常是一个类，里面可以定义切入点和通知
- JointPoint(连接点):程序执行过程中明确的点，一般是方法的调用
- Advice(通知):AOP在特定的切入点上执行的增强处理，有before,after,afterReturning,afterThrowing,around
- Pointcut(切入点):就是带有通知的连接点，在程序中主要体现为书写切入点表达式
- AOP代理：AOP框架创建的对象，代理就是目标对象的加强。Spring中的AOP代理可以使JDK动态代理，也可以是CGLIB代理，前者基于接口，后者基于子类
[参考](https://www.cnblogs.com/liuruowang/p/5711563.html)

#### 自定义注解开发
它提供了一种安全的类似注释的机制，用来将任何的信息或元数据（metadata）与程序元素（类、方法、成员变量等）进行关联。
为程序的元素（类、方法、成员变量）加上更直观更明了的说明，这些说明信息是与程序的业务逻辑无关，并且供指定的工具或框架使用。
Annontation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。
[参考](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)

#### 消息队列：Rabbitmq
RabbitMQ是一个消息代理：它接受并转发消息。你可以把它当成一个邮局：当你想邮寄信件的时候，你会把信件放在投递箱中，并确信邮递员最终会将信件送到收件人的手里。在这个例子中，RabbitMQ就相当与投递箱、邮局和邮递员。

RabbitMQ与邮局的区别在于：RabbitMQ并不处理纸质信件，而是接受、存储并转发二进制数据---消息。

[参考](https://www.jianshu.com/nb/15067984)
 
 ### 开发功能
 #### 创建消息表
 首先在数据库创建`sys_log`;
 ```sql
DROP TABLE IF EXISTS `sys_log`;
CREATE TABLE `sys_log` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `module` varchar(100) NOT NULL COMMENT '模块名',
  `params` text COMMENT '方法参数',
  `remark` text COMMENT '备注',
  `flag` tinyint(1) NOT NULL COMMENT '是否成功(1成功，0失败)',
  `createTime` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `username` (`username`),
  KEY `createTime` (`createTime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='日志表';
```
#### 实体类
根据数据库创建实体类：`SysLog.java`;
```java
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class SysLog {
    private static final long serialVersionUID = -5398795297842978376L;

    private Long id;
    /** 用户名 */
    private String username;
    /** 模块 */
    private String module;
    /** 参数值 */
    private String params;
    private String remark;
    /** 是否执行成功 */
    private Boolean flag;
    private Date createTime;
}
```
#### 注解类
编写注解类：`Logging.java`;
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
@Inherited  //允许子类继承父类中的注释
public @interface Logging {



    /**
     * 日志模块
     *
     * @return
     */
    String name();

    /**
     * 用户操作名称
     */
    String value() default "";

    /**
     * 记录参数<br>
     * 尽量记录普通参数类型的方法，和能序列化的对象
     *
     * @return
     */
    boolean recordParam() default true;
}
```
