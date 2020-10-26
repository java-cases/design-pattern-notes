# 面向对象设计原则

在软件开发中，为了提高软件系统的可维护性和可复用性，增加软件的可扩展性和灵活性，程序员要尽量根据 7 条原则来开发程序，从而提高软件开发效率、节约软件开发成本和维护成本。

面向对象设计原则

1. **[开闭原则（Open Closed Principle）](open_closed_principle.md)：**软件实体应当对扩展开放，对修改关闭（Software entities should be open for extension，but closed for modification）。开闭原则的含义是：当应用的需求改变时，在不修改软件实体的源代码或者二进制代码的前提下，可以扩展模块的功能，使其满足新的需求。
2. **[单一职责原则（Single Responsibility Principle)](single_responsibility_principle.md)：**一个类应该有且仅有一个引起它变化的原因，否则类应该被拆分（There should never be more than one reason for a class to change）。该原则提出对象不应该承担太多职责。核心就是控制类的粒度大小、将对象解耦、提高其内聚性。
3. **[接口隔离原则（Interface Segregation Principle）](interface_segregation_principle.md)：**客户端不应该被迫依赖于它不使用的方法（Clients should not be forced to depend on methods they do not use）。另外一个定义：一个类对另一个类的依赖应该建立在最小的接口上（The dependency of one class to another one should depend on the smallest possible interface）。两个定义的含义是：要为各个类建立它们需要的专用接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。
4. **[里氏替换原则（Liskov Substitution Principle）](liskov_substitution_principle.md)：**继承必须确保超类所拥有的性质在子类中仍然成立（Inheritance should ensure that any property proved about supertype objects also holds for subtype objects）。里氏替换原则通俗来讲就是：子类可以扩展父类的功能，但不能改变父类原有的功能。也就是说：子类继承父类时，除添加新的方法完成新增功能外，尽量不要重写父类的方法。
5. **[依赖倒置原则（Dependence Inversion Principle）](dependence_inversion_principle.md)：**高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象（High level modules shouldnot depend upon low level modules.Both should depend upon abstractions.Abstractions should not depend upon details. Details should depend upon abstractions）。其核心思想是：要面向接口编程，不要面向实现编程。
6. **[迪米特法则（Law of Demeter）](law_of_demeter.md)：**只与你的直接朋友交谈，不跟“陌生人”说话（Talk only to your immediate friends and not to strangers）。其含义是：如果两个软件实体无须直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。
7. **[合成复用原则（Composite Reuse Principle）](composite_reuse_principle.md)：**要求在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现。合成复用原则是通过将已有的对象纳入新对象中，作为新对象的成员对象来实现的，新对象可以调用已有对象的功能，从而达到复用。

