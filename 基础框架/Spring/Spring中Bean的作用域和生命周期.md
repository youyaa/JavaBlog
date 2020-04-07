## Spring中Bean的作用域

Spring为容器中的Bean定义了五种作用域，可通过scope属性指定：

1. **singleton(单例模式)**

默认的作用域。在IOC容器中只会存在一个共享Bean实例，该模式在多线程环境下是不安全的。

2. **prototype(原型模式)**

每次通过Spring容器去取Bean的时候，Spring都会生成一个新的实例。需要注意的是，若是原型模式，Spring在创建了对象后，将不再管理Bean后续的生命周期了。

> 对有状态的Bean使用prototype，无状态的Bean使用singleton。

3. **Request(一次request，一个实例)**

在一次HTTP请求中，容器会返回该Bean的同一个实例。不同的HTTP请求则会产生新的Bean，而且该Bean仅在当前HTTP request内有效，请求结束，该Bean实例也会被销毁。仅适用于web环境。

4. **session**

和request类似。在一次session中返回Bean的同一实例。

5. **global session**

在全局的Session中，Spring都会返回同一个Bean的实例。

## Spring中Bean的生命周期

来看下Bean从创建到销毁可能会做的一些事

## init和destroy

有时我们需要在Bean属性值set好之后和Bean销毁之前做一些事情，比如检查Bean中某个属性是否被正常的设置好值了。Spring框架提供了多种方法让我们可以在Spring Bean的生命周期中执行initialization和pre-destroy方法。

1. **实现InitializingBean和DisposableBean接口**

   这两个接口都只包含一个方法。通过实现InitializingBean接口的afterPropertiesSet()方法可以在Bean属性值设置好之后做一些操作，实现DisposableBean接口的destroy()方法可以在销毁Bean之前做一些操作。

   ```java
   public class Student implements InitializingBean, DisposableBean{
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("执行InitializingBean接口的afterPropertiesSet方法");
       }
       @Override
       public void destroy() throws Exception {
           System.out.println("执行DisposableBean接口的destroy方法");
       }  
     }
        
   ```

2. **在bean的配置文件中指定init-method和destroy-method方法**

   Spring允许我们创建自己的init方法和destroy方法，只要在Bean的配置文件中指定init-method和destroy-method的值就可以在Bean初始化时和销毁之前执行一些操作。

    ```java
   public class Student {
       //通过<bean>的destroy-method属性指定的销毁方法
       public void destroyMethod() throws Exception {
           System.out.println("执行配置的destroy-method");
       }
       //通过<bean>的init-method属性指定的初始化方法
       public void initMethod() throws Exception {
           System.out.println("执行配置的init-method");
       }
   }
    ```

   在配置文件中：

   ```java
   <bean id="student" class="wit.sqf.domain.Student" init-method="initMethod"
           destroy-method="destroyMethod"/>
   ```

   注意：自定义的init-method和destroy-method方法可以抛异常，但是不能有参数。

   这种方式比较推荐，不需要和Spring的类进行耦合。

3. **使用@PostConstruct和@PreDestroy注解**

    除了xml配置的方式，Spring也支持用`@PostConstruct`和 `@PreDestroy`注解来指定init和destroy方法。这两               个注解均在`javax.annotation`包中。

     为了注解可以生效，需要在配置文件中开启注解。

   ```java
   <context:annotation-config/>
   ```

      ```java
   public class Student {
       @PostConstruct
       public void initPostConstruct(){
           System.out.println("执行PostConstruct注解标注的方法");
       }
       @PreDestroy
       public void preDestroy(){
           System.out.println("执行preDestroy注解标注的方法");
       }
   }
      ```



## 实现*Aware接口 在Bean中使用Spring框架的一些对象

 有些时候我们需要在Bean的初始化中使用Spring框架自身的一些对象来执行一些操作，比如获取ServletContext的一些参数，获取ApplicaitionContext中的BeanDefinition的名字，获取Bean在容器中的名字等等。为了让Bean可以获取到框架自身的一些对象，Spring提供了一组名为*Aware的接口。
