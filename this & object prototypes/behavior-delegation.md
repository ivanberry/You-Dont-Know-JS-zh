## delegation行为

上一章节中，我们强调了[[Prototype]]的一些基础性原理，实现机制等等，也阐述了多年来对它关于“类”或者“继承”的一些误解，它好像一个魔盒一样的帮我们完成一些事情，但是，就学习应用的角度而言，我们应该深入学习它，在这一章里，我们来拉开原型的神秘面纱，从本质上掌握它，从而更好地，更直接多利用原型，这无疑是有利于我们的开发，甚至是代码维护工作的。

[[Prototype]]的机制是：某对象上指向另一对象的索引。

这种关系主要存在于对某对象的某一个属性或方法查询时，但是这个对象本身又没有对应的属性或方法，这种情况下，[[Prototype]]就会站出来帮忙，从我这个楼梯走到你关联的另一个对象，看它那有没有你要的这个属性或方法，假如，在这个对象同样也没有找到，那么它本身的[[Prototype]]也会站出来，来，来，来，从我这个走，到你自己关联的对象上去找找。这些对象间的联系我们就称为原型链。

换句话说，原型链最关键本质的就是对象关联。

## 面向delegation的设计

为了更直接使用[[Prototype]]，首先，我们必须明确，它同面向类设计的差异！建立好这种思维后，我们才能最有效的利用好的[[Prototype]]。

### 类

这里是关于类的一些基本原理，我就自己看看原文了，就不做翻译，因为我也不是很懂，只提最关键的一点：类的实例是复制。

### delegation

假设我们有一系列相同的任务，我们需要对它们模块化，上面我们省略掉了基于类的设计，这里细细学习下基于delegation的设计：

首先，我们定义一个叫`Tasks`的对象，它包含一些常用的方法，它们可以用来delegation to，接着，为每一个任务新建一个对象，它们包含自己的特有的任务或数据等等，同时，通过与`Tasks`对象的关联，我们可以访问到定义在`Tasks`上的那些常用方法：

```js
var Task = {
				setID: function(ID) {
								this.id = ID;
				},
				outputID: function() {
								console.log( this.id );
				}
};

//make XYZ delegation to Task

var XYZ = Object.create( Task );
XYZ.prepareTask = function( ID, Label ) {
				this.setID( ID );
				this.label = Label;
};

XYZ.outputTaskDetails = function() {
				this.output( ID );
				console.log( this.label );
};

```

上述代码中：`Task`和`XYZ`都不是类，它们都仅仅只是一个对象而已，其中`XYZ`通过`Object.create()`至`[[Prototype]]``代理到`Task`对象。

我们可以称这种和传统面向对象的编码为面向关联的编码（这原作者也是脑洞大开），简称(OLOO)，我们真正关心地是`XYZ`对象代理到`Task`。

JavaScript中，[[Prototype]]机制使得对象间建立了联系，这种方式没有面向对象的那些抽象概念，不管你怎么说服自己，只不过是在用复杂的方法来实现一个本来就很自然的功能而已，用咱中国人话说：你这是费力不讨好啊！啊哈哈！

正经点说：

1. `ID`和`Label`都是实实在在存在于代理(`XYZ`和`ABC`)，而不是在`Task`上的。
2. 面向对象设计时，实例常会定义和类相同的函数名，以此可以覆盖类上的方法，OLOO这是相反地，基于代理机制，我们尽可能定义不同方法，这可能没有很明显的好处，但是，代码的维护和可读性就大大地提高了。
3. `this.setID()`，在`XYZ`对象上调用`setID`方法：哟，这个对象上木有，[[Prototype]]意味着要去`Task`看看，哟，还真有呀！这里，根据`this`的调用机制，很明显是绑定到XYZ对象上的（不记得呢？你丫的回去复习复习this绑定规则1234😡)。

代理方法：在某对象上没有找到属性或方法时，通过代理，以索引的方式链接到另外的对象。

### 调试

JS的调试就是要利用好浏览器，因为没有标准的方法，而每个浏览器又都有自己的一套规则，或者说技术，这使得我们对JS的调试都不那么统一：

```js
function Foo() {}
var a1 = new Foo();
a1; //Foo{}
```
最后一行代码在Chrome和Firefox中会有差异，我们就不深究差异何处，我们仔细研究Chrome的表现吧！

```js
function Foo() {}
var a1 = new Foo();
a1.constructor; //Foo(){};
a1.constructor.name; //Foo
```

从这里我们就有疑惑了，是不是我们访问`a1`时，Chrome就只是访问了它的`constructor.name`属性呢？令人不解的是，有时是这么回事，有时又不是这么回事的。

```js
function Foo() {}
var a1 = new Foo();
Foo.prototype.constructor = function Gotcha() {};
a1.constructor; //Gotcha(){};
a1.constructor.name; //Gotcha
a1; //Foo {}
```

依然返回的是`Foo`，这说明并不是直接获取`constructor.name`，所以好像看起来对之前的问题有了答案了，且慢，再看看下面**OLOO**风格的代码：

```js
var Foo = {};
var a1 = Object.create( Foo );
a1; //Object
Object.defineProperty( Foo, 'constructor', {
				enumerable: false,
				value: function Gotcha() {}
});
a1; //Gotcha
```
原作者说，这里返回`Gotcha`是个bug，说以后版本会改，但是我现在还是一样的，其实这些知识是一些琐碎的细节，了解了解就好的。

### 比较

到现在为止，我们从原理层面上了解了“类”和“代理”的一些差异，接下来看看它们：

面向“类”（原型）：

```js
function Foo(who) {
				this.me = who;
}

Foo.prototype.identify = function() {
				return "I am " + this.name;
};

function Bar(who) {
				Foo.call( this, who );
}

Bar.prototype.speak = function() {
				console.log( "Hello, " + this.identify() + ".");
};

var b1 = new Bar( "b1" );
var b2 = new Bra( "b2" );
b1.speak();
b2.speak();
```
这里有顶层父类`Foo`，`Bar`继承于它，而由`Bar`“构造”出`b1`,`b2`。`b1`代理到`Bar.prototype`，而`Bar.prototype`代理到`Foo.prototype`上。

**OLOO**风格：

```js
var Foo = {
				init: function(who) {
								this.me = who;
				},
				identify: function() {
								return "I am " + this.me;
				}
};

var Bar = Object.create( Foo );
Bar.speak = function() {
				console.log( "Hello, " + this.identify() + ".");
};
var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );
b1.speak();
b2.speak();
```

同样，我们建立起了`b1`-->`Bar`-->`Foo`间的联系，和OO风格一样，但是我们简化了整个代码，仅仅只是建立了这个三个对象的链接，而且看起来也不想类（本身就不是类，只是模拟了一下而已）。

我们看看二者的模型：

![模拟类的模型](./fig4.png)

![模拟类的简化模型](./fig5.png)

![OLOO模型](./fig6.png)











