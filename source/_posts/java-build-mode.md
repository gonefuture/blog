---
title: java build mode （转载自zooooooooy）
---

还有一种通过内部类来实现单例的方式，静态内部类只能访问外部类的静态方法和静态属性。现在一般利用这个特性来实现单例模式。应该类在初始化的时候线程是互斥的，可以完美的解决单例创建冲突的问题。

借用mybatis单例的实现来看一下：

Class:ParameterMapping

构造方法私有化：



private ParameterMapping() {
}

静态内部类来初始化外部对象：

public static class Builder {
    private ParameterMapping parameterMapping = new ParameterMapping();

    public Builder(Configuration configuration, String property, TypeHandler<?> typeHandler) { --生成ParameterMapping对象
      parameterMapping.configuration = configuration;
      parameterMapping.property = property;
      parameterMapping.typeHandler = typeHandler;
      parameterMapping.mode = ParameterMode.IN;
    }

    public Builder(Configuration configuration, String property, Class<?> javaType) {
      parameterMapping.configuration = configuration;
      parameterMapping.property = property;
      parameterMapping.javaType = javaType;
      parameterMapping.mode = ParameterMode.IN;
    }
}

提供公共build方法来返回该单例对象：



public ParameterMapping build() {
      resolveTypeHandler();
      validate();
      return parameterMapping;
}

这种方式相对于一般的单例方式的实现，复杂度会相对高一点,其实是java的builder建造者模式。








