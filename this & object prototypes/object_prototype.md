# 对象

介绍我们的`this`值时，多次提到了对象这个鬼，然而，对象到底是什么鬼呢？这一部分，我们就好好来说道说道。

## 语法

对象有两种形式：对象字面量和构造对象，其中字面量形式为：

```js
var myObj = {
				key: value,
				key: value
};
```

构造形式：

```js
var myObj = new Object();
myObj.key = value;
```

两种方式最终的对象是一样的，没有什么差别，仅仅存在的一点差异是，对象字面量可以同时添加对个对象属性，而，通过构造的对象只能一条一条的添加。需要了解的是，其实，真正应用上，我们很少通过构造的形式来声明一个新的对象。

## 类型

对象是JS中6种数据类型的一种：

- striing
- number
- boolean
- null
- undefined
- object

其中简单数据类型(`string`,`number`,boolean`,`null`,`undefined`)本身不是对象，`null`有时是指一个对象，但是这是来自于Javascript语言的`typeof null = Object`的bug。实际上`null`属于简单数据类型。

有一个很普遍的错误认识：**一切皆对象**

相对而言，确实存在一些对象子类，可以称为复杂数据类型，其中最典型的就是`function`，它是一种可调用的对象，函数是JS中一等公民，它们也是普通的对象，可以被调用，正因如此，我们同样可以向普通对象一样看待它。

## 内建对象

语言中有一些内建对象，可以认为是对象的一些子类，它们从字面上看，跟对应的简单数据类型神似，实际上却比简单数据类型复杂一些：

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

Js中，这些通常是语言内建的函数，它们多用做构造函数，通过`new`调用它们后，会定义一个新的对应的子类对象：

```js
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false
var strObject = new String( "I am a string" );
typeof strObject; //Object

strObject instanceof Object; //true
//查看对象子类型
Object.prototype.toString.call( strObject ); //[Object String]
```

之后我们在详细介绍`Object.prototype.toString...`，这里简单来说，通过这个方法我们可以查看`strObject`的对象子类，它告诉我们`strObject`是通过`String`构造的一个对象。

`"I am string"`不是对象，它是基本数据类型，并且不能被修改，要想对它有一些运算操作，例如检查它的长度，单独的字符等等，就必须先得是`String`类型才行。

幸运的是，JS在必要的时候自动帮我把基本数据类型的数据转为对象类型，然后在还原，也就是说我们不要另外在建立对象字符的对象形式来完成我们的运算操作，正因为此，大多数JS社区强烈推荐我们使用对象字面量来声明，定义对象，而不是对象构造函数：

```js
var strPrimitive = "I am a string ";
console.log(strPrimitive.length); 	//13
console.log(strPrimitive.charAt( 3 ); //"m"
```
如上，当我们对字符类数据有属性和方法等操作是，JS会自动的将它转换为`String`对象，因此方法和属性才能有效,同样地，对于其他基本数据类型值一样会发生以上的自动转换。

`null`和`undefined`没有对象类型，它们就是基础类型，`Date`只能通过构造函数声明，它没有对应得字面量形式。

`Object, Array,Function RegExp`既能通过构造声明又能通过字面量声明，根据最佳实践，只有当我们需要定义额外的信息是才采用构造的方式，其他一般多用字面量形式。

`Error`对象很少在代码中声明，更多的是引擎抛出,它可以通过`new Error(...)`来构造。

## 内容

我们上面有提到，对象内容可为任意类型的键值对，称为属性。

根据对象的形式，我们很容易联想到：既然是内容，那么这些键值对，数据什么的肯定是存储在对象内部的。其实这只是表象而已。实际保存在对象中的只是这些属性名而已，它们充当的是指针，指向数据真正保存的位置。

```js
var myObject = {
				a:2
};

myObject.a; //2
myObject["a"]; //2
```
获取对象属性的方法有两种`.`和`[]`，它们之间的区别或者优劣就不多说了，基本上能能使用`.`的地方可以使用`[]`，不能使用`.`的地方还是能使用`[]`。需要提到的一句：对象中属性名总是以字符串的形式出现，也就是说。不论你定义其他非字符串的属性名，都会自动转换为字符串，这包括数值。

```js
var myObject = {};
myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";
myObject["true"]; //foo
myObject["3"]; //bar
myObject["["object object"]"]; //baz
```

### 拼装属性名

ES6中提供了类似字符拼装的属性名定义方法：

```js
var prefix = "foo";
var myObject = {
				[prefix + "bar"]: "hello",
				[prefix + "baz"]: "world"
};
```

## 属性VS方法

JS语言中对属性和方法的区别是没有什么大的必要的，当访问对象的某个属性时就是属性访问，不管这会返回什么样的数据，假如刚好返回的是一个函数，这也没有必要说是一个方法访问，两者称谓的差别是没有什么很大意义的。

```js
function foo() {
				console.log( "foo" );
}

var someFoo = foo; // variable reference to 'foo'
var myObject = {
				someFoo: foo
};

foo; // function foo() {}
someFoo; //function foo() {}
myObject.someFoo; //function foo() {}
```
`someFoo`和`myObject.someFoo`是对相同函数的两个不同独立的索引，这并没有使得函数绑定到了某个对象，它们的区别可能在于可能存在函数中`this`值得隐性绑定。

就算是在对象中定义函数表达式，函数也不一定是属于这个对象的，我们还是可以通过不同索引访问到它：

```js
var myObject = {
				foo: function foo() {
								console.log( "foo" );
				}
};

var someFoo = myObject.foo;

someFoo; //function foo() {..}
myObject.foo; //function foo() {...}
```

## 数组

数组是通过数字`[]`来获取值的，这意味着数组值是存放在某个位置---索引，我们通过非负整数来访问它们。

```js
var myArray = [ "foo", 42, "bar"];

myArray.length; //3
myArray[0]; //foo
myArry[2]; //bar
```

尽管数组是通过索引值来访问的，我们依然可以在数组中加入属性名：

```js
var myArray = ["foo", 32, "bar"];
myArray.baz = "baz";
myArray.length; //3
myArray.baz; // "baz"
```

## 对象复制

开发者可能最需要的对对象的一个处理是对对象的复制，看起来我们需要一个内建的`copy`函数，然而并没有这么厉害，因为对对象的复制不是一个简单的事情，它可能涉及到复制的层级，这就需要某种算法来实现，某种程度上的对象复制了。

```js
function anotherFunction() { /*...*/ }

var anotherObject = {
				c: true
};
var anotherArray = [];
var myObject = {
				a: 2,
				b: anotherObject, //refrence, not a copy
				c: anotherArray, //another refrence
				d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

要复制一个对象：

首先，我们应该明确我们是要“浅浅的”表层复制还是“深深地”深度复制呢？如上列，表层复制仅仅会对基本数据类型值进行拷贝，而对复制数据类型等则是新建一个索引，指向对应的值，而深度复制是对复制数据类型也复制，而上述代码中，我们的`anotherArray`里推入了`myObject`，这里我们就有一个复制循环了，这该不如是好呢？是陷入无限循环的噩梦？还是直接报错“你妹啊，这玩意叫我怎么深度给你复制，内存不够”，或者屈服二者

另外，复制一个函数是有歧义的。

因此，我们怎么解决这些问题呢？不同的JS框架都有自己的一套解决办法，但是，谁会是标准的做法呢？长时间来，都没有一个定论。一个相对好一点的做法是：JSON-safe，将对象序列化为JSON字符，然后在解序列化，这很容易实现对对象的复制：

```js
var newObject = JSON.parse( JSON.stringify( someObject ) );
```

当然，这种方法需要我们明确你的对象是JSON-safe的，但是，它也不是万能的，有些情况下不是那么有效。

同时，表层复制是一个相对容易接受的复制过程，相对有更好的问题，因此，ES6中给出了`Object.assign(..)`来实现表层复制。它接受个目标对象为第一参数，一个或多个源对象为第二，三个更多的参数,这个方法会遍历*enumerable*(能通过`for .. in..`访问到)，直接的属性值键值，通过`=`复制源对象给目标对象，最终方法返回目标对象。

```js
var newObj = Object.assign( {}, myObject );

anotherObj = {
				a: 1,
				b: "Object.assign method",
				c: "pretty nice WOW!"
};

myObj = {
				a: 2,
				d: "Nice try",
				e: anotherObj
};

newObj.e === anotherObj; //true
```

这个方式很是厉害，值得学习，下一章节，我们学习属性的，并能通过`Object.definedProperty(..)`，来实现更深层次的复制。


## 属性类型

ES5中，属性有了自己的类型，不同的类型可能会有不同的特性：

```js
var myObject = {
				a: 2
};
Object.getOwnPropertyDescriptor( myObject, "a");
//{
				value: a,
				writable: true,
				enumerable: true,
				configurable: true
//}

如上，通过上述几个描述符，我们知道对象某个值不仅仅有`value`为`2`，还有其他的描述符，通过`Oject.definedProperty(..)，我们可以新增或修改对象属性的类型。

```js
myObject = {
				Object.definedProperty( myObject, "a", {
								value: 2,
								writable: true,
								configurable: true,
								enumerable: true
				});

myObject.a; //2
```

通过这个方法我们定义了一个属性值，但是除非要对属性本身有操作，不然我们不会使用这种方法来实现对象的定义的。

### 可写

属性是否可以表更是由`writable`来控制的：

```js
var myObject = {};
Object.defineProperty( myObject, "a", {
				value: 2,
				writable: false, //not writable
				configurable: true,
				enumerable: true
} );

myObject.a = 3;
myObject.a; //2
``` 

如上述代码，我们修改`writable : falase`的对象属性，我们不能得到想要的结果，而且在严格模式下，更会报错：

```js
"use strict";
var myObject = {};
Object.defineProperty( myObjet, "a", {
				value: 2,
				writable: false,
				configurable: true,
				enumerable: true
};

myObject.a = 3; //TypeError
```

`TyoeError`报错信息提醒我们：对不可写的属性我们是不能进行复制操作的。

###可配置

通过更改`configurable`描述符，我们可以规定它们是否能被修改，但是谨记一点，一旦`configurable: false`后，我们就不能再对它们修改了，这是一个不可逆的过程。

###枚举

枚举描述符很好的传达了它可配置的信息：当它为真时，属性可以枚举，比如`for...in`，一旦设置为`false`，我们就不能通过枚举访问到它了。

###不可改变

有时，我们需设计一个不能该表的属性或者对象，ES5中有多种方法可以实现这种需求，但是我们需要了解的是，这些操作都是*表层*地操作，只能影响到对象和直接的属性值，如果对象中包含对另一对象的索引（数组，函数和对象等），那么索引对象的值是可以改变的，这是很重要的一点。

```js
myImmutableObjec.foo; //[1,2,3]
myImmutableObjec.foo.push( 4) ;
muImmutableObjec.foo; //[1,2,3,4]
```
###对象常量

通过设置`writable`和`configurable`我们可以定义**常量**，一个不能被修改，不能删除的真正意义的常量：

var myObject = {};
Object.defineProperty( myObject, "FAVORITE_NUMBER", {
				value: 42,
				writable: false,
				configurable: false
});
```

###阻止对象拓展

通过`Object.preventExtensions()`我们可以锁死对象，使得其不能新增属性。

```js
var myObject = {
				a:a
};
Object.preventExtensions( myObject );
myObject.b = 3;

myOject.b; //undefined
```

关于属性的一些描述符，这里不再多说，需要进一步学习的可以参考英文原版。

###遍历

`for...in`循环可以遍历对象的`enumerable`属性，但是假如你想遍历值呢？
对于普通数组而言，对值得遍历可以用`for`循环实现：

```js
var myArray = [1,2,3];
for(var i = 0; i < myArray.length; i++ ) {
				console.log( myArray[i] );
}
//1 2 3
```

这其实不是通过值得遍历，这是通过数组索引的遍历，并通过数字索引访问到数组的值而已，ES5中新增了一些对数组遍历的有用方法：`forEach, every, some`等等，它们都接受一个回调函数，并应用于数组的每个元素，差别仅在于回调函数返回的值不相同。

`forEach`会遍历数组的所有值，并会忽略回调函数返回的任何值，`every`遍历会持续至回调返回`false`，而`some`则是持续遍历至回调返回`true`为止。

`every`和`some`的这些特性，有点像是`for`循环中添加了一个条件判断的`break`，一旦满足某种条件就跳出循环。

如果你通过`for...in`来遍历某个对象，你仅仅也只能间接访问到对象的可遍历属性。但是如果，你想直接遍历数组或对象值，我们可以使用`for..of`方法：

```js
var myArray = [1,2,3];
for( var v of myArray ) {
				console.log( v );
};
//1
//2
//3
```

# 总结

对象可以通过对象字面量声明，同样也可以通过`new`构造声明，但是一般情况下，我们可优先使用对象字面量方法，只有当我们需要对对象更多的操作是使用构造函数定义对象会更有优势.

对象键值对的存储，对象属性值可以通过`.propName`或["propName"]来访问，但一个属性被访问时，实际是调用了内部默认的[[GET]]方法（设置水属性当然就是调用[[SET]]方法），这不仅仅是查询对象直接的属性值，它会根据原型链进行深度遍历。










