# 软件设计模式概述

设计模式（Design Pattern）是前辈们**对代码开发经验的总结，是解决特定问题的一系列套路**。它不是语法规定，而是**一套用来提高代码可复用性、可维护性、可读性、稳健性以及安全性的解决方案**。

1995 年，GoF（Gang of Four，四人组/四人帮）合作出版了**《设计模式：可复用面向对象软件的基础》一书，共收录了 23 种设计模式，从此树立了软件设计模式领域的里程碑，人称「GoF设计模式」**。

这 23 种**设计模式的本质是面向对象设计原则的实际运用，是对类的封装性、继承性和多态性，以及类的关联关系和组合关系的充分理解**。

当然，软件设计模式只是一个引导，在实际的软件开发中，必须根据具体的需求来选择：

- 对于简单的程序，可能写一个简单的算法要比引入某种设计模式更加容易；
- 但是对于大型项目开发或者框架设计，用设计模式来组织代码显然更好。

## 产生背景

“设计模式”这个术语最初并不是出现在软件设计中，而是被用于建筑领域的设计中。

1977 年，美国著名建筑大师、加利福尼亚大学伯克利分校环境结构中心主任克里斯托夫·亚历山大（Christopher Alexander）在他的著作《建筑模式语言：城镇、建筑、构造（A Pattern Language: Towns Building Construction）中描述了一些常见的建筑设计问题，并提出了 253 种关于对城镇、邻里、住宅、花园和房间等进行设计的基本模式。

1979 年他的另一部经典著作《建筑的永恒之道》（The Timeless Way of Building）进一步强化了设计模式的思想，为后来的建筑设计指明了方向。

1987 年，肯特·贝克（Kent Beck）和沃德·坎宁安（Ward Cunningham）首先将克里斯托夫·亚历山大的模式思想应用在 Smalltalk 中的图形用户接口的生成中，但没有引起软件界的关注。

直到 1990 年，软件工程界才开始研讨设计模式的话题，后来召开了多次关于设计模式的研讨会。

1995 年，艾瑞克·伽马（ErichGamma）、理査德·海尔姆（Richard Helm）、拉尔夫·约翰森（Ralph Johnson）、约翰·威利斯迪斯（John Vlissides）等 4 位作者合作出版了《设计模式：可复用面向对象软件的基础》（Design Patterns: Elements of Reusable Object-Oriented Software）一书，在本教程中收录了 23 个设计模式，这是设计模式领域里程碑的事件，导致了软件设计模式的突破。这 4 位作者在软件开发领域里也以他们的“四人组”（Gang of Four，GoF）匿名著称。

直到今天，狭义的设计模式还是本教程中所介绍的 23 种经典设计模式。

## 软件设计模式的概念与意义
有关软件设计模式的定义很多，有些从模式的特点来说明，有些从模式的作用来说明。本教程给出的定义是大多数学者公认的，从以下两个方面来说明。

#### 1. 软件设计模式的概念

软件设计模式（Software Design Pattern），又称设计模式，是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。它描述了在软件设计过程中的一些不断重复发生的问题，以及该问题的解决方案。也就是说，它是解决特定问题的一系列套路，是前辈们的代码设计经验的总结，具有一定的普遍性，可以反复使用。其目的是为了提高代码的可重用性、代码的可读性和代码的可靠性。

#### 2. 学习设计模式的意义

设计模式的本质是面向对象设计原则的实际运用，是对类的封装性、继承性和多态性以及类的关联关系和组合关系的充分理解。正确使用设计模式具有以下优点。
- 可以提高程序员的思维能力、编程能力和设计能力。
- 使程序设计更加标准化、代码编制更加工程化，使软件开发效率大大提高，从而缩短软件的开发周期。
- 使设计的代码可重用性高、可读性强、可靠性高、灵活性好、可维护性强。

当然，软件设计模式只是一个引导。在具体的软件幵发中，必须根据设计的应用系统的特点和要求来恰当选择。对于简单的程序开发，苛能写一个简单的算法要比引入某种设计模式更加容易。但对大项目的开发或者框架设计，用设计模式来组织代码显然更好。

