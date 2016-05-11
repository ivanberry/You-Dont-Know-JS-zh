# Chapter 1: this OR That?

`Javasrcipt`中最令人困惑的就是`this`关键字，它是一个自动定义在每个函数作用域中的特殊的标志符，但是它，就算对大牛来说，也是一个编码的”恶魔“。

## this能干吗？要它做甚？

假如它这么不惹人疼爱，为什么我们还需要它呢？直接不用不就好了吗？这说明它还是有用处的：

```js
function indentify() {
				return this.name.toUpperCase();
}

function speak() {
				var greenting = "hello, I'm " + indentify.call( this );
				console.log( greenting);
}

var me = {
				name: "Kyle"
};
var you = {
				name: "Reader"
};

indentify.call( me ); //KYLE
indentify.call( you ); //READER
speak.call( me ); //hello, I'm KYLE
speak.call( you ); // hello, I'm READER
```
这段代码使得`indentify`和`speak`在不同上下文(`me`& `you`）复用，而不用重新为每个对象定义新的方法。

其实，除了依赖`this`之外，我们可以把上下文作为形式参数传入方法中，同样可以复用：


```js
function indentify( context ) {
				return context.name.toUpperCase();
}

function speak( context ) {
				var greeting = "hello, I'm " + indentify( context );
				console.log( greeting );
}
indentify( you ) ; //READER
speak( me ); //hello, I'm KYLE

```

然而，`this`好像能更加优雅的实现复用，代码也更清晰。既然这样子我们就把它搞懂吧！

## 困惑

正式学习`this`的使用前，我们先把环境理清楚，开发者们对`this`过于字面量的理解，常见的两种误解：

### 它本身

>`this`指函数本身

为什么你需要一个指针指向函数本身呢？最大的可能是在函数本身调用自己了。

我们根据代码学习吧：

```js
function foo(num) {
				console.log( "foo: " + num );
				//keep track of how many times 'foo' is called
				this.count++;
}

foo.count = 0;
for (var i = 0; i < 10; i++) {
				if ( i > 5 ) {
								foo( i );
				}
}
//foo: 6
//foo: 7
//foo: 8
//foo: 9
//how many times was `foo` called
console.log( foo.count ); // 0 --! WTF?
```

这是什么鬼？命名调用了4次`foo`，怎么我们的`foo.count`还是0呢？这里，就是把`this`太字面量理解了，以为`this.count++`就是`foo.count++`，所以才会有这样子的预期结果。而实际情况是：当运行到`foo.count++`时，我们只是对函数`foo`新增一个属性`count`,但是函数内的`this.count++`并不是这个函数中的`count`属性，尽管它们的名字是一样的，它们对应的根对象是不同，所有我们才会有`WTF?`的感叹。

这个时候我们可能就会问：那我`count++`自增的是谁啊？

根据浏览器调试的结果，我们能发现调用`foo(i)`时，我们在全局建立了一个新的变量`count`，而且最后它的值会是`NaN`。

我们先不深究为什么，先看看会有一些什么样的解决办法：

```js
function foo(num) {
				console.log( "foo: " + num );
				//keep track of how many times `foo` is called
				data.count++;
}
var data = {
				count = 0
}
for ( var i = 0; i < 10; i++) {
				if (i > 5) {
								foo(i);
				}
}
// This will be the except results

```

这里解决问题的本质是什么呢？是绕过`this`呢，直接用词法作用域的概念来处理这个问题，可是这样子我们还是不懂`this`关键字。

**说明**：词法作用域是非常有用的机制，但是这不能是我们不使用`this`的原因，也不能因为对`this`理解不够，就绕过直接使用词法作用域，本质问题还是需要解决的。

通过`this`来实现函数指向自己不是很有效，所以我们通过一个词法标志来指向我们的函数对象:

```js
function foo() {
				foo.count = 4; //`foo` refers to itself
}
setTimeout( function() {
				//anonymous function, cannot refer to itself
},10);

```

第一个函数中，我们通过函数名`foo`来调用自己的属性值，但是一旦遇到匿名函数，我们也嗝屁罗，因为它没有名字！！

**说明**：一种弃用的办法： arguments.callee能在函数中调用自己

上述方法确实能实现我们的需求，但是和上上段一样，只是让我们跳过对`this`的掌握，没有从本质上很好解决问题，不是很可取。还有一些方法，但是都没有从本质打通对`this`的掌握，所以先暂且不提。

### 它,作用域

> `this`是指函数的作用域

这种说话有时候是正确的，但是却不定一定一直是,首先，明确一定，任何时候，`this`都不会指向函数词法作用域，作用域是值包含各有效标志的属性的对象，但是作用域这个”对象”对JavaScript而言，确实不可访问的，它是引擎运行时的一部分。**implementation**怎么翻译！！

下面展示一个尝试使用`this`来访问函数词法作用域的失败例子：

```js
function foo() {
				var a = 2;
				this.bar();
}
function bar() {
				console.log( this.a );
}
foo(); //undefined
```

以上的代码不是为了展示而杜撰的，是实际存在的，它的出现表明我们很多时候没有认真的把`this`等搞明白，闹清楚。

1. 以上代码通过`this.bar`访问到函数`bar`纯粹是碰巧，就上述代码而言，我们完全可以忽略掉`foo`中的`this`，直接使用调用`bar`就好了，这利用的是词法作用域最基本的观点。而书写以上代码的开发者本意是想搭建`foo`与`bar`互通的桥梁，但是这是不可能的，你不可能通过`this`来实现在词法作用域中查询什么东西。

所以，我们最好记住，每当你想通过`this`来实现某种连接时，你应该醒悟：擦，又在异想天开！

## `this`到底是什么鬼？

了解部分对`this`的误解后，我们总得知道它到底是什么鬼吧？

早先说过：`this`值不决定于声明时，而是调用时，它决定于函数被调用的形式，当函数定义时，`this`值是没有任何意思的，但是根据函数被调用的不同，它也可能是各种不同的值。

当一个函数被调用时，一个执行环境对象会生成，它包含着：函数调用位置，函数调用形式，传入哪些参数等等。其中就包含了存在于函数执行时的`this`指针。

## 复习一

`this`的掌握总是那么的不彻底的，我们层层剥开它的神秘面纱，看看到底是不是一个美女？

要知道`this`到底是什么，我们需要先知道它一定不是什么，这样子可以减少很大部分对它的误解，它既不是函数本身的指引，也不是函数词法作用域的指引。`this`的值确定于函数调用时，而不是函数声明时。


# 关于`this`的一切

## 调用位置

要理解`this`的绑定，首先我们要理解一点：函数调用的位置（不是函数定义的位置），通过对调用位置的理解，我们就能明白：`this`到底指什么？,要知道函数在哪儿被调用，却不是听起来那么简单，一些编程的模式可能会对实际函数调用的定位有副作用。理解两个名词：

>call-stack: 调用栈
>call-site: 调用位置

```js
functin baz() {
				//call-stack: baz
				//so our call-site is in the global
				console.log( "baz" );
				bar(); // <-- call-site for bar
}

function bar() {
				//call-stack is : baz --> bar
				//so our call-site is in the baz
				console.log( "bar" );
				foo(); //<-- call-site for foo
}

function foo() {
				//call-stack is: baz --> bar --> foo
				//so our call-site is in the bar
				console.log( "foo" );
}
baz(); //<-- call-site for baz
```

我们需要认真分析函数调用栈，因为这是唯一决定我们函数调用位置。

**注意**：这里我们表象话调用栈了，但是这种表现形式不是很正确，其实我们的浏览器中对JS调试非常友好，利用好断点，能很好的调试call-stack等。

在知道`call-stack`后，我们就需要了解关于`this`的4条规则，一下听我们细细道来。

### 规则1：默认绑定

首先需要学些的就是`this`最普遍的绑定方式，这可以理解为是最普遍调用函数的方法**独立函数调用的结果**,当其他规则不适用时，那么就是现在学习的这条了。

```js
function foo() {
				console.log( this.a );
				}

var a = 2;
foo(); //standalone invocation
			//2
```
又要明确一点：这里全局定义的变量`a`和全局对象的属性`a`是同名的，它们是指向一个东西的，谁都不是谁的复制，就想硬币的两面一样，都是指硬币。

当我们的`foo`被调用时，`this.a`指向全局变量`a`，为什么呢？这是因为这个例子中，默认的绑定给函数调用，因此`this`指向了全局对象。但是，我们如何判断默认规则的适用呢？这就需要对`call-stack`，这就需要我们的调用栈帮助理解，在调试工具中，我们在调用栈中看不到其他函数或者环境，这就意味着这个函数的调用时**独立调用**，那么`this`就需要匹配我们的默认规则了。

假如我们处于严格模式，默认的绑定方式就跟全局对象没有什么关系了，因此我们的`this`就是`undefined`了。

```js
function foo() {
				"use strict";
				console.log( this.a );
}
var a = 2;
foo(); //TypeError: this is undefined
			//TypeError: Cannot read property 'a' of undefined
```

一个很重要的细节：尽管`this`的绑定是完全基于**函数调用位置**的，`this`与全局对象的绑定是函数运行时是否处于严格模式相关，而与调用函数时是否是严格模式无关。

```js
function foo() {
				console.log( this.a );
			}
			
var a = 2;

(function() {
				"use strict";
				foo(); //2
})();
```

```js
function foo() {
				"use strict";
				console.log( this.a );
				}

var a = 2;
(function() {
				foo(); //TypeError: Cannot read property a of undefiend.
})();
```

上面两个代码块很好的证明了我们的观点：`this`于全局对象的绑定是跟函数运行时是否处于严格模式有关，而与调用它时是否处于严格模式无关。我们的代码最好是要么处于严格模式，要么非严格模式，混合会带来一些不可思议的问题。

### 隐晦的绑定

这是另外一个规则：调用函数的位置有存在一个上下文，同样属于某个对象时---- /TODO

```js
function foo() {
				console.log( this.a );
}

var obj = {
				a: 2,
				foo: foo
};

obj.foo(); //2
```

首先，我们注意`foo`的声明方式，紧接着它作为一个属性添加给`obj`对象，不管函数`foo`是定义在对象中，还是作为指引属性添加，这两种情况中，`foo`都不真正属于`obj`对象。

但是，函数调用位置利用了`obj`上下文来调用函数，因此。我们可以说当函数被调用时,`obj`对象“拥有”或者“包含”我们的函数指引。

不管我们怎么称呼这种模式的调用，它作为一个对象的指引被调用。当一个函数指引存在于一个上下文对象时，“隐晦的绑定”模式就产生了，接着就是情理之中的`this`绑定给了当前上下文，因此对于`this.a`等同于`obj.a`就没有什么疑问了。

同时我们要注意到，只有最接近的外层的对象属性指引才会影响到call-site,其实我们自己可以理解下，这里可以用词法作用域来解释：

```js
function foo() {
				console.log( this.a );
}

var obj2 = {
				a: 42,
				foo: foo
};

var obj1 = {
				a: 2,
				obj2: obj2
};

obj1.obj2.foo(); //42
```

### 绑定丢失

最最令人困惑的一种情形是，当隐晦含蓄绑定的函数丢失绑定时，这经常意味着它会“优雅”回退到默认绑定方式，根据是否为严格模式为全局对象或者是`undefined`。

```js
function foo() {
				console.log( this.a );
}

var obj = {
				a: 2,
				foo: foo
};

var bar = obj.foo; // function refrence alias!
var a = "opps oops, global!"; // `a' also property on global object

bar(); //"opps oops, global!"
```

尽管说`bar`以`obj.foo`的一指引出现，其实它本质就是`foo`的指引而已，而且，调用的位置也很关键，调用方式是`bar()`，所以这里就应该是默认的绑定方式。

```js
function foo() {
				console.log( this.a );
}
function doFoo( fn ) {
				// `fn` is just another reference to `foo`
				fn(); // <-- call-site!
}

var obj = {
				a: 2,
				foo: foo
};

var a = "oops, global"; // 'a' also property on global object
doFoo( obj.foo ); // "opps, global"
```

形式参数的传入其实就是含蓄的赋值过程，此处我们传入一个函数，所以这里就是一个含蓄的指引赋值过程而已，上面两段代码就没有什么差别了，实质是一样的。

回调函数的`this`绑定变化是很常见的，但令人惊讶的是当函数作为回调传入时，有意更改`this`的绑定，例如，事件处理函数就偏好这么处理我们的回调函数，给`this`绑定给了触发的DOM节点元素，这种方式有时候很好用，但是有时我们不能很好的把控。但是我们总是能找到一些办法的！

### 明确的绑定

隐晦的绑定方式中，我们需要一个新的对象，并给它添加指向自己的属性后，然后，通过对象的方法来间接的绑定`thi`给这个对象，但是如果你想不通过对象属性指引来实现绑定给特定对象，那我们改怎么做呢？

所有的函数都有一些内置属性或方法来实现这样的需求，特别地，函数拥有`call`和`apply`方法，我们如何利用这些呢？这两个方法接受一个对象做为它们的第一个参数，作为`this`，紧接着调用函数，因为，你明确`this`成为啥子，所以我们可成为这种方式是“明确的绑定”。

```js
function foo() {
				console.log( this.a );
}

var obj = {
				var a =2;
}

foo.call( obj ); //2

```
这里通过`foo.call(obj)`来明确绑定`this`给`obj`,所以我能返回`obj.a`。

如果我们通过`call`传入了简单数值（字符，数字或者布尔值），那么传入的值会以对象的形式（new String(...) etc),对于`call`与`apply`的差别不大，主要是传入的参数不同而不同，暂时不表。

### 硬绑定

“明确绑定”的一个变种可能会有些奇怪：

```js
function foo() {
				console.log( this.a );
}

var obj = {
				a: 2
};

var bar = function() {
				foo.call( obj );
};

bar(); //2

setTimeout( bar, 100 ); //2
//bar 使得`foo`的`this`硬绑定到了`obj`上
//它们打死都不要分离，好感动啊，有木有！

bar.call( window ); //2

```
这里我们定义了函数`bar`，紧接着，通过`foo.call( obj )`强制绑定`this`到`obj`，之后，不管你如何调用`bar`，这个绑定既明显又硬性，我们简称为硬绑定。

```js
function foo( something ) {
				console.log( this.a , something );
				return this.a + something;
}

var obj = {
				a: 2;
}

var bar = function() {
				return foo.apply( obj, arguments );
				};

var b = bar( 3 ); // 2 3
console.log( b ); //5
```
上面的代码块是常用一个硬绑定的模式，另外一种方式就是创建一个能复用的`helper`函数：

```js

function foo( something ) {
				console.log( this.a , something );
				return this.a + something;
}

//simple helper function
function bind( fn, obj ) {
				return function() {
								return fn.apply( obj, arguments );
				};
}

var obj = {
				a:2
}

var bar = bind( foo, obj );
var b = bar( 3 ); //2 3
console.log ( b ); //5
```
因为硬绑定是如此的常用，ES6中内置了`Function.prototype.bind`：

```js
function foo( something ) {
				console.log( this.a , something );
				return this.a + something;
}
var obj = {
				a:2
};

var bar = foo.bind( obj );
var b = bar( 3 ); //2 3
console.log( b ); //5
```

内置的`bind`函数返回一个硬绑定`this`上下文到指定变量，这就不需要我们在使用`apply`或者`call`，可以方便的使用`ES6`语法来实现，美哉。

### API 绑定上下文

许多的库或者运行环境(浏览器或者node)下都有提供一个可选参数，常称之为"context"，它的存在使得我们不一定要使用`bind`函数来实现特定的`this`值绑定。

```js
function foo(el) {
				console.log( el, this.id);
}

var obj = {
				id: "awesome"
}
//use obj as this for foo(..) calls

[1,2,4].forEach( foo, obj );
```
这些方法大都是利用`call`或`apply`来显式的绑定。

### 关键字`new`

最后一种关于`this`的绑定方法能引起我们对JavaScript中的函数和对象一番思考，传统面向对象的编程语言中，构造函数类中一个特别的方法，当利用`new`关键字来一实例时，构造函数就会被调用：

```js
something = new MyClass(...);

```
Javascript中同样也有`new`操作符，大部分开发者同样也认为它同其他面向对象的语言一样，做了类似的事情，实际上，却没有什么联系。

首先，我们明确一下JS中构造函数是什么鬼？Js中，构造函数仅仅只是碰巧利用`new`来调用的函数而已，它没有绑定给特定的类，也没有实例化某个类，它更不是一个特别类型的函数，它们也只是普通的函数而已，不过只是被`new`这个东西给*绑架*了。

举个栗子,`Number(...)`：

> from ES5.1
> 15.7.2 The Number Construtro
> When Number is called as part of a new expression it is a constructor: it initialises the newly creater object.

上面这段鬼文大概的意思就是：JS中任何函数，甚至包括内置函数都可以通过`new`的方式来调用，这使得函数是一种`构造调用`，这看上去没有什么，但是本质就是说：没有所谓的构造函数，只是函数的构造调用方法。（真尼玛绕嘴）

当某个函数通过`new`来实现调用时，就会自动有以下的步骤：

1. 一个新的对象就这么生产
2. 新的对象通过[[Prototype]]联系着
3. 新构造对象的`this`绑定到调用函数
4. 除非函数返回替代的对象，不然`new`调用函数会返回新构造的对象

**这一段比较生涩，待我理解后再来**

来看个例子：

```js
function foo(a) {
				this.a = a;
}

var bar = new foo(2);
console.log( bar.a); //2
```
通过`new`调用`foo`，我们新建了一个对象，并且将调用`foo`的`this`设置为了这个新的对象，因此`new`是函数`this`值中最后一个可能绑定方式，可称为“构造绑定”。

## 顺序之我知

至此，我们已经了解了4中不同关于`this`的规则，那么需要我们做的就是：找到`call-site`（函数调用的位置），并配对应的规则就好了，那么问题来了：要是`call-site`能匹配上多条规则，那该怎么办呢？这就涉及到上述几条规则的优先级问题了。首先，能明确的是，默认的绑定方式肯定是优先级最低的，所以我们可以先把它放置一边，先看看其他几条规则：

隐性绑定和显性绑定哪个优先级高呢？

```js
function foo() {
				    console.log( this.a );
}

var obj1 = {
				    a:2,
				    foo: foo
};

var obj2 = {
				    a:3,
				    foo: foo
};
// 隐性绑定
obj1.foo(); //2
obj2.foo(); //3
// 显性绑定
obj1.foo.call(obj2); //3
obj2.foo.call(obj1); //2
```

从上得知，显性绑定优先于隐性绑定的，所以当我们寻找`call-site`并发现有多条规则对应它时，显性绑定是优先于隐性绑定的。构造绑定怎么样呢？

```js
function foo(something) {
				    this.a = something;
}

var obj1 = {
				    foo: foo
};

var obj2 = {};

obj1.foo(2);
console.log( obj1.a);//2

obj1.foo.call( obj2,3 );
console.log(obj2.a);//3

var bar = new obj1.foo(4);
console.log(obj1.a);// 2

//这里输出4而不是2就说明构造绑定要优先于隐性绑定
console.log(bar.a);//4? 2? 4
```
那接着就是构造绑定和显性绑定谁强谁弱呢？

**注意**：`new`和`call/apply`不能同时使用，也就是说不会有`new foo.call( obj1)`这样的调用，但是我们还是可以通过`硬绑定`来测试二者的优先级的。

先回忆下硬绑定，它是通过`Function.prototype.bind()`来强制绑定`this`到特定对象的，这种方法是一种显性绑定方式，很厉害，我们猜想它的优先级别应该要高于`new`的。

```js
function foo(something) {
				this.a = something;
				}

var obj1 = {};
var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a); //2

