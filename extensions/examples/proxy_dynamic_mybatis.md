# 代理模式在MyBatis源码中的应用

MyBatis 是一个应用非常广泛的优秀持久层框架，它几乎避免了所有 JDBC 代码、手动设置参数和获取结果集等工作。MyBatis 可以使用简单的 XML 配置文件或注解来映射类、接口和 POJO 与数据库记录的对应关系。

如果您使用过 MyBatis，则会发现 MyBatis 的使用非常简单。*首先需要定义一个 Dao 接口，然后编写一个与 Dao 接口对应的 Mapper 配置文件。Java 对象与数据库字段的映射关系和 Dao 接口对应的 SQL 语句都写在配置文件中，非常简单清晰。*

那么 Dao 接口是怎么和 Mapper 文件映射的呢？只有一个 Dao 接口，又是怎么以对象的形式来实现数据库读写操作的呢？

在了解代理模式后，我们应该很容易猜到，可以**通过动态代理来创建 Dao 接口的代理对象，并通过这个代理对象实现数据库的操作**。

在 MyBatis 中，MapperProxyFactory、MapperProxy、MapperMethod 是三个很重要的类。我们只需要了解这 3 个类，就能明白 Dao 接口与 SQL 的映射原理。

## 1. MapperProxyFactory

调用 addMapper() 方法时，跟踪源码就会发现 MapperRegistry 类的 addMapper() 方法中有如下语句：

knownMappers.put(type, new MapperProxyFactory<T>(type));

type 就是 Dao 接口，下面来看 MapperProxyFactory 类。

```java
public class MapperProxyFactory<T> {    
    private final Class<T> mapperInterface;    
    private Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<Method, MapperMethod>();    
    
    public MapperProxyFactory(Class<T> mapperInterface) {        
        this.mapperInterface = mapperInterface;    
    }    
    
    public Class<T> getMapperInterface() {        
        return mapperInterface;    
    }    
    
    public Map<Method, MapperMethod> getMethodCache() {        
        return methodCache;    
    }    
    
    @SuppressWarnings("unchecked")    
    protected T newInstance(MapperProxy<T> mapperProxy) {        
        return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), 
                                          new Class[]{mapperInterface}, mapperProxy);    
    }    
    
    public T newInstance(SqlSession sqlSession) {        
        final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, 
                                                              mapperInterface, methodCache);        
        return newInstance(mapperProxy);    
    }
}
```

MapperProxyFactory 看名字就知道这是一个工厂类，目的就是为了生成 MapperProxy。下面我们主要看它的构造方法和 newInstance() 方法。

构造方法中传入了一个 Class，通过参数名 mapperInterface 可以很容易猜到这个类就是个 Dao 接口。

MapperProxyFactory 中有 2 个 newInstance() 方法，权限修饰符不同，一个是 protected，一个是 public。

1. protected 的 newInstance() 方法中首先创建了一个 MapperProxy 类的对象，然后通过 Proxy.newProxyInstance() 方法创建了一个对象并返回，也是通过同样的方法返回了 mapperInterface 接口的代理对象。
2. public 的 newInstance() 方法中有一个参数 SqlSession，SqlSession 处理的就是执行一次 SQL 的过程。


上面提到的 MapperProxy 类显然是 InvocationHandler 接口的实现。因此，可以说 MapperProxyFactory 类是一个创建代理对象的工厂类，它通过构造函数传入自定义的 Dao 接口，并通过 newInstance 方法返回 Dao 接口的代理对象。

## 2. MapperProxy

