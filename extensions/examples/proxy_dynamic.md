# 彻底搞懂JDK动态代理核心原理

通过前面的学习，我们知道 JDK 动态代理是代理模式的一种实现方式，那么它是如何实现的呢？俗话说：不仅知其然，还得知其所以然。下面主要探究一下 JDK 动态代理的原理，并模仿 JDK 动态代理编写一个属于自己的动态代理。

**JDK 动态代理采用字节重组，重新生成对象来替代原始对象，以达到动态代理的目的**。JDK 动态代理生成对象的步骤大致如下。

1. *获取被代理对象的引用，并且获取它的所有接口。*
2. *JDK 动态代理类重新生成一个新的类，同时新的类要实现被代理类实现的所有接口。*
3. *动态生成 Java 代码，新加的业务逻辑方法由一定的逻辑代码调用（在代码中体现），拿到被代理对象的引用。*
4. *编译新生成的 Java 代码 .class 字节码文件。*
5. *重新加载到 JVM 中运行*。


以上过程就叫作字节码重组。

*JDK 中有一个规范，在 ClassPath 目录下只要是 $ 开头的 .class 文件，一般都是自动生成的。*

下面我们查看以 $ 开头的 .class 文件的内容。方法为：首先将内存中的对象字节码通过文件流输出到一个新的 .class 文件，然后使用反编译工具查看源码。

```java
public static void main(String[] args) {    
    try {        
        IPerson obj = (IPerson) new JdkFuDao().getInstance(new ZhangSan());        
        obj.findTeacher();        
        
        //通过反编译工具查看源代码        
        byte bytes[] = ProxyGenerator.generateProxyClass("$Proxy0", 
                                                         new Class[]{IPerson.class}); 
        FileOutputStream os = new FileOutputStream("D://$Proxy0.class");        
        os.write(bytes);        
        os.close();  
        
    } catch (Exception e) {        
        e.printStackTrace();    
    }
}
```

运行以上代码，成功后就可以在 D 盘找到 $Proxy0.class 文件了。

这里我们使用 Jad 工具进行反编译，当然您也可以使用其他反编译工具。反编译后得到 $Proxy0.jad 文件，文件内容如下：

```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3)
import java.lang.reflect.*;
import proxy.IPerson;

public final class $Proxy0 extends Proxy implements IPerson{    
    private static Method m1;    
    private static Method m0;    
    private static Method m3;    
    private static Method m2; 
    
    static{        
        try{            
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { 
                Class.forName("java.lang.Object")});            
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);            
            m3 = Class.forName("proxy.IPerson").getMethod("findTeacher", new Class[0]);            
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);        
        } catch(NoSuchMethodException nosuchmethodexception){            
            throw new NoSuchMethodError(nosuchmethodexception.getMessage());       
        } catch(ClassNotFoundException classnotfoundexception){           
            throw new NoClassDefFoundError(classnotfoundexception.getMessage());        
        }    
    }
    
    public $Proxy0(InvocationHandler invocationhandler){        
        super(invocationhandler);   
    }   
    
    public final boolean equals(Object obj){        
        try {            
            return ((Boolean)super.h.invoke(this, m1, new Object[] {obj})).booleanValue();        
        } catch(Error _ex) { 
        } catch(Throwable throwable){            
            throw new UndeclaredThrowableException(throwable);        
        }    
    }    
    
    public final int hashCode(){        
        try {            
            return ((Integer)super.h.invoke(this, m0, null)).intValue();        
        } catch(Error _ex) { 
        } catch(Throwable throwable){            
            throw new UndeclaredThrowableException(throwable);        
        }    
    }  
    
    public final String toString(){        
        try {            
            return (String)super.h.invoke(this, m2, null);        
        } catch(Error _ex) { 
        } catch(Throwable throwable){            
            throw new UndeclaredThrowableException(throwable);        
        }    
    }  
    
    public final void findTeacher(){        
        try{            
            super.h.invoke(this, m3, null);            
            return;        
        }catch(Error _ex) {
        }catch(Throwable throwable){            
            throw new UndeclaredThrowableException(throwable);        
        }    
    }    
}
```

