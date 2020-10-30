# 建造者模式在框架源码中的应用

本节主要介绍建造者模式在 JDK 和 MyBatis 源码中的应用。

## 建造者模式在JDK源码中的应用

JDK 的 StringBuilder 类中提供了 append() 方法，这就是一种链式创建对象的方法，开放构造步骤，最后调用 toString() 方法就可以获得一个完整的对象。StringBuilder 类源码如下：

```java
public final class StringBuilder extends AbstractStringBuilder    
    implements java.io.Serializable, CharSequence {    
    ...    
    public StringBuilder append(Object obj) {        
        return append(String.valueOf(obj));    
    }    
    ...    
    public String toString() {       
        // Create a copy, don't share the array       
        return new String(value, 0, count);    
    }
    ...
}
```

## 建造者模式在MyBatis源码中的应用

MyBatis 中 SqlSessionFactoryBuiler 类用到了建造者模式。且在 MyBatis 中 SqlSessionFactory是由 SqlSessionFactoryBuilder 产生的，代码如下：

```java
public SqlSessionFactory build(Configuration config) {    
    return new DefaultSqlSessionFactory(config);
}
```

DefaultSqlSessionFactory 的构造器需要传入 MyBatis 核心配置类 Configuration 的对象作为参数，而 Configuration 庞大复杂，初始化比较麻烦，因此使用了专门的建造者 XMLConfigBuilder 进行构建。

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {    
    try {        
        // 创建建造者XMLConfigBuilder实例        
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);        
        
        // XMLConfigBuilder的parse()构建Configuration实例        
        return build(parser.parse());    
    } catch (Exception e) {        
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);    
    } finally {        
        ErrorContext.instance().reset();        
        try {            
            inputStream.close();        
        } catch (IOException e) {            
            // Intentionally ignore. Prefer previous error.        
        }    
    }
}
```

XMLConfigBuilder 负责 Configuration 各个组件的创建和装配，整个装配的流程化过程如下：

```java
private void parseConfiguration(XNode root) {    
    try {        
        //issue #117 read properties first        
        // Configuration#        
        propertiesElement(root.evalNode("properties"));        
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        loadCustomVfs(settings);    
        
        typeAliasesElement(root.evalNode("typeAliases"));        
        pluginElement(root.evalNode("plugins"));
        
        objectFactoryElement(root.evalNode("objectFactory"));
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        reflectorFactoryElement(root.evalNode("reflectorFactory"));        
        settingsElement(settings);        
        
        // read it after objectFactory and objectWrapperFactory issue #631
        environmentsElement(root.evalNode("environments")); 
        databaseIdProviderElement(root.evalNode("databaseIdProvider")); 
        typeHandlerElement(root.evalNode("typeHandlers"));        
        mapperElement(root.evalNode("mappers"));    
    } catch (Exception e) {        
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);    
    }
}
```

XMLConfigBuilder 负责创建复杂对象 Configuration，其实就是一个具体建造者角色。SqlSessionFactoryBuilder 只不过是做了一层封装去构建 SqlSessionFactory 实例，这就是建造者模式简化构建的过程。