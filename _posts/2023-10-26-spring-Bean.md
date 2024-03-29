```
layout: post
title: bean的加载方式
categories: [spring]
description: spring-bean的加载
keywords: spring,bean
```

# Bean的加载方式

Bean的加载方式

1.XML方式声明bean

2.XML+注解方式声明bean

3.注解方式声明配置类

扩展1——FactoryBean

扩展2——配置类中导入原始的配置文件(系统迁移）

扩展3——proxyBeanMethods

4.使用@Import导入要注入的bean

扩展4——使用@Import注解还可以导入配置类

5.使用上下文对象在容器初始化完毕后注入bean

6.导入实现了ImportSelector接口的类,实现对导入源的编程式处理

bean的加载方式（七）

bean的加载方式（八）



## 1.XML方式声明bean



```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--声明自定义bean-->
    <bean id="bookService" class="com.itheima.service.impl.BookServiceImpl"/>
    <!--声明第三方开发bean-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"/>
</beans>
```



## 2.XML+注解方式声明bean



```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.spring"/>
</beans>
```



```
@Service
public class BookServiceImpl implements BookService {
}
```



（1）使用@Bean定义第三方bean，并将所在类定义为配置类



```
@Configuration
public class DbConfig {
    @Bean
    public DruidDataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        return ds;
} }
```



（2）使用@Component及其衍生注解@Controller 、 @Service、 @Repository定义bean

注意：@Controller 、 @Service、 @Repository作用其实跟Component注解一样，只是给开发人员

看的，让我们能够便于分辨组件的作用。



## 3.注解方式声明配置类



（1）定义配置类



```
@ComponentScan({"com.spring.bean"})
public class SpringConfig3 {
    @Bean
    public DruidDataSource getDataSource(){
        DruidDataSource ds = new DruidDataSource();
        return ds;
} }
```



（2）使用AnnotationConfigApplicationContext加载配置类



```
public class App3 {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig3.class);
        String[] names = ctx.getBeanDefinitionNames();//定义所有bean的名字
        for (String name : names) {
            System.out.println(name);
        }  }  }
```



注意：使用AnnotationConfigApplicationContext加载一个配置类，这个配置类自己也会变成一个bean



## 扩展1——FactoryBean



初始化实现FactoryBean接口的类， 实现对bean加载到容器之前的批处理操作



```
public class BookFactoryBean implements FactoryBean<Book> {
    public Book getObject() throws Exception {
        Book book = new Book();
        // 造book对象之前,可以对book对象进行相关的初始化工作
        return book;//对象
}
    public Class<?> getObjectType() {
        return Book.class;//类型
} }
```



```
@ComponentScan({"com.spring.bean"})
public class SpringConfig3 {
@Bean
public BookFactoryBean book(){
    return new BookFactoryBean();
} }
```



## 扩展2——配置类中导入原始的配置文件(系统迁移）



（1）原始配置文件applicationContext-config.xml



```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context                
       https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- xml方式声明自己开发的bean -->
    <bean id="cat" class="com.spring.bean.Cat"></bean>
    <bean class="com.spring.bean.Dog"></bean>
    <!--    xml方式声明第三方开发的bean-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"></bean>
</beans>
```



（2）使用@ImportResource注解导入



```
@ImportResource("applicationContext-config.xml")
public class SpringConfig2 {
}
```



## 扩展3——proxyBeanMethods



proxyBean就相当于@Configuration配置类最终出来的对象是个代理对象，Methods就是去拿创建的bean



①proxyBeanMethods=true，我们在运行对应的方法的时候，这个方法曾经在spring容器中加载过bean，再调用这个方法，

就是从容器中拿那个bean，而不再重新创建



②proxyBeanMethods=false，使用当前@Configuration配置类的对象，去执行@Bean注解下的方法，每执行一次就重新

创建一个对象



```
Configuration(proxyBeanMethods = false)
public class SpringConfig3 {
    @Bean
    public Book book(){
        return new Book();
} }
public class AppObject {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(Config.class);
        SpringConfig3 config = ctx.getBean("Config", Config.class);
        config.book();
        config.book();
}
```



## 4.使用@Import导入要注入的bean



此形式可以有效解耦，实现无侵入式编程，在spring技术底层及诸多框架的整合中大量使用

（1）被导入的bean无需使用注解声明为bean

public class Dog {

}

（2）导入要注入的bean对应的字节码



```
@Import(Dog.class)
public class SpringConfig4 {
}
```



注意：这种方法生成的bean的名称是全路径类名

扩展4——使用@Import注解还可以导入配置类(配置类内的bean也被加载)



```
@Import({Dog.class,DbConfig.class})
public class SpringConfig {
}
```



```
@Configuration
public class DbConfig {
    @Bean
    public DruidDataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        return ds;
    } }
```



## 5.使用上下文对象在容器初始化完毕后注入bean



```
public class AppImport {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig5.class);
        ctx.register(Cat.class);
        String[] names = ctx.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
} } }
```



6.导入实现了ImportSelector接口的类,实现对导入源的编程式处理



```
@Configuration
@Import(MyImportSelector.class)
public class SpringConfig6 {
}
```



```
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata metadata) {
//        各种条件的判定,判定完毕后,再决定是否装载指定的bean
//       metadata指的是使用import出现的位置
//        判定SpringConfig6是否有Configuration注解
        boolean flag = metadata.hasAnnotation("org.springframework.context.annotation.Configuration");
        if (flag) {
            return new String[]{"com.spring.bean.Dog"};
        }
        return new String[]{"com.spring.bean.Cat"};
    }
}
```



## bean的加载方式（七）



导入实现了ImportBeanDefinitionRegistrar接口的类，通过BeanDefinition的注册器注册实名bean，实现对

容器中bean的裁定，例如对现有bean的覆盖，进而达成不修改源代码的情况下更换实现的效果

和第七种方法的ImportSelector很相似，不过它把bean的管理也开启了



```
@Configuration
@Import(MyRegistrar.class)
public class SpringConfig7 {
}
```



```
public class MyRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Dog.class).getBeanDefinition();
        registry.registerBeanDefinition("dog",beanDefinition);
    }
}
```



## bean的加载方式（八）



导入实现了BeanDefinitionRegistryPostProcessor接口的类，通过BeanDefinition的注册器注册实名bean，

实现对容器中bean的最终裁定



```
public class MyPostProcessor implements BeanDefinitionRegistryPostProcessor {
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
        BeanDefinition beanDefinition = BeanDefinitionBuilder
            .rootBeanDefinition(BookServiceImpl4.class)
            .getBeanDefinition();
            registry.registerBeanDefinition("bookService", beanDefinition);
} }
```