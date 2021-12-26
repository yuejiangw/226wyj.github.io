## 声明 Bean 的注解

1. @Component：组件，没有明确的角色，该注解可以让 Spring 容器自动创建被修饰的对象，不需要开发者手动通过 new 关键字来创建
2. @Service：在业务逻辑层使用
3. @Repository：在数据访问层使用
4. @Controller：在展现层使用

使用上述四种注解的作用是等效的，可以根据需要使用。

## 注入 Bean 的注解

1. @Autowired
2. @Inject
3. @Resource

## Java配置注解

1. @Configuration：声明当前类是一个配置类，相当于一个Spring配置的xml文件
2. @Bean：注解在方法上，声明当前方法返回值为一个Bean
3. @ComponentScan：自动扫描包名下所有使用 @Service, @Component, @Repository, 和 @Controller 的类，并注册为 Bean

原则：全局配置使用 Java 配置（如数据库相关配置、MVC相关配置），业务 Bean 的配置使用注解配置（@Service、@Component、@Repository、@Controller）



## 开发流程

1. 编写功能类Bean，用声明Bean的注解修饰
2. 使用功能类的Bean，用注入Bean的注解修饰
3. 编写配置类，用@Configuration修饰，并扫描包名下所有用声明Bean注解修饰的类，将其注册为Bean
4. 运行，使用AnnotationConfigApplicationContext作为Spring的容器，接受输入一个配置类作为参数，之后就可以获得其声明配置的Bean了


## 常用实例和方法
1. ApplicationContext: 一个接口，用来初始化 Spring 容器时作类型声明
2. AnnotationConfigApplicationContext: 实现了 ApplicationContext 的一个实例，代表通过注解来初始化 Spring 容器
3. getBean(): 从容器中获取某个 Bean 对象


## IOC

依赖注入、控制反转



## 面向切面编程

AOP：相当于 OOP 面向对象编程