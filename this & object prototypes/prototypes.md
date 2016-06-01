# 原型

## [[Prototype]]
JS对象中有个内置的属性：[[Prototype]]，它就是简单的指向另一个对象，每一个对象建立时都会有一个非`null`的[[Prototype]]，当然，我们能实现[[Prototype]]为`null`，虽然它不是那么常用。

```js
var myObject = {
				a: 2
}
myObject.a; //2
```
那么[[Prototype]]是干什么用的呢？第三章中我们介绍了[[Get]]运算符，当我们访问一个对象的属性时，它就会被调用，而它默认的行为就是检查对象本身是否有对应的属性存在，假如有的话，就访问它。

假如，当前对象我们不能得到对应的属性，那又该怎么办呢？这时候[[Prototype]]指向的对象就起到作用了，默认的[[Get]]会紧接着查询[[Prototype]]指向的对象，看是否有我们需要的属性：

```js
var anotherObject = {
				a: 2
};

//create an object linked to 'anothermyObject`
var myObject = Object.create( anotherObject );
myObject.a ; //2
```

我们基于`anotheObject`建立了一个新的对象，也就是说`myObject`的[[Prototype]]是指向`anotherObject`的，当我们访问`myObject.a`时,调用[[Get]]，它首先检查当前对象是否存在属性`a`，很明显没有这么个属性，紧接着，它将在`anotherObject`查到一个属性`a`，并取得值为2。

那如果我们在`anotherObject`上也找不到这么个属性，我们又该怎么办呢？其实是一样的，如果在`anotherObject`上同样不能找到这么个属性，那[[Get]]将继续查询这个对象的[[Prototype]]。

这个流程将一直持续下去，直到对应属性找到，或者原型链的末端，假如最后也没有找到对应的属性，那么[[Get]]将返回`undefined`。

和原型链查询类似，假如利用`for...in`来遍历一个对象，我们可以通过原型链访问到所有能被遍历的属性值（enumerable控制），如果，你通过`in`操作符查询某对象中是否存在某个属性值，`in`会完整的检查对象的整个原型链，不管是否可以遍历（enumerable 不管是真还是假)。

```js
var anotherObject = {
				a: 2
};
//create a object linked to 'anotherObjet `
var myObject = Object.create( anotherObject );
for ( var i in myObject ) {
				console.log( i );
}
//found: a
("a" in myObject ); //true
```


## 对象原型

那么疑问来了，对象的[[Prototype]]止于何处呢？

每一个普通的原型链都止于内建的`Object.prototype`，这个对象包含有JS种一些常用的工具化方法，这些方法中有我们已经了解的`toString`和`valueOf`，还有`hasOwnProperty`等等。

## 属性更改

前面章节我们提到了，对对象添加属性不仅仅就是对它新增一个属性而已，这里我们详细说明下具体的机制：

```js
myObject.foo = "bar";
```

假如`myObject`中已经有一名为`foo`的属性，上述操作就是对已经存在的属性重新赋值，若`foo`不在`myOject`中，就会遍历它的原型链，如果还是没有找打这么个属性，那么会直接在`myObject`直接添加一个属性`foo`，但还有一种情况，假如`foo`存在于某已级别的原型链中呢？

上述`foo`既存在于`myObject`本身，原型链遍历又能找到同样的属性，这种情况我们称之为**shadowing**，`myObject`中的`a`对其他更高级别的`a`有隐藏的作用，也就是说，对某属性的查询总是会找到原型链中对低级别的属性。

我们仔细探讨下：

1. 假如我们能在原型链更高级别中找到可写的（writable: true)，的`foo`，那么`foo`将直接添加到`myObject`上，从而形成对原型链高级别的隐藏。
2. 假如我们在原型链更高级别中找到不可写的（writable: false)的`foo`值，那么我们将不能在原型链和`myObject`上添加对应属性`foo`，严格模式会报错，其他则可能会直接忽略。**这里有问题**
3. 假如我们在高级别的原型链中能找到，并且它是一个`setter`，那么这个方法将被调用，而不会再`myObject`上新增，也不会对`foo setter`重新赋值。

大部分开发者，都会假设说当某属性已经存在原型链中时，而我们在当前在给它赋值就一定会造成对高级原型链中这个属性的遮盖，然后从上面三条规则来看，其实只有第一种情况才如预想一般。假如你想在第二，三中同样修改这个属性的值，`=`将不再适用了，而需要`Object.defineProperty`来帮忙了。

总体上来说，这种遮蔽高级原型链的方法可能会有很多意想不到的问题，我们可能需要尽量避免使用它们，下一章节，我们会学习到更好的设计模式。

## "类"

这里大家可能会猜想，我擦，为什么我要将一个对象链接到另一个对象啊？这对开发真的有什么好处吗？这是一个好问题，但是在回答这个问题之前，我们先要搞清楚：[[Prototype]]是什么鬼！然后才是为什么要这么做！

前面提到，和传统的面向类的语言不通，JavaScript中不存在真正意义上的类，只有对象,只有对象，只有对象。

## "类"函数

JavaScript开发中有一类被开发滥用的模拟真正“类”的方法：

```js
function Foo() {
				//
}

