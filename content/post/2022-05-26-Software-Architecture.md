---                                                                                     
author: "fishwings"                                                                     
date: 2022-05-26                                                                        
title: "软件设计原则 1"                                                    
tags: [                                                                                 
    "软件架构",                                                                               
]                                                                                       
categories: [                                                                           
    "软件设计",                                                                               
]                                                                                       
---                                                                                     
<feff># 软件设计原则1

# 1. 设计原则-简单设计四个基本原则

## 1.1 通过所有测试

软件系统需求要正确完成，包含功能需求和非功能需求，要通过客户验收。

## 1.2 尽可能消除重复

 代码要高内聚，低耦合，有良好的正交性

不是所有重复都可以消除，最小化重复

## 1.3 尽可能清晰表达

漂亮代码如同优美的散文，不隐藏设计者的意图，抽象恰当，控制简洁直接

代码被阅读的次数远远大于修改次数

## 1.4 更少的代码元素

尽可能降低设计的复杂度，保持简单。

# 2 正交四原则

## 2.1 最小化重复

重复即耦合，需要修改，很可能漏改。

## 2.2 分离变化

识别变化方向并预留变换的扩展接口。

## 2.3 缩小依赖范围

最小化知识原则，依赖接口，接口尽可能简洁

## 2.4 向稳定方向依赖

在使用者角度去定义API，关注what，而不是why

# other

1. 设计模式是用来解决同一问题不同表相的。设计模式的两大主题是系统的复用和拓展。

## 23 种设计模式

创建型：

1. Factory Method：隔离创建对象细节，使得创建对象的行为可拓展。
2. Abstract Factory：模式抽象出创建一组相关对象的接口，其中每个方法为factory method
3. Builder：与factory 不同点，该模式包含了对象构造的若干过程，天然和template结合
4. Prototype: 用于某个对象为模 创建一个新对象的场景，例如幻灯片的复制，对象的clone
5. Singleton：全局只创建一个对象。

结构型

1. Adapter Class/Object： 处理遗留系统的好手段，用于用空方法实现接口作为抽象父类

适配器模式：
场景：现有类接口不满足需求。复用现有类，并使不兼容类一起工作。复用现有子类，通过对象适配器来适配父类接口。
优点：提升系统的拓展性，避免现有代码的修改。
缺点：过度使用适配器，系统变凌乱。

1. Bridge：使用关联替代继承，解决类多维度扩展的类爆炸
2. Composite：将组件组装为整体使用
3. Decorator： 常用各种wrapper，用于在原函数执行前后做额外的工作
4. Facade：封装扇出，利用树形结构减少调用者的复杂度
    
    外观模式：
    场景：为一个复杂的系统对外提供统一接口。对子系统进行分层，并简化层次间的接口依赖。
    优点：降低了系统间及系统和外部的耦合度，提升安全性。简化了系统的接口，更易于使用系统。
    缺点：对同一接口的使用可能会引入较多约束，降低了系统的灵活性。不易于修改，子系统的变动可能会引起外观层甚至客户端的改动，违背开闭原则。
    
5. Flyweight：复用变化少的对象

享元模式
解释；主要用于减少创建对象的数量，减少内存占用和提高性能。它提供了减少数量改善应用所需的对象结构的方式。
优点：大大减少对象的创建，降低系统的内存，使效率提高。
缺点：提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态是可以固化的性质，不随着内部状态的变化而变化。

1. Proxy：原对象的一个完整的替代品

行为型

1. Interpreter：解释执行自定义的某种语法
2. Template Method：框架和钩子
3. Chain of Responsibility：一组对象按照既定的顺序关联依次处理，其中任意对象都有权停止调用传递
4. Command：将行为抽象和解耦
5. Iterator：封装数据的访问行为（顺序和可见性）
6. Mediator：用一个中介对象来封装一系列的交互；新增一个模块处理两个模块的交互
7. Observer：订阅/发布模型，用于事件驱动的设计
8. Stat：封装FSM的状态和状态迁移，每个状态定义了自身的输入与状态迁移
9. Strategy：使用接口即使用strategy，用于隔离变化
10. Visitor：数据与行为分离方法。可做到一个被访问者动态添加新的操作而无需其他的修改                                                     