我们发现，$Proxy0 继承了 Proxy 类，并且实现了 IPerson 的接口，而且重写了 equals、hashCode、toString、findTeacher() 等方法。其中，在静态代码块中，通过反射获取了代理类的所有方法，而且保存了所有方法的引用，重写的方法用反射调用目标对象的方法。通过 invoke 执行代理类中的目标方法 findTeacher。

学到这里，大家一定会好奇，$Proxy0.jad 中的代码都是从哪里来的？这些都是 JDK 自动生成的。

## 手动模拟实现动态代理

下面我们不依赖 JDK，自己来动态生成源码、动态完成编译，然后替代目标对象并执行。

JDK 代理需要实现 java.lang.reflect.InvocationHandler 接口，并使用 java.lang.reflect.Proxy.newProxyInstance() 方法生成代理对象。

> 下面我们使用 JDK 代理的类名和方法名定义，由于篇幅原因，没有展示其源代码，大家可以自行查看。

仿照 InvocationHandler 接口，创建 MyInvocationHandler 接口并定义 invoke 方法，代码如下：

```java
public interface MyInvocationHandler {    
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

仿照 Proxy 类，创建 MyProxy 类，代码如下：

```java
/*** 自己实现的代理类，用来生成字节码文件，并动态加载到JVM中*/
public class MyProxy {    
    public static final String ln = "\r\n";    
    
    public static Object newProxyInstance(MyClassLoader  classLoader, 
                                          Class<?>[] interfaces, MyInvocationHandler h) {        
        try {            
            //1、动态生成源代码.java文件            
            String src = generateSrc(interfaces);            
            
            //2、Java文件输出磁盘            
            String filePath = MyProxy.class.getResource("").getPath();            
            File f = new File(filePath + "$Proxy0.java");            
            FileWriter fw = new FileWriter(f);            
            fw.write(src);            
            fw.flush();            
            fw.close();            
            
            //3、把生成的.java文件编译成.class文件            
            JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();            
            StandardJavaFileManager manage = compiler.getStandardFileManager(null, null, null); 
            Iterable iterable = manage.getJavaFileObjects(f);            
            JavaCompiler.CompilationTask task = compiler.getTask(null, manage, null, null, null, iterable);
            task.call();            
            manage.close();            
            
            //4、编译生成的.class文件加载到JVM中来            
            Class proxyClass = classLoader.findClass("$Proxy0");            
            Constructor c = proxyClass.getConstructor(MyInvocationHandler.class);            
            f.delete();            
            
            //5、返回字节码重组以后的新的代理对象            
            return c.newInstance(h);  
            
        } catch (Exception e) {            
            e.printStackTrace();        
        }        
        
        return null;    
    }    
    
    private static String generateSrc(Class<?>[] interfaces) {        
        StringBuffer sb = new StringBuffer();        
        sb.append(MyProxy.class.getPackage() + ";" + ln);        
        sb.append("import " + interfaces[0].getName() + ";" + ln);        
        sb.append("import java.lang.reflect.*;" + ln);        
        sb.append("public class $Proxy0 implements " + interfaces[0].getName() + "{" + ln); 
        sb.append("MyInvocationHandler h;" + ln);        
        sb.append("public $Proxy0(MyInvocationHandler h) { " + ln);        
        sb.append("this.h = h;");        
        sb.append("}" + ln);        
        
        for (Method m : interfaces[0].getMethods()) {            
            Class<?>[] params = m.getParameterTypes();            
            StringBuffer paramNames = new StringBuffer();           
            StringBuffer paramValues = new StringBuffer();            
            StringBuffer paramClasses = new StringBuffer();            
            
            for (int i = 0; i < params.length; i++) {                
                Class clazz = params[i];                
                String type = clazz.getName();                
                String paramName = toLowerFirstCase(clazz.getSimpleName()); 
                paramNames.append(type + " " + paramName);                
                paramValues.append(paramName);                
                paramClasses.append(clazz.getName() + ".class");                
                
                if (i > 0 && i < params.length - 1) {                    
                    paramNames.append(",");                    
                    paramClasses.append(",");                    
                    paramValues.append(",");                
                }            
            }            
            
            sb.append("public " + m.getReturnType().getName() + " " + 
                      m.getName() + "(" + paramNames.toString() + ") {" + ln);            
            sb.append("try{" + ln);            
            sb.append("Method m = " + interfaces[0].getName() + ".class.getMethod(\"" + 
                      m.getName() + "\",new Class[]{" + paramClasses.toString() + "});" + ln); 
            sb.append((hasReturnValue(m.getReturnType()) ? "return " : "") +
                      getCaseCode("this.h.invoke(this,m,new Object[]{" + 
                                  paramValues + "})", m.getReturnType()) + ";" + ln);
            sb.append("}catch(Error _ex) { }");            
            sb.append("catch(Throwable e){" + ln);            
            sb.append("throw new UndeclaredThrowableException(e);" + ln);            
            sb.append("}");            
            sb.append(getReturnEmptyCode(m.getReturnType()));            
            sb.append("}");        
        }       
        
        sb.append("}" + ln);        
        return sb.toString();    
    }    
    
