# Spring、Spring Boot 与 MyBatis

返回索引：[[00-Java面试题库索引]]

## Spring 核心

- 两大核心：IOC / DI 和 AOP

### IOC / DI

- 对象创建交给容器管理。
- 容器负责依赖关系维护。
- 常见注入方式：
- setter 注入
- 构造器注入

### AOP

- 在不改核心业务代码的前提下进行增强。
- 常见场景：日志、权限、事务。
- 底层常基于动态代理。

## Bean 生命周期

- 实例化
- 属性注入
- 初始化
- 使用
- 销毁

## Bean 作用域

- `singleton`：默认单例
- `prototype`
- `request`
- `session`
- `globalSession`

## @Autowired 和 @Resource

- `@Autowired`：Spring 注解，默认按类型注入。
- `@Resource`：JSR 注解，默认按名称注入，也可指定类型。

## Spring 事务管理

- 编程式事务
- 声明式事务
- 声明式事务通常依赖 AOP 和 `@Transactional`

## BeanFactory 和 FactoryBean

- `BeanFactory`：IOC 容器顶层规范之一。
- `FactoryBean`：一种特殊 Bean，用来创建或定制其他 Bean。

## MVC

- `Model`：数据与业务对象
- `View`：视图展示
- `Controller`：请求调度与业务编排

## Spring MVC 执行流程

- 请求进入 `DispatcherServlet`
- 通过 `HandlerMapping` 找到处理器
- 通过 `HandlerAdapter` 调用处理器
- 返回 `ModelAndView`
- `ViewResolver` 解析视图并渲染响应

## Spring Boot

### Spring Boot 是什么

- 本质上是 Spring 的快速开发脚手架。
- 目标是简化配置、提升项目启动和集成效率。

### 优点

- 自动配置
- 起步依赖
- 内嵌容器
- 便于监控和运维

## 常用注解

- `@SpringBootApplication`
- `@ComponentScan`
- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@Conditional`

## 起步依赖 Starter

- 将某类场景的依赖和约定配置打包。
- 常见如 `spring-boot-starter-web`。

## Spring Boot 启动流程

- 找到启动类
- 创建 `SpringApplication`
- 调用 `run()`
- 初始化环境、容器、监听器
- 完成自动装配并启动应用

## 自动装配

- 核心思想：按条件自动把配置类和 Bean 装入容器。
- 一般依赖类路径中的 starter、自动配置类和条件注解共同完成。

## MyBatis 中 ${} 与 #{}

- `${}`：字符串拼接，存在 SQL 注入风险。
- `#{}`：预编译占位符，更安全，推荐使用。

## MyBatis 缓存

### 一级缓存

- 默认开启。
- 作用域是 `SqlSession`。

### 二级缓存

- 默认不开启。
- 作用域通常是 `namespace`。
- 可跨 `SqlSession` 共享。