var baz = new bar(3);
console.log(obj1.a); //2
console.log(baz.a); //3
```
这里，没有如我们预想一般改变`obj1.a`的值，但是通过硬绑定的`bar`的函数却被`new`复写了，其实这里不能算是对`bar`的复写，而是通过`new`我们返回了一个新的对象`baz`，而`baz.a`值为3而已。

那么构造绑定能复写硬绑定有什么用？

最大的用处在于：我们可以建立一个函数（通过`new`来构造一个对象）,并从本质上来忽略`this`的硬绑定。另外`bind(...)`会将在第一个`this`后的参数最为默认的标准参数传递给函数。**这里暂时搁浅深入学习**


## **this**之于我

我们学习了关于`this`的4条规则，现在我们来总结下载函数调用位置决定`this`的各种不同，根据它们的优先级来讨论：

1. 函数是通过`new`调用的？(构造绑定)若为是那么`this`就是我们新构造的对象。

`var bar = new foo()`

2. 函数是通过`call/apply`调用的？(显性绑定),或者是通过`bind`硬绑定？若是，`this`就是显性绑定给特定的对象。

`var bar = foo.call( obj2 );`

3. 函数通过上下文被调用？(隐性绑定),或者函数本身就是属于或者包含于对象？若是，`this`就是这个上下文。

`var bar = obj1.foo()`

4. 其他所有，就是默认的`this`值（默认绑定），若是`use strict`模式，就是`undefined`，不是就是`window/global`。

`var bar = foo()`

以上就是我们几乎全乎的对`this`的理解了，知道这个就基本明了`this`。

## 一些例外

总有一些例外想颠覆规则，总有一天要好好收拾你们：

### 忽略

当使用显性绑定，并传入`null undefined `（call, apply, bind)时，这就意味着此处的显性绑定，`this`是被忽略了，取而代之的是默认绑定：

```js
function foo() {
				console.log( this.a );
}

var a = 2;
foo.call( null ); //2
```
这是这么厉害，话说这里这么绑定是为什么呀？我现在还不知道这种用处是干什么用的，当时既然存在可能就有某种意义吧，以后遇到可以学习学些！
没想到这么快就遇到了，通过`apply`传入一个数组参数的方式调用函数，同样地，通过`bind()`引入某些形参：

```js
function foo(a,b) {
				console.log( "a:" + a + ", b:" + b);
}
//将数组作为形参传递
foo.apply( null, [2,3]); // a:2, b:3
//bind(...)
var bar = foo.bind(null,2); //若this为null，那么后面的参数作为默认值引入
bar(3); // a:2, b:3
```

好像很有用的样子，需要实例的联系才能更好的理解，这里就可以设置一个目标了，比如通过学习某个工具库来实现对这些知识更好的掌握，哪个工具库呢，之后决定吧！可能是`underscore loadsh`。

### 






























				

















