# 静态代理和动态代理

**静态代理**就是按照代理模式书写的代码，其特点是代理类和目标类在代码中是确定的，因此称为静态。静态代理可以在不修改目标对象功能的前提下，对目标功能进行扩展。

但是静态代理显然不够灵活，这时就需要动态代理。

**动态代理**也叫 **JDK 代理**或**接口代理**，有以下特点：

- 代理对象不需要实现接口
- 代理对象的生成是利用 JDK 的 API 动态的在内存中构建代理对象
- 能在代码运行时动态地改变某个对象的代理，并且能为代理对象动态地增加方法、增加行为


一般情况下，动态代理的底层不用我们亲自去实现，可以使用线程提供的 API 。例如，在 Java 生态中，目前普遍使用的是 JDK 自带的代理和 GGLib 提供的类库。

JDK 实现代理只需要使用 newProxyInstance 方法，该方法需要接收三个参数，语法格式如下：

```java
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h )
```

注意该方法在 Proxy 类中是静态方法，且接收的三个参数说明依次为：

- ClassLoader loader：指定当前目标对象使用类加载器，获取加载器的方法是固定的
- Class<?>[] interfaces：目标对象实现的接口的类型，使用泛型方式确认类型
- InvocationHandler h：事件处理，执行目标对象的方法时，会触发事件处理器的方法，把当前执行目标对象的方法作为参数传入


下面根据实例介绍静态代理和动态代理。

众所周知，编程非常培养孩子的逻辑思维能力。很多父母为了不让孩子输在起跑线上，就开始到处为孩子找辅导老师。下面来看代码实现。

创建顶层接口 IPerson，代码如下：

```java
public interface IPerson {    
    void findTeacher(); //找老师
}
```

儿子张三要找老师，实现 IPerson 接口，ZhangSan 类代码如下：

```java
public class ZhangSan implements IPerson {    
    
    @Override    
    public void findTeacher() {        
        System.out.println("儿子张三提出要求");    
    }
}
```

父亲张老三要帮儿子张三找老师，实现 IPerson 接口，ZhangLaoSan 类代码如下：

```java
public class ZhangLaoSan implements IPerson {    
    private ZhangSan zhangsan;   
    
    public ZhangLaoSan(ZhangSan zhangsan) {        
        this.zhangsan = zhangsan;    
    }    
    
    @Override    
    public void findTeacher() {        
        System.out.println("张老三开始找老师");        
        zhangsan.findTeacher();        
        System.out.println("开始学习");    
    }
}
```

新建 Test 类测试代码。

```java
public class Test {    
    public static void main(String[] args) {        
        ZhangLaoSan zhanglaosan = new ZhangLaoSan(new ZhangSan());        
        zhanglaosan.findTeacher();    
    }
}
```

运行结果如下所示：

张老三开始找老师
儿子张三提出要求
开始学习

上面的场景有个弊端，就是自己的父亲只会帮自己的子女去挑选辅导老师，别人家的孩子是不会管的。于是社会上这样业务发展成了一个产业，出现了答疑、辅导班、培训机构等，还有各种各样的定制套餐。

这样如果还是用静态代理成本就太高了，需要一个更加通用的解决方案，满足任何想学习编程找老师的需求。这就由静态代理升级到了动态代理。采用动态代理基本上只要是人（IPerson）就可以提供找老师服务。

下面基于 JDK 动态代理支持来升级一下代码。

首先创建辅导班类 JdkFuDao。

```java
public class JdkFuDao implements InvocationHandler {    
    private IPerson target;    
    
    public IPerson getInstance(IPerson target) {        
        this.target = target;        
        Class<?> clazz = target.getClass();        
        return (IPerson) Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);    
    }    
    
    @Override    
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {        
        before();        
        Object result = method.invoke(this.target, args);        
        after();       
        
        return result;    
    }    
    
    private void after() {        
        System.out.println("双方同意，开始辅导");    
    }    
    
    private void before() {        
        System.out.println("这里是C语言中文网辅导班，已经收集到您的需求，开始挑选老师");    
    }
}
```

然后创建一个类 ZhaoLiu。

```java
public class ZhaoLiu implements IPerson {    
    
    @Override    
    public void findTeacher() {        
        System.out.println("符合赵六的要求");    
    }    
    
    public void buyInsure() {    
    }
}
```

需要注意的是，代理对象不需要实现接口，但是目标对象一定要实现接口，否则不能用动态代理。

最后客户端测试代码如下：

```java
public class Test {    
    public static void main(String[] args) {        
        JdkFuDao jdkFuDao = new JdkFuDao();        
        IPerson zhaoliu = jdkFuDao.getInstance(new ZhaoLiu());
        
        zhaoliu.findTeacher();    
    }
}
```

运行结果如下所示：

这里是C语言中文网辅导班，已经收集到您的需求，开始挑选老师
符合赵六的要求
双方同意，开始辅导

## 静态代理和动态代理的区别

静态代理和动态代理主要有以下几点区别：

- **静态代理只能通过手动完成代理操作，如果被代理类增加了新的方法，则代理类需要同步增加，违背开闭原则**。
- **动态代理采用在运行时动态生成代码的方式，取消了对被代理类的扩展限制，遵循开闭原则**。
- 若动态代理要对目标类的增强逻辑进行扩展，结合策略模式，只需要新增策略类便可完成，无需修改代理类的代码。