## 声明 Bean 的注解

1. @Component：组件，没有明确的角色
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
3. @ComponentScan：自动扫描包名下所有使用@Service, @Component, @Repository, 和@Controller 的类，并注册为Bean

原则：全局配置使用Java配置（如数据库相关配置、MVC相关配置），业务Bean的配置使用注解配置（@Service、@Component、@Repository、@Controller）



## 开发流程

1. 编写功能类Bean，用声明Bean的注解修饰
2. 使用功能类的Bean，用注入Bean的注解修饰
3. 编写配置类，用@Configuration修饰，并扫描包名下所有用声明Bean注解修饰的类，将其注册为Bean
4. 运行，使用AnnotationConfigApplicationContext作为Spring的容器，接受输入一个配置类作为参数，之后就可以获得其声明配置的Bean了



## IOC

依赖注入、控制反转



## 面向切面编程

AOP：相当于 OOP 面向对象编程