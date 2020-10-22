# gof - relationships of 23 design patterns

## 设计模式关系图

![[design-patters-relationships.png]](http://www.4e00.com/blog/img/java/design-patterns/gof-23-design-patterns.png)

## 设计模式中设计的可变方面

|    目的    |                           模式描述                           |                           设计模式                           |                          可变的方面                          |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 创建型模式 | 这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。 | 抽象工厂模式（Abstract Factory）建造者模式（Builder）工厂方法模式（Factory Method）原型模式（Prototype）单例模式（Singleton） | 产品对象家族如何创建一个组合对象被实例化的子类被实例化的类一个类的唯一实例 |
| 结构型模式 |               这些设计模式关注类和对象的组合。               | 适配器模式（Adapter）桥接模式（Bridge）组合模式（Composite）装饰器模式（Decorator）外观模式（Facade）享元模式（Flyweight）代理模式（Proxy） | 对象的接口对象的实现一个对象的结构和组成对象的职责，不生成子类一个子系统的接口对象的存储开销如何访问一个对象；该对象的位置 |
| 行为型模式 |             这些设计模式特别关注对象之间的通信。             | 责任链模式（Chain of Responsibility）命令模式（Command）解释器模式（Interpreter）迭代器模式（Iterator）中介者模式（Mediator）备忘录模式（Memento）观察者模式（Observer）状态模式（State）策略模式（Strategy）模板方法模式（Template Method）访问者模式（Visitor） | 满足一个请求的对象何时、怎样满足一个请求一个语言的文法及解释如何遍历、访问一个聚合的各元素对象间怎样交互、和谁交互一个对象中哪些私有信息存放在该对象之外、以及在什么时候进行存储多个对象依赖于另外一个对象，而这些对象又如何保持一致对象的状态算法算法中的某些步骤某些可作用于一个（组）对象上的操作，但不修改这些对象的类 |

## References

1. GoF - Design Patterns: Elements of Reusable Object-Oriented Software



 参考：http://www.4e00.com/blog/java/2017/09/23/GoF-relationships-of-23-design-patterns.html