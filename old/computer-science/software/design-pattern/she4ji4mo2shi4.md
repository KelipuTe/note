## 设计模式

- 2021-11-02

### 设计模式是什么

设计模式（design pattern）是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。

使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。 项目中合理地运用设计模式可以完美地解决很多问题。

每种模式在现实中都有相应的原理来与之对应，每种模式都描述了一个在我们周围不断重复发生的问题，以及该问题的核心解决方案。

这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

### 设计模式的分类

#### 创建型模式

创建型设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用new运算符直接实例化对象。

这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。

- 工厂模式（factory pattern）
- 抽象工厂模式（abstract factory pattern）
- 单例模式（singleton pattern）
- 建造者模式（builder pattern）
- 原型模式（prototype pattern）

#### 结构型模式

结构型设计模式关注类和对象的组合。

继承的概念被用来组合接口和定义组合对象获得新功能的方式。

- 适配器模式（adapter pattern）
- 桥接模式（bridge pattern）
- 过滤器模式（filter、criteria pattern）
- 组合模式（composite pattern）
- 装饰器模式（decorator pattern）
- 外观模式（facade pattern）
- 享元模式（flyweight pattern）
- 代理模式（proxy pattern）

#### 行为型模式

行为型设计模式特别关注对象之间的通信。

- 责任链模式（chain of responsibility pattern）
- 命令模式（command pattern）
- 解释器模式（interpreter pattern）
- 迭代器模式（iterator pattern）
- 中介者模式（mediator pattern）
- 备忘录模式（memento pattern）
- 观察者模式（observer pattern）
- 状态模式（state pattern）
- 空对象模式（null object pattern）
- 策略模式（strategy pattern）
- 模板模式（template pattern）
- 访问者模式（visitor pattern）

### 设计模式几个原则

#### 单一原则

单一原则（single responsibility principle）：一个类或者一个方法只负责一项职责。

#### 开闭原则

开闭原则（open close principle）：对扩展开放，对修改关闭。

在程序需要进行拓展的时候，不能去修改原有的代码。

#### 里氏代换原则

里氏替换原则（liskov substitution principle）：任何基类可以出现的地方，子类一定可以出现。

子类可以扩展父类的功能，但不能改变原有父类的功能。

里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，里氏替换原则是对实现抽象化的具体步骤的规范。

#### 依赖倒置原则

依赖倒置原则（dependence inversion principle）：抽象不应该依赖细节，细节应该依赖抽象。

上层模块不应该依赖下层模块，两者应依赖其抽象。

依赖倒置原则是开闭原则的基础。

#### 接口隔离原则

接口隔离原则（interface segregation principle）：客户端不应该被强迫实现一些他们不会使用的接口。

复杂的接口，根据业务拆分成多个简单接口，降低类之间的耦合度。

接口的设计粒度越小，系统越灵活，但是灵活的同时结构复杂性提高，开发难度也会变大，维护性降低。

#### 迪米特法则

迪米特法则（law of demeter）：一个对象应当对其它对象有尽可能少的了解。

又称最少知道原则。尽量降低类与类之间的耦合。

#### 合成复用原则

合成复用原则（composite reuse principle）：尽量使用组合或聚合的方式，而不是使用继承。