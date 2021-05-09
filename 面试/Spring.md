### Spring

- AOP 原理（JDK 动态代理，CGLIB 动态代理）和 IOC 原理
- Spring Bean 生命周期
- SpringMVC 原理
- SpringBoot 常用注解



### BeanPostProcessor 接口

https://cloud.tencent.com/developer/article/1409315

该接口我们也叫后置处理器，它里面有两个方法。

```java
// 在bean被实例化和属性注入之后，调用。在自定义初始化方法（init-method指定的方法）之前调用。
default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
// 在自定义初始化方法执行之后执行的。
    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean class="com.dpb.pojo.User" id="user" init-method="start">
		<property name="name" value="波波烤鸭" />
	</bean>
	
	<!-- 注册处理器 -->
	<bean class="com.dpb.processor.MyBeanPostProcessor"></bean>
</beans>
```

那么也可以定义多个后置处理器，调用顺序可以通过实现Ordered来指定。还有一点需要注意的是，在spring中有两种容器。BeanFactory需要手动注册，而ApplicationContext则是自动注册。

### BeanFactory和ApplicationContext有什么区别？

BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能。

BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

几乎所有场合都可以直接使用ApplicationContext而不是底层的BeanFactory。

### Spring Bean 在容器的生命周期是什么样的？

1. 实例化 Bean 对象
   - Spring 容器根据配置中的 Bean Definition(定义)中**实例化** Bean 对象。Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。
   - Spring 使用依赖注入**填充**所有属性，如 Bean 中所定义的配置。
2. Aware 相关的属性，注入到 Bean 对象
   - 如果 Bean 实现 **BeanNameAware** 接口，则工厂通过传递 Bean 的 beanName 来调用 `#setBeanName(String name)` 方法。
   - 如果 Bean 实现 **BeanFactoryAware** 接口，工厂通过传递自身的实例来调用 `#setBeanFactory(BeanFactory beanFactory)` 方法。
3. 调用相应的方法，进一步初始化 Bean 对象
   - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则调用 `#preProcessBeforeInitialization(Object bean, String beanName)` 方法。
   - 如果 Bean 实现 **InitializingBean** 接口，则会调用 `#afterPropertiesSet()` 方法。
   - 如果为 Bean 指定了 **init** 方法（例如 `<bean />` 的 `init-method` 属性），那么将调用该方法。
   - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则将调用 `#postProcessAfterInitialization(Object bean, String beanName)` 方法。
4. **销毁**流程如下：
   - 如果 Bean 实现 **DisposableBean** 接口，当 spring 容器关闭时，会调用 `#destroy()` 方法。
   - 如果为 bean 指定了 **destroy** 方法（例如 `<bean />` 的 `destroy-method` 属性），那么将调用该方法。

### AOP

在aop中，概念多。要清楚区分每一个概念。

在spring中，提供了aop的功能。作用就是面向切面编程。对于目标类中的目标方法进行增强。做法就是将增强代码织入原目标方法的前后来实现的。那么，具体的实现有几个可以方面可以考虑。一个是在编译期的织入，这个需要有编译器的支持，第二种是在类装载的时候的织入，这个也需要有特定的类加载器支持。第三种就是运行期为目标类添加增强(Advice)生成子类的方式。那么，spring使用的是第三种。运行期的织入也分为jdk的动态代理和CGLIB的动态代理。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 `final` ，那么它是无法使用 CGLIB 做动态代理的。

> #### advice(增强)

所谓的增强，就是你要添加的代码。直接举个例子，比如说你要做一个日志记录的操作。那么用aop来做。此时，这些添加记录日志的代码就是增强。就是在源代码前后添加的代码，称为增强。

> #### 连接点(join point)

任何一个方法都可以是连接点。所谓的连接点有点像动态代理里面的真实方法。就是原方法。它是被切入的。

> #### 切点(point cut)

切点就是来描述在哪些连接点切入的。在哪个连接点切入，来增强代码。

> ####  目标对象(Target)

织入 advice 的目标对象. 目标对象也被称为 `advised object`.因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)
注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类.

在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象.

### spring aop 和 AspectJ 

Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入.

- 代理方式不同
  - Spring AOP 基于动态代理方式实现。
  - AspectJ AOP 基于静态代理方式实现。
- PointCut 支持力度不同
  - Spring AOP **仅**支持方法级别的 PointCut 。
  - AspectJ AOP 提供了完全的 AOP 支持，它还支持属性级别的 PointCut 。

### spring 事务

https://zhuanlan.zhihu.com/p/148504094

spring事务是在数据库事务的基础上进行封装扩展，其主要特性如下:

- 支持原有的数据事务的隔离级别
- 加入了事务传播的概念，提供多个事务的合并和隔离的功能
- 提供声明式事务，让业务代码与事务分离，事务更易用

spring 把七种隔离级别都定义在了一个枚举类中。Propagation类中。这个 Propagation 必须要配合@Transactional 来使用。

###### 常用的事务传播机制：

- PROPAGATION_REQUIRED

>  如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中这个是默认传播机制。 

- PROPAGATION_NOT_SUPPORTED

>  以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。可以用于发送提示信息，站内信，邮件提示灯。不属于并且不应该影响主体业务逻辑，即时发送失败也不应该对主题业务逻辑回滚。 

- PROPAGATION_REQUIRED_NEW

>  新建一个事务，如果存在当前事务，则将事务挂起。总是新启一个事务，这个传播机制适用于不受父类方法事务影响的操作，比如某些业务场景下需要记录业务日志，用于异步反查，那么不管主体业务逻辑是否完成，日志都需要记录下来，不能因为主体业务逻辑报错而丢失日志。

声明式事务是通过aop来实现的。而当调用同一个类的方法时，是不会走代理逻辑的，自然事务的配置也会失效。**同一个Service类中的方法相互调用需要使用注入的对象来调用，不要直接使用this.方法名来调用，this.方法名调用是对象内部方法调用，不会通过Spring代理，也就是事务不会起作用。**