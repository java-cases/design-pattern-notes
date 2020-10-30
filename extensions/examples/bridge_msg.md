# 使用桥接模式设计复杂消息系统

对于什么时候使用桥接模式，大家可能还不是很清楚。举个例子，我们平时工作时经常需要通过邮件消息、短信消息或者系统内消息与同事沟通。尤其在走一些审批流程的时候，需要记录这些过程以详细调查。

根据类型划分，消息可以分为**邮件消息、短信消息和系统内消息**。但是，根据紧急程度来划分，消息可以分为**普通消息、加急消息和特急消息**。显然，整个消息系统可以划分为两个维度。

这种情况如果用继承，实现就会比较复杂，而且也不利于扩展。比如，邮件消息可以是普通的，也可以是加急的。同样，短信消息可以是普通的，也可以是加急的。

下面我们用桥接模式来解决这个问题。

首先创建 IMsg 接口担任桥接的角色，代码如下：

```java
/*** 实现消息发送的统一接口*/
public interface IMsg {       
    void send(String msg, String toUser);
}
```

创建邮件消息 EmailMsg 类。

```java
/*** 邮件消息的实现类*/
public class EmailMsg implements IMsg {    
    public void send(String msg, String toUser) {        
        System.out.println("使用邮件消息发送" + msg + "给" + toUser);    
    }
}
```

创建短信消息 SmsMsg 类。

```java
/*** 短信消息的实现类*/
public class SmsMsg implements IMsg {    
    public void send(String msg, String toUser) {        
        System.out.println("使用短信消息发送" + msg + "给" + toUser);    
    }
}
```

然后创建桥接角色 AbstractMsg 类。

```java
/*** 抽象消息类*/
public abstract class AbstractMsg {    
    //持有一个实现部分的对象    
    IMsg msg;    
    
    //构造方法，传入实现部分的对象    
    public AbstractMsg(IMsg msg) {        
        this.msg = msg;    
    }    
    
    //发送消息，委派给实现部分的方法    
    public void sendMsg(String msg, String toUser) {        
        this.msg.send(msg, toUser);    
    }
}
```

创建具体普通消息 NomalMsg 类。

```java
/*** 普通消息类*/
public class NomalMsg extends AbstractMsg {    
    //构造方法，传入实现部分的对象    
    public NomalMsg(IMsg msg) {        
        super(msg);    
    }    
    
    @Override    
    public void sendMsg(String msg, String toUser) {        
        //对于普通消息，直接调用父类方法发送消息即可        
        super.sendMsg(msg, toUser);    
    }
}
```

创建具体加急消息 UrgencyMsg 类。

```java
/*** 加急消息类*/
public class UrgencyMsg extends AbstractMsg {      
    public UrgencyMsg(IMsg msg) {        
        super(msg);    
    }    
    
    @Override    
    public void sendMsg(String msg, String toUser) {        
        msg = "【加急】" + msg;        
        super.sendMsg(msg, toUser);    
    }    
    
    //扩展它功能，监控某个消息的处理状态    
    public Object watch(String msgId) {        
        //根据给出的消息编码（msgId）查询消息的处理状态        
        //组织成监控的处理状态，然后返回        
        return null;    
    }
}
```

最后编写客户端测试代码。

```java
public class Test {    
    public static void main(String[] args) {        
        IMsg msg = new SmsMsg();        
        AbstractMsg abstractMsg = new NomalMsg(msg);        
        abstractMsg.sendMsg("加班申请速批", "C语言中文网严总");
        
        Msg = new EmailMsg();        
        abstractMsg = new UrgencyMsg(msg);        
        abstractMsg.sendMsg("加班申请速批", "C语言中文网严总");    
    }
}
```

运行结果如下所示。

使用短信消息发送加班申请速批给C语言中文网严总
使用邮件消息发送【加急】加班申请速批给C语言中文网严总

在上面的案例中，我们**采用桥接模式解耦了“消息类型”和“消息紧急程度”这两个独立变化的维度**。

后续如果有更多的消息类型，比如微信、钉钉等，则直接新建一个类实现 IMsg 即可。如果紧急程度需要新增，则同样只需新建一个类继承 AbstractMsg 类即可。