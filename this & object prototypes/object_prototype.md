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







