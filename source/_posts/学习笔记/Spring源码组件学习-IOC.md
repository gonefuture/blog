# SpringIOC源码分析

## IOC初始化

1. XmlBeanFactory的整个流程
2. FileSystemXmlApplicationContext的IOC容器流程。
3. AbstractApplicationContext子类的refresh函数载入Bean定义过程
4. AbstractApplicationContext子类的refrshBeanFactory()方法
5. AbstractRefreshableApplicationContext子类的loadBeandefinitions方法
6. AbstractBeanDefinitionReader读取Bean定义资源
7. 资源加载器获取要读入的资源
8. XmlBeanDefinitionReader加载Bean定义资源
9. DecumentLoader将Bean定义资源转换为Decument对象
10. XmlBeanDefinitionReader加载Bean定义资源文件
11. DefaultBeanDefinitionDocumentReader对定Bean定义的Decument对象解析
12. BeanDefinitionParserDelegate解析Bean定义资源文件中的`<Bean>`元素。
13. BeanDefinitionParserDelegate解析`<property>`元素。
14. 解析`<property>`元素的子元素
15. 解析`<list>`子元素
16. 解析过后的BeanDefinition在IoC容器中注册：
17. DefaultListableBeanFactory向Ioc容器注册解析后的BeanDefinition

---

## IOC体系

    Spring Bean的创建是典型的工厂模式，这一系列的Bean工厂，也即IOC容器为开发者管理对象间的依赖关系提供了很多便利和基础服务，在Spring中有许多次的IOC容器的实现供用户选择和使用，其相互关系如下：

### BeanFactory

    SpringFactory定义了IOC容器的最基本形式，并提供了IOC容器应遵守的最基本的接口，也就是Spring代码中，BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，都是附加了某种功能的实现。

