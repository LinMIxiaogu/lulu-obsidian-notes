# Spring 框架面试题

## Q1. Spring 简介

**Spring 的两大核心：**

1. **IOC 容器（控制反转/依赖注入）：** 不需要自己 new 对象，将创建对象的权限交给 IOC 容器管理，直接从容器中取出对象使用，IOC 也帮维护其中对象的依赖关系，降低耦合度，提高开发效率

2. **AOP（面向切面编程）：** 底层使用动态代理，将业务逻辑分割为多个切面，对多个类中共性需求的代码单独封装，配置织入切面位置，提高代码可重用性（典型：日志管理、权限验证）

---

## Q2. IOC 的理解

**IOC（控制反转）：** 不需要自己 new 对象，将创建对象的权限交给 IOC 容器管理，IOC 帮维护对象的依赖关系，降低耦合度

**DI（依赖注入）的三种方式：**
1. ~~接口注入~~（已淘汰）
2. **setter 注入**：Spring 通过无参构造器或静态工厂创建 Bean 对象，再调用 setter 方法注入
3. **构造器注入**：Spring 直接调用有参构造方法实现依赖注入，每个参数就是一个依赖

---

## Q3. AOP 的理解

**AOP（面向切面编程）：** 底层使用动态代理，一种不修改源代码前提下对方法进行功能增强的编程思想

**核心概念：**
- **切面（Aspect）**：横切关注点的模块化
- **切点（Pointcut）**：切面匹配的连接点集合
- **连接点（JoinPoint）**：程序执行过程中的某个特定点
- **增强（Advice）**：切面在特定连接点执行的动作

**典型应用：** 日志管理、权限验证

---

## Q4. Bean 的生命周期

`定义` → `初始化` → `生存期（调用）` → `销毁`

这个过程由 Spring 自己管理，但我们可以自定义初始化方法和销毁方法

- **Bean 定义：** 可以通过构造器和工厂方法创建 Bean 实例
- Spring 默认**单例模式**，应用启动时 Bean 随着容器创建，应用停止时 Bean 随着容器销毁

---

## Q5. @Autowired 和 @Resource 注解的区别

| 特性 | @Autowired | @Resource |
|------|-----------|----------|
| 来源 | Spring 提供 | JDK 提供 |
| 注入方式 | 只能按类型（byType）装配注入 | 默认通过名称（byName）注入，也可以通过类型注入 |
| 指定属性 | 配合 @Qualifier 指定名称 | 可在小括号里指定 `name=xxx, type=xxx` |

---

## Q6. Spring 事务管理

**两种事务管理：**

| 类型 | 说明 | 粒度 |
|------|------|------|
| 编程式事务 | 精确控制事务管理的范围 | 代码块级别 |
| 声明式事务 | 建立在 AOP 上，通过 XML 配置或 `@Transactional` 注解 | 最小粒度只能到方法级别 |

`@Transactional` 注解：方便，但最小粒度只能到方法级别。编程式事务管理粒度更小可以达到代码块级别

---

## Q7. Bean 的作用域

Bean 在 IOC 容器中的**默认作用域为单例（Singleton）**，可通过 `@Scope` 注解修改：

| 作用域 | 说明 |
|-------|------|
| Singleton | 默认，容器中只有一个实例 |
| Prototype | 每次调用 `getBean()` 返回一个新的 Bean |
| Request | 每个 HTTP 请求创建一个新的 Bean |
| Session | 同一个 Session 中的 HTTP 请求共享一个 Bean |
| GlobalSession | 同一个全局 Session 共享一个 Bean |

---

## Q8. BeanFactory 和 FactoryBean 的区别

- **BeanFactory**：是 Factory，即 IOC 容器或对象工厂。是最基础的 IOC 容器，给 IOC 容器提供了一套完整的规范，提供了 `getBean()` 方法从容器中获取 Bean 实例

- **FactoryBean**：是工厂 Bean，是 Spring IOC 容器创建 Bean 的一种形式，是一个接口。它可以产生或修饰对象的工厂 Bean

---

## Q9. MVC 的理解

MVC 是一种设计模式：
- **M（Model）：** 封装数据和对数据的处理，可以和数据库交互
- **V（View）：** 展示给用户看的用户界面
- **C（Controller）：** 处理逻辑，连接模型层和视图层的桥梁，负责将用户请求发送给相应的模型，将模型的改变响应到视图上

**分层优势：** 降低耦合，相互不干扰，便于代码的维护

