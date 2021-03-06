#面向对象编程

## 基础理论的介绍

"类或继承"描述的是一种代码的组织形式和架构，是对真实世界问题的模型化，而面向对象编程或者说面向类编程强调数据与行为的紧密联系，适当的设计需要把数据和行为捆绑，这在软件开发过程中被称为数据结构的设计。

例如，一长串的字符，我们常称之字符串，字符串是数据，但是，你一定不只是关心你的数据，你可能要对它们做点什么事情吧？所以对数据的操作（计算长度，插入更多数据，查询等等）就是字符串类型`String`中定义的各种方法。

任何一个给定得字符串就是`String`的一个实例，这就意味着我们可以对它进行一系列的处理，类也可称为类别化一定的数据结构，就是我们基于一个普遍的类型而具体化一个特别的类型。

例如，`car`就是基于`Vehicle`更加具体的类了，`Vehicle`中我们可能定义了一些抽象的事物：发动机，承载量，这些都所有的交通工具都适用，假如，我们总是重新定义某个对象能承载多少人，一遍又一遍的定义既显得啰嗦，又是低效的，为何我们不先定义一个基类呢？然后基于这个基类，我们再具体化实例中的值，实现对基类的继承与扩展。

类，继承，实例化就是这么自然的发生了。面向对象的编程理论建议我们基于父类来创建子类，而子类可实现对父类属性和方法的继承，设置复写，利用对象的继承我们可以使得代码更加清晰。

## 类的设计模式

你可能从来没有把类的实现作为一种设计模式，它和遍历，观察者模式，工厂模式一样在面向对象编程中是如此的常见。

## JS的"类"

JS中有`new`,`instanceof`这样的关键字，甚至在ES6中有了`class`，那么JavaScript中是不是真的存在类呢?答案是很明显也很简单，NO。

这一章节主要介绍一些关于传统语言的类，看的不是很懂，先搁置吧！
