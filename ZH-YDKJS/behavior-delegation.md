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

![模拟类的模型](../this\ \&\ object\ prototypes/fig4.png)

![模拟类的简化模型](../this\ \&\ object\ prototypes/fig5.png)

![OLOO模型](../this\ \&\ object\ prototypes/fig6.png)

这三个图一出，我想我们都会选择最简单的模型吧，最后一个种方法就是我们上面提及的**OLOO**风格的编码，它少了很多可能出现问题的环节，但是也是最大程度的利用了我们的原型链来实现继承或者说是代理，也就是完成了我们最核心最实质的需求：建立对象间的联系。

### 昆仑论剑

我们说了很多关于类和对象间的理论性差异等，但是实践起来真的有说起来这么好听好吗？*Talk is cheaper, show me the code*.我们用一个widget来试试。

#### 类widget

我们可能曾经习惯使用OO的设计模式，我们可能能立刻给出一个方案：来个一个父类（Widget），它有各种widget的共同属性，接着基于它创建一个子类（比如按钮），从代码层面来说
就是这个样子的：

```js

//parent class
function Widget(width, height) {
				this.width = width || 50;
				this.height = height || 50;
				this.$element = null;
}

//add method
Widget.prototype.render = function($where) {
				if(this.$element) {
								this.$element.css( {
												width: this.width + 'px';
												height: this.height + 'px';
								}).appendTo( $where );
				}
};

//Child class
function Button(width, height, label) {
				//super constructor call
				Widget.call(this, width, height);
				this.label = label || 'Default';
				this.$element = $('<button>').text( this.$label );
}

//make Button inherit from Widget
Button.prototype =Object.create( Widget.prototype );
//overwrite base inherited render method
Button.prototype.render = function($where) {
				//super call
				Widget.prototype.render.call( this. $element );
				ths.$element.click ( this.onClick.bind( this ) );
};
Button.prototype.onClick = function(event) {
				console.log( "Button '" + this.label + "' clicked! " );
};

$(document).ready( function() {
				var $body = $(document.body);
				var btn1 = new Button( 125, 30, "Hello");
				var btn2 = new Button( 150, 40, "world");
				btn1.render( $body );
				btn2.render( $body );
});
```
#### *ES6* `class`

先不提吧！

#### OLOO风格

我们尝试用OLOO风格来实现下：

```js
var Widget = {
				init: function( width, height) {
								this.width = width || 50;
								this.height = width || 50;
								this.$element = null;
				},
				insert: function( $where ) {
								if(this.$element) {
												this.$element.css({
																width: this.width + 'px';
																height: this.height + 'px';
												}).appendTo( $where );
								}
				}
};

var Button = Object.create( Widget );
Button.setup = function(width, height, label) {
				this.init( width, height);
				this.label = label || 'default';
				this.$element = $( '<button>').text( this.label );
};

Button.build = function($where) {
				//delegated call
				this.insert ( $where );
				this.$element.click ( this.onClick.bind (this));
};

Button.onClick = function(event) {
				console.log( "Button '" + this.label + " 'clicked!");
};

$( document ).ready( function() {
				var $body = $( document.body );
				var btn1 = Object.create( Button );
				btn1.setup( 125,30,"LOL");
				btn1.build( $body );
};
```

我们不把Widget当做是一个父类，同样也不把Button当做它的一个子类，它们都是独立的一个对象，Widget是一个有各种好用属性和方法的集合，任何特定的widget都可能代理到它，
而我们的Button刚好建立了这么个代理关系。

OLOO风格的代码看起来更简单，明了，联想到我们的模型，它也确实简单很多，也咩有`prototype`等干扰，总而言之，在完成功能的前提下，我们把架构简化了，这也是降低错误
一个方向。

#### 简洁的设计

OLOO给我们更简单，更灵活地设计风格，通过代理的方式可以让我们的拥有强壮的代码架构，我们在来看看一些例子：

两个控制器，一个是登录，一个是服务器授权。

经典的类设计风格，我们需要将这个任务分割成一个基础类（Controller），再分支处两个子类*LoginController*和*AuthController*，它们都继承与*Controller*，并拥有自己的
特定的方法：

```js
//parent class
function Controller() {
				this.errors = [];
};

Controller.prototype.showDiag = function( title, message ) {
				//display title message to user in a dialog
};
Controller.prototype.success = function( message ) {
				this.showDialog( "Sucess", message );
};
Controller.prototype.failure = function( error ) {
				this.errors.push( error );
				this.showDialog( "Error", error );
};
```

```js
//child class
function LoginController() {
				Controller.call( this );
}

//link child class to parent
LoginController.prototype = Object.create( Controller.prototype );
LogonController.prototype.getUser = function() {
				return document.getElementById( 'login_username' ).value;
};
LoginController.prototype.getPassword = function() {
				return document.getElementById( 'login_password' ).value;
}

LoginController.prototype.validateEntry = function(user,password) {
				user = user || this.getUser;
				password = password || this.getPassword;
				if( !(user password) ) {
								return this.failuer( 'Please enter a username and password' );
				}else if( password.length <= 5 ) {
								return this.faliure ( 'Password must be 6 characters and above!' );
				}
				//be here? validated
				return true;
};

//Override to extend base 'failure()'
LoginController.prototype.failure = function( error ) {
				//super call
				Controller.prototype.failuer.call( this, "Login invalid: " + error;
};
```