    private static Map<Class, Class> mappings = new HashMap<Class, Class>();  
    
    static {        
        mappings.put(int.class, Integer.class);    
    }    
    
    private static String getReturnEmptyCode(Class<?> returnClass) {        
        if (mappings.containsKey(returnClass)) {            
            return "return 0;";        
        } else if (returnClass == void.class) {            
            return "";        
        } else {            
            return "return null;";        
        }    
    }    
    
    private static String getCaseCode(String code, Class<?> returnClass) {        
        if (mappings.containsKey(returnClass)) {            
            return "((" + mappings.get(returnClass).getName() + 
                ")" + code + ")." + returnClass.getSimpleName() + "Value()";        
        }        
        
        return code;    
    }    
    
    private static boolean hasReturnValue(Class<?> clazz) {        
        return clazz != void.class;    
    }    
    
    private static String toLowerFirstCase(String src) {        
        char[] chars = src.toCharArray();        
        chars[0] += 32;        
        return String.valueOf(chars);    
    }
}
```

创建 MyClassLoader 类，代码如下：

```java
public class MyClassLoader  extends ClassLoader {    
    private File classPathFile;    
    
    public MyClassLoader () {        
        String classPath = MyClassLoader.class.getResource("").getPath();        
        this.classPathFile = new File(classPath);    
    }    
    
    @Override    
    protected Class<?> findClass(String name) throws ClassNotFoundException {        
        String className = MyClassLoader.class.getPackage().getName() + "." + name;        
        
        if (classPathFile != null) {            
            File classFile = new File(classPathFile, name.replaceAll("\\.", "/") + ".class");            
            
            if (classFile.exists()) {                
                FileInputStream in = null;                
                ByteArrayOutputStream out = null;                
                
                try {                    
                    in = new FileInputStream(classFile);                    
                    out = new ByteArrayOutputStream();                    
                    byte[] buff = new byte[1024];                    
                    int len;                    
                    while ((len = in.read(buff)) != -1) {                        
                        out.write(buff, 0, len);                    
                    }                    
                    
                    return defineClass(className, out.toByteArray(), 0, out.size());               
                } catch (Exception e) {                    
                    e.printStackTrace();                
                }            
            }        
        }   
        
        return null;    
    }
}
```

创建 MyFuDao 类，代码如下：

```java
public class MyFuDao implements MyInvocationHandler {    
    private IPerson target;    
    
    public IPerson getInstance(IPerson target) {        
        this.target = target;        
        Class<?> clazz = target.getClass();        
        return (IPerson) MyProxy.newProxyInstance(new MyClassLoader (), clazz.getInterfaces(), this);    
    }    
    
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
        System.out.println("这里是C语言中文网，已经收集到你的需求，开始挑选");    
    }
}
```

客户端测试代码如下：

```java
public class Test {    
    public static void main(String[] args) {        
        MyFuDao MyFuDao = new MyFuDao();        
        IPerson zhangsan = MyFuDao.getInstance(new ZhangSan()); 
        
        zhangsan.findTeacher();    
    }
}
```

运行结果如下：

这里是C语言中文网，已经收集到你的需求，开始挑选
儿子张三提出要求
双方同意，开始辅导

到这里，手写JDK动态代理就完成了。