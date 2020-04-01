---
title: springboot （转载自zooooooooy）
---

### springMVC
springboot提供了开箱即用的web mvc，项目中用到的是gradle，只需要引用两个包就可以了
```
compile('org.springframework.boot:spring-boot-starter')
compile('org.springframework.boot:spring-boot-starter-web')
```
并不需要做什么其他配置，就可以启动web项目，编写restful接口。还是有一些需要个性化定制的功能需要我们手动去配置。有一些并不是springboot独有，配置的时候都是采用代码代替xml的方式。问题记录如下：
*	Json序列化，
springmvc都是需要配置的，之前都是通过xml来配置。在springboot里面需要自定义MessageConverter。注册bean的方式加载到spring消息转换器里面，默认是放置到转换器队列的最前面优先解析。
springmvc默认的json解析器是Jackson,我这边算是先继承MappingJackson2HttpMessageConverter，修改覆盖初始方法来定制化项目需要的json格式化规则。
    ```
    public class JsonMessageConverter extends MappingJackson2HttpMessageConverter {

            @Override
            protected void init(ObjectMapper objectMapper) {
                objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
                objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
                objectMapper.enable(SerializationFeature.WRITE_ENUMS_USING_TO_STRING);
                objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
                objectMapper.enable(DeserializationFeature.READ_ENUMS_USING_TO_STRING);
                objectMapper.enable(DeserializationFeature.ACCEPT_EMPTY_ARRAY_AS_NULL_OBJECT);

                objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

                super.init(objectMapper);
            }
        }

    ```
后面需要将这个注册到spring bean管理里面。在configuration类里面注册
   ```
   @Bean
    public HttpMessageConverters customConverters() {
        return new HttpMessageConverters(new JsonMessageConverter());
    }

	```
*	UTF8编码
同样是在configuration类里面注册，其实configuration就相当于spring的一个xml文件，每个方法相当于定义的一个bean，方法之间是可以互相依赖的。spring已经考虑到这点，不用担心依赖的时候破坏单例的特性。
    ```
    @Bean
    public CharacterEncodingFilter encodingFilter() {

        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceEncoding(true);
        return encodingFilter;
    }

	```
*	拦截器
用到拦截器的时候，需要继承WebMvcConfigurerAdapter，通过适配器的模式，覆盖相应的方法。当然同时这个类需要标注为configuration类。

    | 方法 |
	|--------|
	|configurePathMatch|
    |configureContentNegotiation|
    |configureAsyncSupport|
    |configureDefaultServletHandling|
    |addFormatters|
    |addInterceptors|
    |addResourceHandlers|
    |addCorsMappings|
    |addViewControllers|
    |configureViewResolvers|
    |addArgumentResolvers|
    |addReturnValueHandlers|
    |configureMessageConverters|
    |extendMessageConverters|
    |configureHandlerExceptionResolvers|
    |extendHandlerExceptionResolvers|
    |getValidator|
    |getMessageCodesResolver|
拦截器添加选择覆盖addInterceptors
```
@Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MappedInterceptor(new String[]{"/**"}, new String[]{""},
                new MvcInterceptor()));
    }
```
MvcInterceptor是自定义的拦截器，如果需要用到url拦截的功能，需要使用spring带的MappedInterceptor定义拦截的url和排除的url
项目中还用到了addArgumentResolvers，addReturnValueHandlers就不一一列举了。

### 项目
*	属性文件
	使用@PropertySources({@PropertySource(value = "classpath:setting.properties")})来引入，不过这个不是springboot里面的东西。

*	项目分布
	*	css,js前端等用到的组件都分布在public下面或者static下面
    需要渲染的模板定义在templates里面，安利一下我正在使用的模板[pebble](http://www.mitchellbosecke.com/pebble/documentation/guide/basic-usage)
    *	数据库使用的是mybatis，sql还是写在xml里面在，始终觉得在代码里面写sql比较别扭。
    *	打包工具使用的是gradle，第一次接触，感觉还是不错，在编译速度上超过了maven。
    	```
        compile('org.springframework.boot:spring-boot-starter')
        compile('org.springframework.boot:spring-boot-starter-log4j2')
        compile('org.springframework.boot:spring-boot-starter-web') {
            //exclude group:'org.springframework.boot',module:"spring-boot-starter-tomcat"
        }
        compile('org.springframework.boot:spring-boot-devtools')
        compile('org.springframework.boot:spring-boot-starter-test')
        compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.0')

        compile('mysql:mysql-connector-java:6.0.5')
        compile('com.zaxxer:HikariCP:2.6.1')

        compile('org.projectlombok:lombok:1.16.16')

        //token
        compile("io.jsonwebtoken:jjwt:0.7.0")

        //bean copy
        compile('commons-beanutils:commons-beanutils:1.9.3')

        //apache commons lang3
        compile('org.apache.commons:commons-lang3:3.6')

        compile('javax.validation:validation-api:2.0.0.CR2')

        testCompile('junit:junit:4.12')

		```
项目中用到的jar包如上，springboot的版本号定义在buildscript里面在。使用的是1.5.2

*	项目发布
项目还是采用外置的tomcat发布，需要在启动类Application继承SpringBootServletInitializer，覆盖
    ```
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

	```
    发布的脚本上gradle clean build -x test
*	profile配置
	使用了gradle，在dev环境和prod环境的配置上，相对来说是没有maven好使的，
    需要启动的时候在环境变量里面配置-Dspring.profiles.active=dev
	其他的跟平常的spring开发还都是保持一致的。随着后续功能的开发慢慢会涉及到更多springboot的功能，个人感觉在开发效率上还是提升了不少。

_ _ _
后续研究了下gradle的分环境打包方式，参数网上的例子，操作步骤如下
*	添加配置文件config.groovy
	environments {
    	dev {
        }
        prod {
        }
    }
在dev和prod写不同环境下的变量值，按照groovy的方式
*	在build.gradle获取打包时指定的环境参数
```
    ext {
        profile = System.properties['spring.profiles.active']
    }
```
*	读取config.groovy对应环境的的变量值
```
    def loadGroovyConfig(){
        def configFile = file('config.groovy')
        new ConfigSlurper(profile).parse(configFile.toURL()).toProperties()
    }

    processResources {
        from(sourceSets.main.resources.srcDirs) {
            filter(org.apache.tools.ant.filters.ReplaceTokens, tokens: loadGroovyConfig())
        }
    }
```
* 变量覆盖如图
  ![log4j2](/images/TIM截图20180605114425.png)
后续打包还是按照正常的方式，指定-Dspring.profiles.active=dev或者prod就可以