## 软件设计模式的基本要素
软件设计模式使人们可以更加简单方便地复用成功的设计和体系结构，它通常包含以下几个基本要素：模式名称、别名、动机、问题、解决方案、效果、结构、模式角色、合作关系、实现方法、适用性、已知应用、例程、模式扩展和相关模式等，其中最关键的元素包括以下 4 个主要部分。

#### 1. 模式名称
每一个模式都有自己的名字，通常用一两个词来描述，可以根据模式的问题、特点、解决方案、功能和效果来命名。模式名称（PatternName）有助于我们理解和记忆该模式，也方便我们来讨论自己的设计。

#### 2. 问题
问题（Problem）描述了该模式的应用环境，即何时使用该模式。它解释了设计问题和问题存在的前因后果，以及必须满足的一系列先决条件。

#### 3. 解决方案
模式问题的解决方案（Solution）包括设计的组成成分、它们之间的相互关系及各自的职责和协作方式。因为模式就像一个模板，可应用于多种不同场合，所以解决方案并不描述一个特定而具体的设计或实现，而是提供设计问题的抽象描述和怎样用一个具有一般意义的元素组合（类或对象的 组合）来解决这个问题。

#### 4. 效果
描述了模式的应用效果以及使用该模式应该权衡的问题，即模式的优缺点。主要是对时间和空间的衡量，以及该模式对系统的灵活性、扩充性、可移植性的影响，也考虑其实现问题。显式地列出这些效果（Consequence）对理解和评价这些模式有很大的帮助。

主要参考《Java设计模式：23种设计模式全面解析》，仅作学习交流用。http://c.biancheng.net/view/1317.html

# 目录

* [设计模式简介](README.md)
  * [GoF 的 23 种设计模式](gof_23_classification.md)
* [面向对象设计原则](oo_principle/README.md)
  * [开闭原则（Open Closed Principle）](oo_principle/open_closed_principle.md)
  * [单一职责原则（Single Responsibility Principle)](oo_principle/single_responsibility_principle.md)
  * [接口隔离原则（Interface Segregation Principle）](oo_principle/interface_segregation_principle.md)
  * [里氏替换原则（Liskov Substitution Principle）](oo_principle/liskov_substitution_principle.md)
  * [依赖倒置原则（Dependence Inversion Principle）](oo_principle/dependence_inversion_principle.md)
  * [迪米特法则（Law of Demeter）](oo_principle/law_of_demeter.md)
  * [合成复用原则（Composite Reuse Principle）](oo_principle/composite_reuse_principle.md)
* [创建型( Creational)](creational/README.md)
  * [抽象工厂模式(Abstract Factory)](creational/abstract_factory.md)
  * [建造者模式（Builder)](creational/builder.md)
  * [工厂方法模式( Factory Method)](creational/factory_method.md)
  * [原型模式（Prototype)](creational/prototype.md)
  * [单例（Singleton）](creational/singleton.md)
* [结构型(Structural)](structural/README.md)
  * [适配器模式（Adapter）](structural/adapter.md)
  * [桥接模式( Bridge）](structural/bridge.md)
  * [组合模式（Composite）](structural/composite.md)
  * [装饰模式(Decorator）](structural/decorator.md)
  * [外观模式( Facade）](structural/facade.md)
  * [享元模式(Flyweight）](structural/flyweight.md)
  * [代理模式(Proxy）](structural/proxy.md)
* [行为型(Behavioral)](behavioral/README.md)
  * [职责链模式( Chain of Responsibility）](behavioral/chain_of_responsibility.md)
  * [命令模式( Command）](behavioral/command.md)
  * [解释器模式(Interpreter）](behavioral/interpreter.md)
  * [迭代器模式(Iterator）](behavioral/iterator.md)
  * [中介者模式（Mediator）](behavioral/mediator.md)
  * [备忘录模式（Memento）](behavioral/memento.md)
  * [观察者模式(Observer）](behavioral/observer.md)
  * [状态模式(State）](behavioral/state.md)
  * [策略模式(Strategy）](behavioral/strategy.md)
  * [模板方法模式(Template Method）](behavioral/template_method.md)
  * [访问者模式(Visitor）](behavioral/visitor.md)
* [其他模式(Other)](other/README.md)