Foo.Prototype; // {}
```

上面的方法被滥用了，那它到底实现了什么呢？所有通过`new Foo()`生成的对象中[[Prototype]]将止于`Foo.Prototype`.

```js
function Foo() {
				//....
}
var a = new Foo();
Object.getPrototypeOf (a ) === Foo.prototype; //true
```

当我们通过`Foo`构造一个新的对象时，新的对象就会内建一个与`Foo.Prototype`指向对象的关联,这到底会有什么影响吗？

在面向类的语言中，我们可以从类或者说模型中建立多个副本（实例），这就是所谓类的作用所在，而JS中，根本就没有这样的行为产生，你根本不能由类来实例化多个副本，而只能建立多个对象，这些个对象的[[Prototype]]都指向一个公共的对象，默认情况下是不会有复制的行为的，这些新建的对象都不是独立的，它们并没有和这个公共的对象脱离，而是紧密联系着的。

我们上述的代码中，新建的`a`[[Prototype]]指向了`Foo.Prototype`。

我们新建了一个对象，二者紧密联系着，这完全不是类的实例，我们没有建立类的副本，只是创建了两个紧密联系的对象而已，实际上，很多开发者避免使用构造函数的真正原因就是：构造函数的调用，并没有直接建立对象间的联系，这仅仅只是个副作用而已，构造的过程是间接地实现了我们创建一个链接到另一对象的新对象而已。

那我们有更直接办法吗？`Object.creat()`是可以的。

## 名字是无关紧要的？

Javascript中，我们没有从类建立实例副本，而是建立了对象之间的链接，视觉上，[[Prototype]]就是从左到右，从上到下的链接过程而已，这经常被称为“原型继承”, 更普遍地称为“类继承”的动态语言版本。这种两边不讨好的说法就是为了让大家借助真正的类来理解JS中的原型继承，然而JS中却不完全是，这很是尴尬。

把原型和继承二者联系起来就如同，我左手拿着一个苹果，右手一个橙子，并坚持说橙子就是一个橙色的苹果而已，不管我们的命名怎么取，这都无关紧要，它并不能改变一个是苹果一个是橙子的事实。

所以，我们还是纯粹的理解“类继承”与“原型继承”的概念，这对他们的联系和区别都是有帮助的，混合着使用是利大于弊的，“类继承”是对类执行一个复制的操作，而原型继承没有对对象进行复制，它建立了两个对象之间的链接，通过这个链接一个对象可以访问另外一个对象的属性和方法,"委托”可能是对JS对象链接机制更好的解释。

另外一个可能接触到的是“差异继承”,通过某个特定的属性来形容差异，而不是把所有的差异全部描述是“差异继承”的主要观点，如果你任意对象都想象成可通过委托的行为的总和，并能在你脑海里扁平化它们会为可触碰的物体，接着你可能就能理解什么是“差异继承”。

同“原型继承一样”，“差异继承”

***需要进一步理解和翻译***


## “构造”

让我们再看看之前的代码段：

```js
function Foo() {
				//..
}
var a = new Foo();
```

是什么对象让我们误以为这是一个“类”呢？

首先可能是`new`关键字，这和传统面向类的语言的语法是一只的，另外有我们确实执行了一个构造方法，这里的`Foo`确实是被调用了，这也和面向类的实例化类没有什么很大的差别。

进一步探讨构造的语义困惑，`Foo.Prototype`可能：

```js
function Foo() {
				//.
}
Foo.Prototype.constructor === Foo; //true

var a = new Foo();
a.constructor === Foo; //true
```

在`Foo`定义时，`Foo.Prototype`默认取得一个属性`constructor`，它是一个索引，链接到它关联对象的方法(这里是`Foo`方法），接着，我们通过构造的方法新建了`a`对象，它也会有一个`constructor`属性，并指向创建它的方法。

**注意**: 其实这并不完全正确，`a`并没有一个属性叫`constructor`，`a.constructor`实际上是导向了`Foo`，"constructor"并不是像它看起来那么“构造”，这些我们后面再讲。

这里有一通废话：我们默认构造函数大写开头，这刚开始是一个便利的使用方法而已，后来，慢慢称为默认的规范，使得语法检查工具在你没有正确使用相关时会报错，如直接调用某大写开头的函数呀，通过`new`调用非首字母大写的函数等等。

### 构造还是调用？

上述代码块很容易使我们觉得`Foo`是一个构造函数，因为通过`new`的调用，确实实现了”构造”的过程，实际上，`Foo`和其他任何一个函数没有什么差别，函数本身并不是构造函数，当用`new`来调用时，这使得函数称为了一个**构造**调用，实际上，`new`是劫持了普通的函数，使得它构造出一个对象：

```js
function NothingSpecial() {
				console.log( "NOooo" );
}
var a = new NothingSpecial();