```js

//child class
function AuthController(login) {
				Controller.call ( this );
				//in addtion to inheritance, we also need composition
				this.login = login;
}
//link child class to parent
AuthController.prototype = Object.create( Controller.prototype );
AuthController.prototype.server = function( url, data ) {
				return $.ajax( {
								url: url,
								data: data 
				});
};

AuthController.prototype.checkAuth = function() {
				var user = this.login.getUser();
				var password = this.login.getPassword();

				if( this.login.validateEntry ( user, password ) ) {
								this.server( '/check-auth', {
												user: user,
												password: password
								})
								.then( this.success.bind ( this ) )
								.fail( this.failure.bind( this ) );
				}
};

//override to extend base 'success()'
AuthController.prototype.success = function() {
				//super call
				Controller.prototype.success.call( this, "Authenticated");
};

//override to extend base 'failure()'
AuthController.prototype.failure = function(error) {
				//super call
				Controller.prototype.failure.call( this, "Auth Failed: " + error );
};
```

```js
var auth = new AuthController(
//in addtion to inheritance, we also need composition
new LoginController()
);
auth.checkAuth();
```

代码比较多，需要仔细体会！

#### 去类化

上面的代码对我这个一开始就接触JavaScript而言，也不是那么容易懂，我们先放置，先看看OLOO风格是如何充分利用好原型继承的前提下，简化实现我们的功能：

```js
var LoginController = {
				errors: [],
				getUser: function() {
								return document.getElementById( 'login_username' ).value;
				},
				getPassword: function() {
								return document.getElmenetById( 'login_password' ).value;
				},
				validateEntry: function( user, password ) {
								user = user || this.getUser();
								password = password || this.getPassword();
								if( ! (user && password) ) {
												return this.failure ( 'Please enter a username and password!');
								}else if( password.length < 5 ) {
												return this.failure( 'Password must be 5+ characters!' );
								}
								//get here? validated!!
								return true;
				},
				showDialog: function(title, message) {
								//display some errors
				},
				failure: function( errors ) {
								this.errors.push( errors );
								this.showDialog( 'Error: ', 'Login invalid: ' + errors );
				}
};
```

```js
//link authController to delegate to LoginController
var AuthController = Object.create( LoginController );
AuthController.errors = [];
AuthContoller.checkAuth = function() {
				var user = this.getUser();
				var passwrod = this.getPassword();
				if( this.validateEntry( user, password )) {
								this.server( '/check-auth', {
												user: user,
												password: password
												})
												.then( this.accepted.bind( this ))
												.fail( this.rejected.bind( this ))
												}
};
AuthController.server = function( url, data ) {
				return $.ajax( {
								url: url,
								data: data
				});
};
AuthController.accepted = function() {
				this.showDialog ( 'Sucess', 'Authenticated' );
};
AuthController.rejected = function() {
				this.failure ( 'Auth Failed: ' + errors );
};
```

因为`AuthController`仅仅就是一个对象，`LoginController`同样也只是一个对象，我们不需要对它们进行实例化来使用，需要调用它只要：

```js
AuthController.checkAuth();
```

当然，如果你需要利用代理链来创建更多的对象，也很简单：

```js
var controller1 = Object.create( AuthController );
var controller2 = Object.create( AuthController );
```
通过代理，`AuthController`和`LoginController`都仅仅只是对象，平级的对象，它们间可以随意更改代理关系来实现我们的功能需求，另外OLOO风格的编码方式使得我们仅有两个入口（`LoginController`和`AuthController`，没有更多的。

我们不需要一个公共的`Controller`来共享一些属性，因为原型链的特性本身就提了我们需要的功能，同样，我们也不需要实例化一个类，因为类的概念在JavaScript是不存在的，只有对象而已。另外，我们也不需要组合它们，代理使得两个对象进行地差异化合作。

最后，我们应避免应用相同的属性或方法名来”覆盖“，比如`accepte, reject`和`success, failure`的差异，使用更明确的变量名来描述我们更加独特的特性，代码结构简单，逻辑清晰。

### 总结

类与继承是在软件架构中，你可以选择或者忽略的设计模式。大多数开发者认为”类“是仅有的或者说合适的方法来组织代码，但是我们这里展现了一种同样有效，而且更加简洁，清晰的实现方法：代理(behavior delegation)。

*Behavior delegation*建议我们平级化对象，通过代理建立相互的联系，而不是建立父类与子类的关系。JavaScript的原型特性是很自然语法实现，我们可以充分利用它来实现对象的联系。

**OLOO**利用JavaScript的原生的原型继承链来实现对象的关联，而不需要去模拟”类“这种实际不存于JavaScript中的不友好（至少代码层和模型）来实现对象间的联系，功能上没有差异，简洁，逻辑更加清晰地利用[[Prototype]]，我们何乐而不为呢？



				

				
								

















