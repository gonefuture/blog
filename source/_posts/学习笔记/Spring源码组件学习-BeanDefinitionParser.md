# Spring源码组件学习-BeanDefinitionParser

## Spring自定义xml配置元素，读取xml配置加载bean的过程

xml扩展大概有以下几个步骤：

1. 编写自定义类
2. 编写xml schema来描述自定义元素
3. 编写`NamespaceHandler`实现类编写
4. 编写`BeanDefinitionParser`实现类
5. 把以上组建注册到Spring

## xml的schema约束：

schema约束定义了xml文档的结构、内容和语法，包括元素和属性、关系的结构以及数据类型等等。

1. 所有的标签和属性都需要Schema来定义（schema本身w3c来定义）。
2. 所有的schema文件都需要一个id，即`namespace`，其值为一个url,通常是这个xml的xsd文件地址。
3. `namespace`值由`targetNameSpace`属性来指定。
4. 引入一个`schema`约束，使用属性xmls,属性值即为对应schema文件的命名空间`nameSpace`
5. 如果引入的`schema`非W3c组织定义的，必须指定`schema`文件的位置，`schema`文件的位置由`schemaLocation`来指定。
6. 引入多个`schema`文件需要使用别名。别名形式如下：xmlns:alias。

对于上述配置文件的详细解读

## NamespaceHandler

用于解析自定义命名空间下的所有元素。

Spring提供了默认实现类`NamespaceHandlerSupport`，只需要重写`init()`方法注册每个元素的解析器。

## 注册`handler`和`schema`（将组件串联起来）

为了让Spring在解析xml的时候能够感知到自定义的元素，需要将`handlers`格式和`schemas`格式的文件放入classPath下的`META-INF/`目录中。

1. `srping.handlers`文件包含了`xml schema url`和handler类的映射关系，比如：

    ```handlers
    http\://www.windforce.com/common/event=com.windforce.event.config.EvenrNamespaceHandler
    ```

    上面的冒号表示转义，key部分必须和xsd文件的中的`targetNamespace`值保持一致。

2. spring.schemas文件包含`xml schema url`和xsd文件位置的映射关系。

    ```schemas
    http\://www.windforce.com/common/event/event-1.0.xsd=com/windfore/common/event/config/event-1.0.xsd
    ```

## BeanDefinitionParser

当解析xml配置文件时遇到在`NamespaceHandler`中指定的元素，Spring就会将元素交由相应的的`BeanDefinitionParser`来解析。

`BeanDefinitionParser`是一个接口，只有一个方法`BeanDefinition parse(Element element, ParseContext parseContext)`，返回`BeanDefinition`。