// "Nooo"
a: // {}
```

这里我们很明显看到，这里的`NothingSpecial`是一个很普通又很简单的函数，但通过`new`来调用它之后，它确实构造出了一个新的对象,刚好赋值给了`a`,这个调用时**构造**调用，但是`NothingSpecial`本身并不是构造函数。

换句话说，JS中，更加适当说法应该是：构造函数是通过`new`来调用的任何一个函数，函数并不是构造函数，只是能通过`new`来实现构造调用。

### 机理

这些就是所有一些关于类的讨论吗？

那可就不止了，JS开发者费大力地想尽可能地模拟出面向类的概念：

```js
function Foo(name) {
				this.name = name;
}
Foo.prototype.myName = function() {
				return this.name;
}
var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); //a
b.myName(); //b
```

上述代码块存在两个"类导向"在其中：

1. `this.name = name`：新增`name`属性到每一个对象，这和类实例很相似。
2. `Foo.prototype.myName = ...`：它在`Foo.prototyep`对象上添加了`myName`方法，因此`a.myName()`能被调用，但是它是怎么实现的呢？

`a b`很容易理解成，新建了一个`a b`的对象，`Foo.prototype`上的属性都复制给了`a`, `b`对象，然而，这并没有发生。章节开篇就说道[[Prototype]]链，它在某对象上不能找到某属性时，是怎么处理的。不同于复制`Foo.prototype`，当`a`,`b`建立时，它们的[[Prototype]]链接到了`Foo.prototype`对象，当`myName`在`a`,`b`上找不到时，会通过原型链在`Foo.prototype`上找到，一层一层知道找到或不存在为止。

### 构造redux

我们之前讨论过`constructor`属性时，`a.constructor === true`会返回**true**，这就意味着`a`实际有个属性`constructor`，并指向`Foo`呢？不对，不对！

这很是让人疑惑，`constructor`同样是指向`Foo.prototype`，碰巧（默认）有个`constructor`指向函数`Foo`。通过`Foo`构造出的`a`可以通过`a.constructor`访问到`Foo`，看起来很是方便，但是这仅仅是一个巧合，`a.constructor`通过[[Prototype]]碰巧指向了`Foo`,会存在各种不同的情况使得这种看起来很美的设定，让你掉坑里，爬都爬不出来。

举个栗子，当`Foo`声明时，默认会在其`prototype`上添加`constructor`属性，当你新建一个对象时，并替换掉函数默认的`prototype`指引，那么新的对象是不会神奇的获取到`constructor`属性的。

```js
function Foo() { }
Foo.prototype = {} //create a new prototype object
va al = new Foo();
al.constructor === Foo; //false!
al.constructor === Object; //true
```

`Object()`可没有构造`a1`呀！从表明上看，`Foo`很明确的是构造了`a1`，很多开发者都会认为`Foo`构造了`a1`，然而，当你认为`constructor`就是意味着**由谁构造**的话，事情就变得不是那么正确，如果`construtor`就是由谁构造的话，上述代码块中的`al.constructor`应该就是`Foo`，然而并不是的。

这是什么鬼呢？`al`并没有`constructor`属性，沿原型链向上查询到`Foo.prototype`，但是它也没有这个属性（当然，默认是会有的，只是我们这里复写了），继续沿原型链查询，知道原型链顶端`Object.prototype`，它有个`constructor`属性，指向内建的`Object`函数。

当然我们可以给`Foo.prototype`添加`constructor`属性，但是这需要一些代码来实现，尤其是需要这个属性匹配默认的行为并不被其他不知道的谁影响：

```js
function Foo() { /*somethig here */ }
Foo.prototype = { /* create a new prototype object */ };
//`Oject.defineProperty()`修复`Foo.prototype`上
//消失的`prototype`属性值
Object.defineProperty(Foo.prototype, "constructor", {
				enmuerable: false,
				writable: true,
				configurable: true,
				value: Foo //指向`Foo`的`constructor`属性
});

```

这花费了很多功夫来定义`constructor`属性，但是通过这一段，我们明白了：不能将**constructor**和**构造某对象的函数**混为一谈的。










 




























































