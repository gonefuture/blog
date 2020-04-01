# Spring源码组件学习-BeanDefinition

## BeanDefinition

### `BeanDefinition`的作用

bean的定义、包装是java bean的基础，在Spring中，`BeanDefinition`就是Bean的定义。
一个`BeanDefinition`描述了一个bean的实例，包括属性值、构造方法参数值和继承自它的类的更多信息。`BeanDefinition`仅仅是一个最简单的接口，主要功能是允许`BeanFactoryPostProcessor`例如`PropertyPlaceHolderConfigure`能够检索并修改属性值和别的bean的元数据。

一个`BeanDefinition`里拥有类名、scope、属性、构造函数参数列表、依赖的bean、是否是单例类、是否是懒加载等，其实就是将Bean的定义信息储存到这个`BeanDefiniton`相应中的属性中，后面对Bean的操作就直接对`BeanDefinition`进行，例如拿到这个`BeanDefinition`后，可以根据里面的类名、构造函数、构造函数参数、使用反射进行对象创建。

### `BeanDefinition`的继承关系

父接口：
`AttributeAccessor`, `BeanMetadataElement`
子接口：
`AnnotatedBeanDefinition`
子类:
`AbstractBeanDefinition`, `AnnotatedGenericBeanDefinition`, `ChildBeanDefinition`, `GenericBeanDefinition`, `RootBeanDefinition`, `ScannedGenericBeanDefinition`

其中，`AttaibuteAccessor`接口定义了最基本的对任意对象的元数据的修改或者获取，主要方法有：

```java
String[] attributeName();
Object getAttribute(String name)
boolean hasAttribute(String name)
Object removeAttribute(String name)
void setAttribute(String name, Object value)
```

`BeanMetadataElement`接口提供了一个getResource()方法，用来传输一个可配置的源对象。

### `BeanDefinition`的抽象类`AbstractBeanDefinition`

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor implements BeanDefinition, Cloneable {

}
```

### `RootBeanDefinition`

一个`RootBeanDefinition`定义表明它是一个可合并的bean definiton:即在Spring beanFactory运行期间，可以返回一个特定的bean。`RootBeanDefinition`可以作为一个重要的通用的bean definition视图。

## 

## BeanDefinitionProProcessor