看到这里，是不是有了一种豁然开朗的感觉，但新的疑问又来了，我们定义的 Dao 接口的方法并没有被实现，那这个代理对象又是如何实现增删改查的呢？带着这个疑问，我们来看一下 MapperProxy 类，源码如下：

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {    
    private static final long serialVersionUID = -6424540398559729838L;    
    private final SqlSession sqlSession;    
    private final Class<T> mapperInterface;    
    private final Map<Method, MapperMethod> methodCache;    
    
    public MapperProxy(SqlSession sqlSession, 
                       Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {        
        this.sqlSession = sqlSession;        
        this.mapperInterface = mapperInterface;        
        this.methodCache = methodCache;    
    }    
    
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {        
        if (Object.class.equals(method.getDeclaringClass())) {            
            try {                
                return method.invoke(this, args);            
            } catch (Throwable t) {                
                throw ExceptionUtil.unwrapThrowable(t);            
            }        
        }        
        
        final MapperMethod mapperMethod = cachedMapperMethod(method);        
        return mapperMethod.execute(sqlSession, args);    
    }    
    
    private MapperMethod cachedMapperMethod(Method method) {        
        MapperMethod mapperMethod = methodCache.get(method);        
        if (mapperMethod == null) {            
            mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
            methodCache.put(method, mapperMethod);        
        }        
        
        return mapperMethod;    
    }
}
```

很明显因为 Java 动态代理，MapperProxy 实现了 InvocationHandler 接口。下面先看 invoke() 方法。

在 invoke() 方法中，首先检查了如果是 Object 的方法就直接调用方法本身；如果不是就把方法 Method 包装成 MapperMethod。我们前面已经提到了 MapperMethod 主要就是处理方法的注解，参数，返回值，以及参数与 SQL 语句中参数的对应关系。因为将 Method 处理为 MapperMethod 是一个较频繁的操作，所以这里做了缓存处理。

### MapperProxy中的成员变量

下面我们讲解 MapperProxy 类的 3 个成员变量，分别如下：

#### 1）SqlSession

通过名字就知道 SqlSession 变量是一个执行 SQL 的接口，简单看一下它的接口定义。

```java
public interface SqlSession extends Closeable {    
    <T> T selectOne(String statement);    
    <T> T selectOne(String statement, Object parameter);       
    // 下面省略
}
```

这个接口方法的入参是 statement 和参数（parameter），返回值是数据对象。statement 可能会被误解为 SQL 语句，但其实这里的 statement 是指 Dao 接口方法的名称，自定义的 SQL 语句都被缓存在 Configuration 对象中。

在 SqlSession 中，可以通过 Dao 接口的方法名称找到对应的 SQL 语句。因此，可以想到代理对象本质上就是要将执行的方法名称和参数传入 SqlSession 的对应方法中，根据方法名称找到对应的 SQL 语句并替换参数，最后得到返回的结果。

#### 2）mapperInterface

mapperInterface 的作用要结合第 3 个成员变量来说明。

#### 3）methodCache

methodCache 其实就是一个 Map 键值对结构，键是 Method，值是 MapperMethod。

下面再回到 invoke() 方法的最后两行，它首先通过 cachedMapperMethod() 方法找到与将要执行的 Dao 接口方法对应的 MapperMethod，然后调用 MapperMethod 的 execute() 方法来实现数据库的操作。这里显然是将 SqlSession 传入 MapperMethod 内部，并在 MapperMethod 内部将要执行的方法名和参数再传入到 SqlSession 对应的方法中去执行。

## 3. MapperMethod

下面来看 MapperMethod 类的内部，看它如何完成 SQL 的执行。

```java
public class MapperMethod{    
    private final SqlCommand command;    
    private final MethodSignature method;
}
```

MapperMethod 类中有两个成员变量，分别是 SqlCommand 和 MethodSignature。虽然这两个类的代码看起来很多，但实际上这两个内部类非常简单。

- SqlCommand 主要解析了接口的方法名称和方法类型，定义了诸如 INSERT、SELECT、DELETE 等数据库操作的枚举类型。
- MethodSignature 则解析了接口方法的签名，即接口方法的参数名称和参数值的映射关系，即通过 MethodSignature 类可以将入参的值转换成参数名称和参数值的映射。


最后来看 MapperMethod 类中最重要的 execute() 方法。

```java
public Object execute(SqlSession sqlSession, Object[] args) {    
    Object param;    
    Object result; 
    
    switch(this.command.getType()) {    
        case INSERT:        
            param = this.method.convertArgsToSqlCommandParam(args);       
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));       
            break;    
        case UPDATE:        
            param = this.method.convertArgsToSqlCommandParam(args);        
            result = this.rowCountResult(sqlSession.update(this.command.getName(), param));        
            break;    
        case DELETE:       
            param = this.method.convertArgsToSqlCommandParam(args);        
            result = this.rowCountResult(sqlSession.delete(this.command.getName(), param));        
            break;    
        case SELECT:        
            if (this.method.returnsVoid() && this.method.hasResultHandler()) { 
            // 返回类型为void            
                this.executeWithResultHandler(sqlSession, args);            
                result = null;        
            } else if (this.method.returnsMany()) { 
                // 返回类型为集合或数组            
                result = this.executeForMany(sqlSession, args);        
            } else if (this.method.returnsMap()) {
                // 由@MapKey控制返回            
                result = this.executeForMap(sqlSession, args);        
            } else if (this.method.returnsCursor()) {
                // 返回类型为Cursor<T>，采用游标            
                result = this.executeForCursor(sqlSession, args);        
            } else {            
                // 其他类型            
                param = this.method.convertArgsToSqlCommandParam(args);            
                result = sqlSession.selectOne(this.command.getName(), param);        
            }        
            break;    
        case FLUSH:        
            result = sqlSession.flushStatements();        
            break;    
        default:        
            throw new BindingException("Unknown execution method for: " +
                                       this.command.getName());    
    }    
    
    if (result == null && this.method.getReturnType().isPrimitive() && 
        !this.method.returnsVoid()) { 
        throw new BindingException("Mapper method '" + this.command.getName() + 
                                   " attempted to return null from a method with a primitive return type (" 
                                   + this.method.getReturnType() + ").");    
    } else {        
        return result;    
    }
}
```

通过上面对 SqlCommand 和 MethodSignature 的简单分析，我们很容易理解这段代码。

1. 首先，根据 SqlCommand 中解析出来的方法类型选择对应 SqlSession 中的方法，即如果是 INSERT 类型，则选择 SqlSession.insert() 方法来执行数据库操作。
2. 其次，通过 MethodSignature 将参数值转换为 Map＜Key, Value＞ 的映射，Key 是方法的参数名称，Value 是参数值。
3. 最后，将方法名称和参数传入对应的 SqlSession 的方法中执行。至于在配置文件中定义的 SQL 语句，则被缓存在 SqlSession 的成员变量 中。


Configuration 中有非常多参数，其中一个是 mappedStatements，它保存了我们在配置文件中定义的所有方法。还有一个是 mappedStatement，它保存了我们在配置文件中定义的各种参数，包括 SQL 语句。

到这里，我们应该对 MyBatis 中如何通过将配置与 Dao 接口映射起来、如何通过代理模式生成代理对象来执行数据库读写操作有了较为宏观的认识，至于 SqlSession 中如果将参数与 SQL 语句结合组装成完整的 SQL 语句，以及如何将数据库字段与 Java 对象映射，感兴趣的小伙伴可以自行分析相关的源码。