# Spring源码组件学习-BeanPostProcessor

## 定义

`BeanPostProcessor`称为Bean后置处理器，他是Spring中定义的接口，在Spring容器的创建过程中（具体为Bean初始化前端）会回调`BeanPostProcessor`中定义的两个方法。

```java
public interface BeanPostProcessor {
    Object interface postProcessBeforeInitialization(Object bean,String beanName) throws BeanException;

    Object interface postProcessAfterInitialization(Object bean,String beanName) throws BeanException;
}

```

其中`postProcessBeforeInitiallization`方法会在每一个bean对象的初始化方法调用之前回调；
其中`postProcessAfterInitiallization`方法会在每一个bean对象的初始化方法调用之后回调；

自定义`BeanPostProcessor`

查看BeanPostProcessor源码，可以看到它两个方法的参数都相同，其中第一个参数`Object bean`表示当前正在初始化的bean对象。此外两个方法都返回Object类型的实例，返回值既可以是将入参`Object bean`原封不动的返回出去，也可以对当前bean进行包装再返回。

## 执行原理

`BeanPostProcessor`的执行是定义在容器的刷新过程中，容器刷新对象具体的方法为：
`AbstractApplicationContext.refresh()`
在`refresh`方法执行的调用栈中会去调用
`AbstractAutowireCapableBeanFactory.deCreateBean()`方法，该方法节选源码如下:

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    if(mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    invokeInitMethods(beanName, wrappedBean, mbd);

    if(mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialzation(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

看到在调用初始化方法前后会分别调用`applyBeanPostProcessorsBeforeInitializatioan()`和`applyBeanPostProcessorsAfterInitialization()`。
`applyBeanPostProcessorsBeforeInitialization()`方法的源码如下

```java
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName) throws BeansException {
    Object result = existingBean;
    // 获取所有的BeanPostProcessor进行遍历
    for(BeanPostProcessor beanProcessor : getBeanPostProcessor()) {
        result = beanProcessor.postProcessBeanBeforeInitiallization(result, beanName);
        if(result == null) {
            return result;
        }
    }
    return result;
}
```

可以看到其逻辑为遍历得到容器中所有的`BeanPostProcessor`,然后一次执行`postProcessBeforeInitialization`,一旦返回null,就跳出for循环不执行后面的`BeanPostProcessor.postProcessBeforeInitialization()`,也就是说如果返回的
是null那么我们通过getBean方法将得不到目标Bean。
`applyBeanPostProceorsAfterInitialzation()`方法的逻辑与上面一样。

## Spring中常见的几个Bean后置处理器

Spring底层的很多功能特性都是借助`BeanPostProcessor`的子类来实现。

### AppliacationContextAwareProcessor

`AppliacationContextAwareProcessor`作用是当应用程序定义的Bean实现`ApplicationContextAware`接口时注入`ApplicationContext`对象。

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor {
    private final ConfigurableApplicationCotext applicationContext;

    private final StringValueResolver embeddedVakueResolver;

    // Create a new ApplicationContextAwareProcessor for the given context
    public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext ) {
        this.applicationContext = applicationContext;
        this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
    }

    @Override
    public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
        AccessControlContext acc = null;
        // 这里bean是Car,它实现了ApplicationContextAware接口
        if (System.getSecurityManager() != null &&
                (bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
                        bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
                        bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
            invokeAwareInterfaces(bean);
        }

        return bean;
    }

    private void invokeAwareInterfaces(Object bean) {
        if (bean instanceof Aware) {
            if (bean instanceof EnvironmentAware) {
                ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
            }
            if (bean instanceof EmbeddedValueResolverAware) {
                ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
            }
            if (bean instanceof ResourceLoaderAware) {
                ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
            }
            if (bean instanceof ApplicationEventPublisherAware) {
                ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
            }
            if (bean instanceof MessageSourceAware) {
                ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
            }
            if (bean instanceof ApplicationContextAware) {
                // 会执行这里回调实现类重写的setApplicationContext方法，然后将this.applicationContext注入给Car
                ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
            }
        }
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        return bean;
    }

}
```

### InitDestroyAnnotationBeanPostProcessor

`InitDestroyAnnotationBeanPostProcessor后`置处理器是用来处理自定义的初始化方法和销毁方法。Spring中提供了3种自定义初始化和销毁方法：

1. 通过@Bean指定`init-method`和`destroy-method`属性
2. Bean实现InitializingBean（定义初始化逻辑），DisposableBean（定义销毁逻辑）;
3. @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
    @PreDestroy：在容器销毁bean之前通知我们进行清理工作

`InitDestroyAnnotationBeanPostProcessor`的作用就是让第3种方式生效。

InitDestroyAnnotationBeanPostProcessor会在Bean创建的时候通过反射的方式查找包含@PostConstruct和@PreDestroy注解的方法，然后再通过反射执行方法。InitDestroyAnnotationBeanPostProcessor.postProcessBeforeInitialization()的源码

```java
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    // 获取bean的metadata
    LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
    try {
        // 执行@PostConstruct指定的init方法
        metadata.invokeInitMethods(bean, beanName);
    }
    catch (InvocationTargetException ex) {
        throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
    }
    return bean;
}
```

### ConfigurationClassPostProcessor

### PostProcessorRegistrationDelegate

### CommonAnnotationBean

### RequiredAnnotationBeanPostProcessor

### ApplicationListenerDetetor