---

## Q10. Spring MVC 执行流程

```
用户请求 Http Request
    ↓
DispatcherServlet（前端控制器）解析 URI
    ↓
HandlerMapping（处理器映射器）
    ↓ 返回 HandlerExecutionChain 执行链
HandlerAdapter（处理器适配器）执行 Handler
    ↓ 返回 ModelAndView
ViewResolver（视图解析器）解析 View 信息
    ↓ 插入 model 数据，渲染视图
响应给客户端
```

---

## Q11. Spring Boot

### Spring Boot 的理解

Spring Boot 本质就是 Spring，它是帮助使用 Spring 的工具（脚手架框架）

**优点：**
- 快速构建项目
- 对主流开发框架的无配置集成
- 无需依赖外部容器 Servlet
- 提高开发效率
- 可以对运行的项目进行监控

**三大核心功能：**
1. **自动配置**：对 Spring 应用程序中常见功能提供相关配置
2. **起步依赖**：帮助管理项目依赖（Maven/Gradle），聚合常用库
3. **运行监控**：对运行的项目进行监控

### 常用注解

- `@SpringBootApplication`：启动类注解，开启自动配置（复合注解）
  - `@EnableAutoConfiguration`：实现自动装配的核心，基于 classpath 中引入的类和定义的 Bean
  - `@ComponentScan`：扫描指定的 Spring 包和组件
  - `@SpringBootConfiguration`：配置类注解，根据特定条件控制 Bean 的实例化行为
- `@Conditional`：通过设置的条件判断是否加载某个 Bean 对象

### 起步依赖

Spring Boot 将企业开发中常用场景提取出来，做成 `starter`（启动器）：
- starter 整合了该场景下应该有的依赖（约定俗成的默认配置）
- 用户在 Maven 中直接引入 starter 依赖，Spring Boot 扫描并进行相应配置
- 典型例子：`starter-web`，提供 Web 项目开发需要的所有依赖

### 启动流程

1. 通过 `@SpringBootApplication` 注解找到启动类
2. 调用 `run()` 方法
3. 实例化 SpringApplication 对象
4. 再调用对象自身的 `run()` 方法：
   - 获取监听器参数配置
   - 打印 banner 信息
   - 创建初始化容器
   - 监听器进行消息推送

### 自动装配

**定义：** 将第三方组件的 Bean 装载到 Spring 的 IOC 容器中，无需我们实现装配配置

**具体过程：**
1. 依靠 `@EnableAutoConfiguration` 注解
2. 从 `spring.factories` 文件中寻找满足 `@Conditional` 条件的自动配置类
3. 如果找到，实例化这个自动配置类并加载到 Spring 容器中

---

## Q12. MyBatis

### $ 和 # 的区别

| 特性 | ${} | #{} |
|------|-----|-----|
| SQL 类型 | 普通 SQL 语句，执行时参数拼接 | 预编译 SQL 语句，留下占位符 |
| 注入攻击 | 可能有 SQL 注入风险 | 不会有注入攻击，安全 |
| 性能 | 一般 | 预编译语句执行效率高 |

**推荐使用 `#{}` 方式**

### MyBatis 缓存机制

**一级缓存（本地缓存）：**
- 默认启用且不可关闭
- 存在于 `SqlSession` 生命周期中
- 作用域默认是 `SqlSession`
- 增删改语句会清空一级缓存

**二级缓存：**
- 默认不开启，在命名空间（namespace）中
- 采用 LRU（最近最少使用）算法回收缓存
- 是 `SqlSessionFactory` 对象的缓存，由同一个 `SqlSessionFactory` 创建的 SqlSession 共享
- 可跨 `SqlSession` 使用

**缓存使用流程：**
1. 一个会话的查询语句，数据放在一级缓存中
2. 会话关闭，一级缓存保存到二级缓存中
3. 增删改语句，清空一级缓存

---

## Q13. Spring 常用注解

**事务：**
- `@Transactional`：声明式事务

**AOP：**
- `@Aspect`：声明切面类
- `@Pointcut`：定义切点
- `@Before`、`@After`、`@Around`、`@AfterReturning`、`@AfterThrowing`：通知类型

**IoC 容器：**
- `@Service`、`@Repository`、`@Component`：声明 Bean
- `@Autowired`：按类型注入
- `@Resource`：按名称注入（JDK 提供）
- `@Qualifier`：配合 @Autowired 指定 Bean 名称
- `@Bean`：在配置类中声明 Bean
- `@Scope`：设置 Bean 作用域