![BeanFactory](https://javadoop.com/blogimages/spring-context/2.png)

### ApplicationContext

![Application](https://javadoop.com/blogimages/spring-context/2.png))

1. `ApplicationContext`继承了`ListableBeanFactory`，这个 `Listable`的意思就是，通过这个接口，可以获取多个`Bean`，最顶层 `BeanFactory`接口的方法都是获取单个Bean的。
2. `ApplicationContext`继承了 HierarchicalBeanFactory，Hierarchical 单词本身已经能说明问题了，也就是说我们可以在应用中起多个 BeanFactory，然后可以将各个 BeanFactory 设置为父子关系。
AutowireCapableBeanFactory 这个名字中的 Autowire 大家都非常熟悉，它就是用来自动装配 Bean 用的，但是仔细看上图，ApplicationContext 并没有继承它，不过不用担心，不使用继承，不代表不可以使用组合，如果你看到 ApplicationContext 接口定义中的最后一个方法 getAutowireCapableBeanFactory() 就知道了。
ConfigurableListableBeanFactory 也是一个特殊的接口，看图，特殊之处在于它继承了第二层所有的三个接口，而 ApplicationContext 没有。这点之后会用到。
请先不用花时间在其他的接口和类上，先理解我说的这几点就可以了。

FileSystemXmlApplicationContext 的构造函数需要一个 xml 配置文件在系统中的路径，其他和 ClassPathXmlApplicationContext 基本上一样。

AnnotationConfigApplicationContext 是基于注解来使用的，它不需要配置文件，采用 java 配置类和各种注解来配置，是比较简单的方式，也是大势所趋吧。

```java
public interface BeanFactory {
    // 转义符"&"用来获取FactoryBean本身
    String FACTORY_BEAN_PREFIX = "&";
    // 根据bean的名字进行获取bean的实例，这是IOC最大的最大抽象方法
    Object getBean() throws BeansException;
    // 根据bean的名字和Class类型进行获取Bean实例，和上面方法不同的是，bean名字和Bean的类型无关时会抛出异常
    <T> T getBean(String name, Class<T> requiredType) throws BasesException;
    <T> T getBean(Class<T> requiredType) thows BeansException;
    object getBean(Stirng name, Object... args) throws BeansException;
    // 检测整个IOC容器种是否含有整个Bean
    boolean containsBean(String name);
    // 判断这个Bean是不是单例
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    // 查询指定的bean的名字和Class类型是不是指定的Class类型
    boolean isTypeMatch(String name, Class targetType) throws NoSuchBeanDefinitionException;
    // 这里得到bean的别名，如果根据别名检索，那么其原名也会被检索出来
    String[] getAliases(String name);
}
```

### BeanDefinition

这个接口，可以理解为xml bean元素的数据载体。通过对比xml bean标签的属性列表和BeanDefinition的属性列表一看便知。

解析XML的过程，就是Xml`<bean>`元素内容转换为BeanDefinition对象的过程。而且这个接口，支持层级，对应对象的继承。

BeanDefinitionHolder: BeanDefinitionHolder根据名称或者别名持有beanDefiniton，他承载了name和BeanDefinition的映射信息。
BeanWarpper: 提供对标准javaBean的分析和操作方法：单个或者批量获取和设置属性值，获取属性描述符，查询属性的可读性和可写性等。支持属性的嵌套设置，深度没有限制。

AbstractRefreshableApplicationContext的refreshBeanFactory()这个方法

```java

protected final void refreshBeanFactory() throws BeansException {

    if (hasBeanFactory()) {
        destroyBeans();

        closeBeanFactory();
    }

    try {
        // 创建IOC容器
        DefaultListableBeanFactory beanFactory = createBeanFactory();

        beanFactory.setSerializationId(getId());

        customizeBeanFactory(beanFactory);
        //载入loadBeanDefinitions
        loadBeanDefinitions(beanFactory);

        synchronized (this.beanFactoryMonitor) {

        this.beanFactory = beanFactory;
        }

    } catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}

public abstract class AbstractXmlApplicationContext extends AbstractRefreshableConfigApplicationContext {  

/**
 * Loads the bean definitions via an XmlBeanDefinitionReader.
 */

@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {

    // Create a new XmlBeanDefinitionReader for the given BeanFactory.

    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's

    // resource loading environment.

    beanDefinitionReader.setResourceLoader(this);

    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,

    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);

    loadBeanDefinitions(beanDefinitionReader);
}

// 先调用本类里面的loadBeanDefinitions
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {

    Resource[] configResources = getConfigResources();

    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }

    String[] configLocations = getConfigLocations();

    if (configLocations != null) {
        reader.loadBeanDefinitions(configLocations);
    }
}

// 委托给reader.loadBeanDefinitions(configLocation);    XmlBeanDefinitionReader
// 通过XmlBeanDefinitionReader来读取。下面看一下XmlBeanDefinitionReader这个方法，但其实并不在这个类实现这个方法，而是在它的基类里面AbstractBeanDefinitionReader
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {

    Assert.notNull(locations, "Location array must not be null");

    int counter = 0;
    for (String location : locations) {
    counter += loadBeanDefinitions(location);
    }
    return counter;
}

//进入到loadBeanDefinitions
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    Assert.notNull(encodedResource, "EncodedResource must not be null");
    if (logger.isInfoEnabled()) {
    logger.info("Loading XML bean definitions from " + encodedResource.getResource());  
    }

    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
    if (currentResources == null) {
        currentResources = new HashSet<EncodedResource>(4);
        this.resourcesCurrentlyBeingLoaded.set(currentResources);
    }

    if (!currentResources.add(encodedResource)) {
        throw new BeanDefinitionStoreException(
        "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }
    try {
            //获取IO
            InputStream inputStream = encodedResource.getResource().getInputStream();
        try {
            InputSource inputSource = new InputSource(inputStream);
            if (encodedResource.getEncoding() != null) {
                inputSource.setEncoding(encodedResource.getEncoding());
            }
            //这个方法从流中读取
        return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        } finally {
            inputStream.close();
        }
    } catch (IOException ex) {
        throw new BeanDefinitionStoreException("IOException parsing XML document from " + encodedResource.getResource(), ex);
    } finally {
        currentResources.remove(encodedResource);
        if (currentResources.isEmpty()) {
        this.resourcesCurrentlyBeingLoaded.remove();
        }
    }
}

// 进入到doLoadBeanDefinitions  Resource IO封装
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) throws BeanDefinitionStoreException {
    try {
        int validationMode = getValidationModeForResource(resource);
        Document doc = this.documentLoader.loadDocument(
        inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
        return registerBeanDefinitions(doc, resource); //解析XML
    } catch (BeanDefinitionStoreException ex) {
    throw ex;
    } catch (SAXParseException ex) {
    throw new XmlBeanDefinitionStoreException(resource.getDescription(),
    "Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
    } catch (SAXException ex) {
    throw new XmlBeanDefinitionStoreException(resource.getDescription(),"XML document from " + resource + " is invalid", ex);
    } catch (ParserConfigurationException ex) {
    throw new BeanDefinitionStoreException(resource.getDescription(),"Parser configuration exception parsing XML from " + resource, ex);
    } catch (IOException ex) {
        throw new BeanDefinitionStoreException(resource.getDescription(),"IOException parsing XML document from " + resource, ex);
    } catch (Throwable ex) {
        throw new BeanDefinitionStoreException(resource.getDescription(),"Unexpected exception parsing XML document from " + resource, ex);
    }
}
// 进入到registerBeanDefinitions
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    // Read document based on new BeanDefinitionDocumentReader SPI.
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
//documentReader.registerBeanDefinitionsXML解析
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    logger.debug("Loading bean definitions");
    Element root = doc.getDocumentElement();
    BeanDefinitionParserDelegate delegate = createHelper(readerContext, root);
    preProcessXml(root);
    parseBeanDefinitions(root, delegate);
    postProcessXml(root);
}

//-----遍历节点
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
            for (int i = 0; i < nl.getLength(); i++) {
                Node node = nl.item(i);
                if (node instanceof Element) {
                    Element ele = (Element) node;
                    if (delegate.isDefaultNamespace(ele)) {
                    parseDefaultElement(ele, delegate); //默认解析
                } else {
                    delegate.parseCustomElement(ele);
                }
            }
        }
    } else {
        delegate.parseCustomElement(root);
    }
}

// ---判断解析类
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
    // import类型
    importBeanDefinitionResource(ele);
    } else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        // 别名方式
        processAliasRegistration(ele);
    } else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        // bean解析方式
        processBeanDefinition(ele, delegate);
    }
}
```

---

## Bean生命周期分析

1. Spring对Bean进行实例化(相当于程序中的new Xx())
2. Spring将值和Bean的引用注入Bean对应的属性中
3. 如果Bean实现了`BaenNameAware`接口，Spring将Bean的ID传递给setBeanName()方法（实现`BeanNameAware`接口主要是为了通过Bean的引用来获取Bean得ID，一般业务中是很少有用到Bean的ID的）
4. 如果Bean实现了`BeanfactoryAware`接口，Spring将调用setBeanFactory(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。（实现`BeanFactoryAware`主要目的是为了获取Spring容器，如Bean通过Spring容器发布时间等）
5. 如果Bean实现了`ApplicationContextAware`接口，Spring容器将调用`setApplicationContext()`方法，把bean所在的应用上下文的引用传入。（作用与`BeanFactory`类似都是为了获取Spring容器，不同的是Spring容器在调用`setApplicationContext`方法时会把它自己作为`setApplicationContext`的参数传入，而Spring容器在调用`setBeanFactory`前需要程序员自己指定（注入）`setBeanFactory`里的参数`BeanFactory`）
6. 如果Bean实现了`BeanPostProcess`接口，Spring将调用它们的`postProcessBeforeInitialization`（预初始化）方法 
（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）
7. 如果Bean实现了`InitializingBean`接口，Spring将调用它们的`afterPropertiesSet`方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。
8. 如果Bean实现了`BeanPostProcess`接口，Spring将调用它们的`postProcessAfterInitialization`（后初始化）方法（作用与6的一样，只不过`postProcessBeforeInitialization`是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同)
9. 此时，bean已经准备就绪，可以被程序使用了，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁
10. 如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。
