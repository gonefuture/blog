# SrpingAOP源码分析-AOP

## 概念

切面（Aspect） ：官方的抽象定义为“一个关注点的模块化，这个关注点可能会横切多个对象”。
连接点（Joinpoint） ：程序执行过程中的某一行为。
通知（Advice） ：“切面”对于某个“连接点”所产生的动作。
切入点（Pointcut） ：匹配连接点的断言，在AOP中通知和一个切入点表达式关联。
目标对象（Target Object） ：被一个或者多个切面所通知的对象。
AOP代理（AOP Proxy） 在Spring AOP中有两种代理方式，JDK动态代理和CGLIB代理。

通知（Advice）类型： 

前置通知（Before advice）：在某连接点（JoinPoint）之前执行的通知，但这个通知不能阻止连接点前的执行。ApplicationContext中在`<aop:aspect>`里面使用`<aop:before>`元素进行声明。
后直通知（After advice） ：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。ApplicationContext中在`<aop:aspect>`里面使用`<aop:after>`元素进行声明。
返回后通知（After return advice） ：在某连接点正常完成后执行的通知，不包括抛出异常的情况。ApplicationContext中在`<aop:aspect>`里面使用`<after-returning>`元素进行声明。
环绕通知（Around advice） ：包围一个连接点的通知，类似Web中Servlet规范中的Filter的doFilter方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行。ApplicationContext中在`<aop:aspect>`里面使用`<aop:around>`元素进行声明。
抛出异常后通知（After throwing advice） ： 在方法抛出异常退出时执行的通知。 ApplicationContext中在`<aop:aspect>`里面使用`<aop:after-throwing>`元素进行声明。
---
切入点表达式：如 `executution(* com.spring.service.*.*(...))`

特点
    1. 降低模块之间的耦合度
    2. 使系统容易扩展
    3. 更好的代码复用

流程说明

1. AOP标签的定义解析切点肯定也是从`NamespaceHandlerSupport`的实现类开始解析的，这个实现类就是`AopNamespaceHandler`。
2. 要启用AOP，我们一般会在Spring里面配置`<aop:aspectj-autoproxy/>`，所以在配置文件中在遇到`aspectj-autoproxy`标签的时候我们会采用`AspectJAutoProxyBeanDefinitionParser`解析器
3. 进入`AspectJAutoProxyBeanDefinitionParser`解析器后，调用`AspectJAutoProxyBeanDefinitionParser`已覆盖`BeanDefinitionParser`的parser方法，然后parser方法把请求转交给了`AopNamespaceUtils`的`registerAspectJAnnotationAutoProxyCreatorIfNecessary`去处理
4. 进入`AopNamespaceUtils`的`registerAspectJAnnotationAutoProxyCreatorIfNecessary`方法后，先调用`AopConfigUtils`的`registerAspectJAnnotationAutoProxyCreatorIfNecessary`方法，里面在转发调用给`registerOrEscalateApcAsRequired`，注册或者升级`AnnotationAwareAspectJAutoProxyCreator`类。对于AOP的实现，基本是靠`AnnotationAwareAspectJAutoProxyCreator`去完成的，它可以根据`@point`注解定义的切点来代理相匹配的bean。
5. `AopConfigUtils`的`registerAspectJAnnotationAutoProxyCreatorIfNecessary`方法处理完成之后，接下来会调用`useClassProxyingIfNecessary()` 处理`proxy-target-class`以及`expose-proxy`属性。如果将`proxy-target-class`设置为true的话，那么会强制使用CGLIB代理，否则使用jdk动态代理，`expose-proxy`属性是为了解决有时候目标对象内部的自我调用无法实现切面增强。
6. 最后的调用`registerComponentIfNecessary`方法，注册组建并且通知便于监听器做进一步处理。

创建AOP代理
AOP的核心逻辑是在`AnnotationAwareAspectJAutoProxyCreator`类的`postProcessAfterInitialization()`这个方法，然后接下来是调用`wrapIfNecessary`方法。

```java

```

##