**Web：**
- `@RestController`：@Controller + @ResponseBody
- `@RequestMapping`、`@GetMapping`、`@PostMapping`：路由映射
- `@RequestBody`：请求体 JSON 绑定
- `@RequestParam`：请求参数绑定
- `@PathVariable`：路径变量绑定
- `@ResponseBody`：返回值写入响应体

---

## Q14. @Transactional 用法

`@Transactional` 可以注解在**类、方法、接口**上：
- **类**：该类的所有 public 方法都配置了事务
- **方法**：方法级别配置，优先级高于类级别（覆盖类的事务信息）

---

## Q15. @Transactional 失效的场景

1. **异常被捕获未重新抛出**：已执行的操作不会回滚
2. **抛出非运行时异常**：`@Transactional` 默认只对 `RuntimeException` 回滚，受检异常不回滚（需配置 `rollbackFor = Exception.class`）
3. **方法中新开启了线程**：Spring 事务通过 ThreadLocal 将数据库连接绑定到当前线程，新线程获取的连接不同，事务失效
4. **方法不是 public**：`@Transactional` 必须注解在 public 方法上
5. **数据库不支持事务**：如使用了 MyISAM 存储引擎
6. **同一个类中的方法调用**：`this.方法()` 调用不会触发代理，事务失效（解决：在类中注入自己，或使用 AopContext.currentProxy()）
7. **类未被 Spring 管理**：没有加 @Service 等注解，Spring 无法生成代理

---

## Q16. Spring 事务传播机制

| 传播行为 | 说明 |
|---------|------|
| `REQUIRED`（默认）| 当前存在事务则加入，不存在则创建新事务 |
| `SUPPORTS` | 当前存在事务则加入，不存在则以非事务方式执行 |
| `MANDATORY` | 当前存在事务则加入，不存在则抛出异常 |
| `REQUIRES_NEW` | 创建新事务，挂起当前事务（两个独立事务）|
| `NOT_SUPPORTED` | 始终以非事务方式执行，如果存在事务则挂起 |
| `NEVER` | 不使用事务，如果存在事务则抛出异常 |
| `NESTED` | 当前存在事务则在嵌套事务中执行，不存在则创建新事务（子事务依赖父事务）|

---

## Q17. AOP 动态代理：JDK 动态代理 vs Cglib

| 特性 | JDK 动态代理 | Cglib 动态代理 |
|------|-------------|---------------|
| 要求 | 被代理对象必须实现接口 | 不要求实现接口 |
| 原理 | Java 反射机制生成接口的匿名实现类 | 生成被代理对象的子类作为代理 |
| 限制 | 只能代理接口方法 | 无法代理 final 类和 final 方法 |
| 性能 | JDK 8+ 性能接近 | 创建代理略慢，执行略快 |

Spring 默认：
- 有接口 → JDK 动态代理
- 无接口 → Cglib 动态代理
- Spring Boot 2.x 起默认使用 Cglib

---

## Q18. Spring Boot、Spring、SpringMVC 的区别

**包含关系：** Spring ⊂ SpringMVC ⊂ Spring Boot（SpringMVC ∈ Spring ∈ Spring Boot）

| 框架 | 定位 | 特点 |
|------|------|------|
| Spring | 轻量级 Java 开发框架 | 核心是 IOC + AOP，模块化设计，需要大量 XML 配置 |
| SpringMVC | Spring 中处理 Web 层请求的模块 | 基于 Servlet 的 MVC 框架，解决 Web 开发问题，配置仍然繁琐 |
| Spring Boot | Spring 的脚手架框架 | 约定优于配置，自动装配，内嵌 Tomcat，简化开发流程 |

---

## Q19. Spring 循环依赖

**什么是循环依赖：** A 依赖 B，B 依赖 A，形成循环

**Spring 如何解决（三级缓存）：**
- **一级缓存（singletonObjects）**：存放完全初始化好的 Bean
- **二级缓存（earlySingletonObjects）**：存放提前曝光的半初始化 Bean
- **三级缓存（singletonFactories）**：存放 Bean 的工厂对象，用于生成代理对象

**流程：** 创建 A → 暴露半成品 A 到三级缓存 → 注入 B → 创建 B → 从三级缓存取到 A → B 完成创建 → A 完成注入

**注意：** 构造器注入的循环依赖无法解决，因为在对象创建阶段就需要另一个对象，无法提前暴露