这些接口均继承于`org.springframework.beans.factory.Aware`标记接口，并提供一个将由Bean实现的set*方法,Spring通过基于setter的依赖注入方式使相应的对象可以被Bean使用。
网上说，这些接口是利用观察者模式实现的，类似于servlet listeners，目前还不明白，不过这也不在本文的讨论范围内。
介绍一些重要的Aware接口：

- ApplicationContextAware: 获得ApplicationContext对象,可以用来获取所有Bean definition的名字。
- BeanFactoryAware:获得BeanFactory对象，可以用来检测Bean的作用域。
- BeanNameAware:获得Bean在配置文件中定义的名字。
- ResourceLoaderAware:获得ResourceLoader对象，可以获得classpath中某个文件。
- ServletContextAware:在一个MVC应用中可以获取ServletContext对象，可以读取context中的参数。
- ServletConfigAware在一个MVC应用中可以获取ServletConfig对象，可以读取config中的参数。

```java
public class GiraffeService implements   ApplicationContextAware,
        ApplicationEventPublisherAware, BeanClassLoaderAware, BeanFactoryAware,
        BeanNameAware, EnvironmentAware, ImportAware, ResourceLoaderAware{
         @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("执行setBeanClassLoader,ClassLoader Name = " + classLoader.getClass().getName());
    }
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("执行setBeanFactory,setBeanFactory:: giraffe bean singleton=" +  beanFactory.isSingleton("giraffeService"));
    }
    @Override
    public void setBeanName(String s) {
        System.out.println("执行setBeanName:: Bean Name defined in context="
                + s);
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("执行setApplicationContext:: Bean Definition Names="
                + Arrays.toString(applicationContext.getBeanDefinitionNames()));
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        System.out.println("执行setApplicationEventPublisher");
    }
    @Override
    public void setEnvironment(Environment environment) {
        System.out.println("执行setEnvironment");
    }
    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        Resource resource = resourceLoader.getResource("classpath:spring-beans.xml");
        System.out.println("执行setResourceLoader:: Resource File Name="
                + resource.getFilename());
    }
    @Override
    public void setImportMetadata(AnnotationMetadata annotationMetadata) {
        System.out.println("执行setImportMetadata");
    }
}
```

## BeanPostProcessor

上面的*Aware接口是针对某个实现这些接口的Bean定制初始化的过程，Spring同样可以针对容器中的所有Bean，或者某些Bean定制初始化过程，只需提供一个实现BeanPostProcessor接口的类即可。

 该接口中包含两个方法，postProcessBeforeInitialization和postProcessAfterInitialization。 postProcessBeforeInitialization方法会在容器中的Bean初始化之前执行， postProcessAfterInitialization方法在容器中的Bean初始化之后执行。

```java
public class CustomerBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行BeanPostProcessor的postProcessBeforeInitialization方法,beanName=" + beanName);
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行BeanPostProcessor的postProcessAfterInitialization方法,beanName=" + beanName);
        return bean;
    }
}
```

要将BeanPostProcessor的Bean像其他Bean一样定义在配置文件中:

```java
 <bean class="wit.sqf.domain.CustomerBeanProcessor"/>
```

## 总结

Bean的生命周期应该是这样的：

- Bean容器找到配置文件中Spring Bean的定义。
- Bean容器利用Java Reflection API创建一个Bean的实例。
- 如果涉及到一些属性值 利用set方法设置一些属性值。
- 如果Bean实现了BeanNameAware接口，调用setBeanName()方法，传入Bean的名字。
- 如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
- 如果Bean实现了BeanFactoryAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
- 与上面的类似，如果实现了其他*Aware接口，就调用相应的方法。
- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization()方法
- 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
- 如果Bean在配置文件中的定义包含`init-method`属性，执行指定的方法。
- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessAfterInitialization()方法
- 当要销毁Bean的时候，如果Bean实现了DisposableBean接口，执行destroy()方法。
- 当要销毁Bean的时候，如果Bean在配置文件中的定义包含`destroy-method`属性，执行指定的方法。

![Spring中Bean的生命周期](img/Spring中Bean的生命周期.png)

