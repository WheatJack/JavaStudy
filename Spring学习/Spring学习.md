# Spring学习



1、了解预研究的组件（类）基本使用



2、用单元测试研究组件的特性



3、试着自己实现类似功能



4、最后在深入研究源码





## 1、容器与Bean

#### 1、BeanFactory能做哪些事情

ConfigurationApplicationContext与BeanFactory的关系

![image-20220726210003693](https://tva1.sinaimg.cn/large/e6c9d24egy1h4kmmn6alxj21ww0hkdl6.jpg)

```java
/*
 * 1、到底什么是BeanFactory
 *  - 它是ApplicationContext的父接口
 *  - 它才是Spring的核心容器，主要的ApplicationContext实现都【组合】了它的功能
 */
```

BeanFactory主要的实现类：DefaultListableBeanFactory 

![image-20220726210826488](https://tva1.sinaimg.cn/large/e6c9d24egy1h4kmv8zurej21f90u0jy6.jpg)

单例Bean注册类 DefaultListableBeanFactory 

```java
// private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
// 获取DefaultSingletonBeanRegister


    //拿到所有的私有成员变量
        Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
        singletonObjects.setAccessible(true);
        ConfigurableListableBeanFactory beanFactory = configurableApplicationContext.getBeanFactory();
        // 获取beanFactory的属性
        Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
        map.entrySet().stream().filter(e->e.getKey().startsWith("component")).forEach(e-> System.out.println(e.getKey() + "==" + e.getValue()));

```

```java
/*
 * 2、BeanFactory能干点啥
 *  - 表面上只有getBean
 *  - 实际上 控制反转、基本的依赖注入、直至Bean的生命周期的各种功能，都由它的实现类提供
 */
```



#### 2、ApplicationContext有哪些扩展功能

```java
/*
 * ConfigurableApplicationContext比 BeanFactory多点啥？？？
 *  - MessageSource 国际化处理
 *  - ResourcePatternResolver  通配符 资源问题
 *  - EnvironmentCapable 环境变量
 *  - ApplicationEventPublisher 发布事件 事件解耦
 *  - 完成用户注册与发送短信之间的解耦 用事件方式和AOP方式分布实现
 */
```

**国际化处理**

配置对应的Resources文件

![image-20220727213802236](https://tva1.sinaimg.cn/large/e6c9d24egy1h4ltcelffgj20ue0a4wf7.jpg)

```java
// 获取对应的翻译
System.out.println(configurableApplicationContext.getMessage("Hi", null, Locale.CHINA));
System.out.println(configurableApplicationContext.getMessage("Hi", null, Locale.JAPAN));
System.out.println(configurableApplicationContext.getMessage("Hi", null, Locale.ENGLISH));
```



**根据通配符获取资源**

```java
      // 根据通配符获取资源
//        Resource[] resources = configurableApplicationContext.getResources("application.properties");
        Resource[] resources = configurableApplicationContext.getResources("classpath*:META-INF/spring.factories");
        for (Resource resource : resources) {
            System.out.println(resource);
        }
```



获取各种环境变量

```java
System.out.println(configurableApplicationContext.getEnvironment().getProperty("java_home"));
System.out.println(configurableApplicationContext.getEnvironment().getProperty("server.port"));
```



#### 3、事件解耦

创建对应的事件

```java
public class UserRegisterEvent extends ApplicationEvent {
    public UserRegisterEvent(Object source) {
        super(source);
    }
}
```

发布事件

```java
@Component
public class Component1 {
    @Resource
    private ApplicationEventPublisher eventPublisher;

    public void register(){
        System.err.println("来活了");
        eventPublisher.publishEvent(new UserRegisterEvent("qqqq"));
    }
}
```

监听事件

```java
@Component
public class Component2 {


    @EventListener
    public void getMessage(UserRegisterEvent userRegisterEvent){
        Object source = userRegisterEvent.getSource();
        System.out.println("传入的参数："+source);
        System.out.println("整个的参数："+userRegisterEvent);
    }
}
```



## 2、容器的实现

#### 1、BeanFactory实现的特点





#### 2、ApplicationContext的常见实现和用法

**一共四种常见的实现：**

##### 1、基于classpath的xml创建Bean

```java
public class A02Application {

    public static void main(String[] args) {
       testClassPathXmlApplicationContext();
    }

    // 较为经典的容器，基于classpath 下的xml 格式的配置文件来创建bean
    private static void testClassPathXmlApplicationContext() {
        ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("b01.xml");

        for (String beanDefinitionName : classPathXmlApplicationContext.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
        System.out.println(classPathXmlApplicationContext.getBean(Bean2.class).getBean1());
    }


    static class Bean1 {
        public Bean1() {
            System.out.println("bean1 的构造方法");
        }
    }

    static class Bean2 {

        private Bean1 bean1;

        public Bean2() {
            System.out.println("bean2 的构造方法");
        }

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }

}
```

对应的xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd


    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <bean name="bean1" class="com.example.springstudy.application.context.A02Application.Bean1"/>
    <bean name="bean2" class="com.example.springstudy.application.context.A02Application.Bean2">
        <property name="bean1" ref="bean1"/>
    </bean>

    <!--    加载默认的配置后处理器-->
    <context:annotation-config/>

</beans>
```

打印输出：

```java
21:44:57.609 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@39ed3c8d
21:44:57.960 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 7 bean definitions from class path resource [b01.xml]
21:44:58.019 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
21:44:58.095 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
21:44:58.097 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
21:44:58.102 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
21:44:58.104 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
21:44:58.118 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean1'
bean1 的构造方法
21:44:58.149 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean2'
bean2 的构造方法
bean1
bean2
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
com.example.springstudy.application.context.A02Application$Bean1@70b0b186
```



##### 2、基于基于磁盘路径下的xml创建bean

```java
public class A02Application {

    public static void main(String[] args) {
        testFileSystemXmlApplicationContext();
    }


    // 基于磁盘路径下的xml 配置文件来创建bean
    private static void testFileSystemXmlApplicationContext() {
        FileSystemXmlApplicationContext fileSystemXmlApplicationContext = new FileSystemXmlApplicationContext("/src/main/resources/b01.xml");

        for (String beanDefinitionName : fileSystemXmlApplicationContext.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
        System.out.println(fileSystemXmlApplicationContext.getBean(Bean2.class).getBean1());
    }


    static class Bean1 {
        public Bean1() {
            System.out.println("bean1 的构造方法");
        }
    }

    static class Bean2 {

        private Bean1 bean1;

        public Bean2() {
            System.out.println("bean2 的构造方法");
        }

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }

}
```

输出：

```java
21:46:56.877 [main] DEBUG org.springframework.context.support.FileSystemXmlApplicationContext - Refreshing org.springframework.context.support.FileSystemXmlApplicationContext@39ed3c8d
21:46:57.201 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 7 bean definitions from file [/Users/gaoshang/IdeaProjects/SpringStudy/src/main/resources/b01.xml]
21:46:57.237 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
21:46:57.281 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
21:46:57.283 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
21:46:57.285 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
21:46:57.286 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
21:46:57.294 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean1'
bean1 的构造方法
21:46:57.309 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean2'
bean2 的构造方法
bean1
bean2
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
com.example.springstudy.application.context.A02Application$Bean1@6b1274d2
```



**第一、二种实现总结：**

```java
public class A02Application {

    public static void main(String[] args) {
        // 如何实现--具体实现 还是使用默认的构造Factory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        System.out.println("读取之前的...");
        for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
        System.out.println("读取之后的,,,");
      // 添加对应的后置处理器
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
//        xmlBeanDefinitionReader.loadBeanDefinitions(new ClassPathResource("b01.xml"));
   
        xmlBeanDefinitionReader.loadBeanDefinitions(new FileSystemResource("/Users/gaoshang/IdeaProjects/SpringStudy/src/main/resources/b01.xml"));
        for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }

    static class Bean1 {
        public Bean1() {
            System.out.println("bean1 的构造方法");
        }
    }

    static class Bean2 {

        private Bean1 bean1;

        public Bean2() {
            System.out.println("bean2 的构造方法");
        }

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }

}
```

打印输出：

```java
读取之前的...
读取之后的,,,
21:52:22.674 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 7 bean definitions from file [/Users/gaoshang/IdeaProjects/SpringStudy/src/main/resources/b01.xml]
bean1
bean2
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
```



##### 3、基于Java 配置类来创建

```java
public class A02Application {

    public static void main(String[] args) {

     testAnnotationConfigApplicationContext();

    }

    // 较为经典的容器 基于Java 配置类来创建
    private static void testAnnotationConfigApplicationContext() {
        AnnotationConfigApplicationContext annotationConfigWebApplicationContext = new AnnotationConfigApplicationContext(Config.class);

        for (String beanDefinitionName : annotationConfigWebApplicationContext.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
        System.out.println(annotationConfigWebApplicationContext.getBean(Bean2.class).getBean1());
    }



    @Configuration
    static class Config {
        @Bean
        Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        Bean2 bean2(Bean1 bean1) {
            Bean2 bean2 = new Bean2();
            bean2.setBean1(bean1);
            return bean2;
        }
    }


    static class Bean1 {
        public Bean1() {
            System.out.println("bean1 的构造方法");
        }
    }

    static class Bean2 {

        private Bean1 bean1;

        public Bean2() {
            System.out.println("bean2 的构造方法");
        }

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }

}
```



打印输出：

```java
21:53:47.938 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@7637f22
21:53:47.964 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
21:53:48.264 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
21:53:48.267 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
21:53:48.269 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
21:53:48.275 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
21:53:48.297 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'a02Application.Config'
21:53:48.311 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean1'
bean1 的构造方法
21:53:48.336 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'bean2'
21:53:48.346 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Autowiring by type from bean name 'bean2' via factory method to bean named 'bean1'
bean2 的构造方法
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
a02Application.Config
bean1
bean2
com.example.springstudy.application.context.A02Application$Bean1@3967e60c
```



##### 4、基于java配置类来创建，用于web环境

> **内嵌Tomcat的使用**

```java

public class A02Application {

    public static void main(String[] args) {
        testAnnotationConfigServletWebServerApplicationContext();
    }

 

    // 较为经典的容器 基于java配置类来创建，用于web环境
    private static void testAnnotationConfigServletWebServerApplicationContext() {
        AnnotationConfigServletWebApplicationContext annotationConfigServletWebApplicationContext = new AnnotationConfigServletWebApplicationContext(WebConfig.class);
        for (String beanDefinitionName : annotationConfigServletWebApplicationContext.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }

    //    内嵌Tomcat如何工作的
    @Configuration
    static class WebConfig {

        // 对应的运行服务器 环境等
        @Bean
        public ServletWebServerFactory servletWebServerFactory() {
            return new TomcatServletWebServerFactory();
        }

        // 对应的前置servlet  入口都是dispatcherServlet
        @Bean
        public DispatcherServlet dispatcherServlet() {
            return new DispatcherServlet();
        }

        // 把对应的环境和servlet 关联起来
        @Bean
        public DispatcherServletRegistrationBean dispatcherServletRegistrationBean(DispatcherServlet dispatcherServlet) {
            return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        }

        @Bean("/hello")
        public Controller controller() {
            return (request, response) -> {
                response.getWriter().println("hello");
                return null;
            };
        }
    }




    static class Bean1 {
        public Bean1() {
            System.out.println("bean1 的构造方法");
        }
    }

    static class Bean2 {

        private Bean1 bean1;

        public Bean2() {
            System.out.println("bean2 的构造方法");
        }

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }

}
```

打印输出：

```java
21:57:23.729 [main] DEBUG org.springframework.boot.web.servlet.context.AnnotationConfigServletWebApplicationContext - Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebApplicationContext@69222c14
21:57:23.828 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
21:57:24.052 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
21:57:24.054 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
21:57:24.055 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
21:57:24.058 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
21:57:24.066 [main] DEBUG org.springframework.ui.context.support.UiApplicationContextUtils - Unable to locate ThemeSource with name 'themeSource': using default [org.springframework.ui.context.support.ResourceBundleThemeSource@6dbb137d]
21:57:24.068 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'a02Application.WebConfig'
21:57:24.080 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'servletWebServerFactory'
21:57:24.252 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'dispatcherServlet'
21:57:24.280 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'dispatcherServletRegistrationBean'
21:57:24.285 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Autowiring by type from bean name 'dispatcherServletRegistrationBean' via factory method to bean named 'dispatcherServlet'
21:57:24.294 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean '/hello'
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
a02Application.WebConfig
servletWebServerFactory
dispatcherServlet
dispatcherServletRegistrationBean
/hello
```



#### 3、内嵌容器、注册DispatcherServlet







Go to implentation ctrl +alb +b

File Structure command +F12

Jump  to Source F4

find all structure command+ F12





## 2、AOP





## 3、Web MVC





## 4、Spring Boot





## 5、其他

