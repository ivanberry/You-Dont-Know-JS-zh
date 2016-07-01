## 闭包作用域

我们已经明了作用域的作用机制，接下来就是重中之重：闭包。它看上去可能很是神奇，现在我们就一起来揭开它神秘的面纱:boom:

### 申明

对很多有经验的开发者而言，他们可能并没有真正的理解闭包，理解闭包看起来像是通往顶级的打怪升级，必须而且不那么简单，回想起自己(....作者巴拉巴拉一大堆），

*闭包无所不在，就想空气一般*

闭包其实是依赖词法作用域自然而然出现的，甚至都不需要刻意“创建”一个闭包并利用它，而是在代码编写过程中，它就这么出现了，我们只是缺少一双慧眼来识别它而已，所以这篇文章最重要的点是：

*闭包无所不在，明确识别它们*

### Nitty Gritty

好了，扯了这么多蛋，可以开始正经事情了，先来个普遍的闭包定义：

> 闭包是指函数在其词法作用域外依然能访问它的词法作用域

先上代码为敬：

```js
function foo() {
				var a = 2;
				function bar() {
								console.log( a ); //2
				}
	bar();
}

foo();
```

这跟我们之前提到的作用域嵌套很相似，`bar`能访问到`a`，其实是一个词法作用域的查询规则而已的，那这是一个闭包吗？

额....这算是吧？其实更加精确理解上述代码是：基本的词法作用域查询机制而已，当然这也是闭包的重要规则啦。

这里我们的`bar`函数对`foo`作用域有*封闭*的作用，其实就是说：如果两个作用里都有变量`a`，那么`foo`中的`a`不会被访问到，而是直接访问了`bar`中的变量`a`，其实这是很自然的事情，因为`bar`内嵌于`foo`，根据词法作用域的基本规则，这是必然的嘛！

我们还是把闭包丢到明火中吧！

```js
function foo() {
				var a = 2;
				function bar() {
								console.log( a );
				}
				return bar;
}

var baz = foo();
baz();
```
函数`bar`对`foo`词法作用域有访问权限，这是很自然的事情，但是当我们把它作为一个参数赋值，在这里我们通过`return`来通过`bar`索引返回函数本身。在执行完`foo()`后，我们就把内部函数`bar`赋值给了变量`baz`,紧接着调用它，这自然就是对`bar`的调用嘛，只是用了另外一个不同的标识符而已，函数本身还是那个函数本体。

*函数执行了，但是执行的位置不是它词法作用域中，而是在它词法作用域之外*

根据垃圾回收机制，`foo`函数执行完毕后理应回收一切关于函数`foo`占用的内存，所以，我们应该也是不能访问到`foo`函数作用域中的变量，因为这一次都随风而去了，正是闭包的存在，神奇的事情发生了：

`bar`作用域其实还在使用中，谁使用这个作用域，其实即使它自己。和它定义位置的对`foo`作用域的封闭一样，`bar`依然保持着对它词法作用域的封闭，并且对`bar`而言，在任何时候都能被调用。

`bar`依然保持有对`foo`词法作用域访问的索引，这个索引称为闭包。

所以，当我们执行`baz`时（本质上就是调用了内部函数*bar*)，它理所当然地能访问变量`a`。函数可以在词法作用域之外调用，闭包使得这个函数依然能访问它定义时的作用域。，当然，将函数作为值传递方式不止一种，我们还可以这样：

```js
function foo() {
				var a = 2;
				function baz() {
								console.log( a ); //2
				}
				bar( baz );
}
funtion bar( fn ) {
				fn();
}
```

```js
var fn;
function foo() {
				var a = 2;
				function baz() {
								console.log( a );
				}
				fn = baz; //assign `baz` to global variable
}
function bar() {
				fn(); // look ma, I saw clourse
}
foo();
bar();
```

不管我们通过什么方式把内部函数传递到外部，它都将维护一个索引，指向它定义的位置，无论我们在哪里调用它，闭包就起作用哒:boom::boom::boom:

### 我们得到了什么

之前的代码有很重的学术气味，我们实际编码过程中可能更多的会是这样：

```js
function wait( message ) {

				setTimeout( funciton timer() {
								console.log( message );
				}, 1000);
}
wait( "hello, clourse" );
```

我们定义另一个`timer`函数，并将它传递给`setTimeout`函数，因此`timer`对`wait`作用域有封闭作用（就是说若两个函数中存在相同的变量，首先获取的是`timer`内部的变量值，这就是所谓的作用域封闭作用），`timer`保持并使用一个索引指向变量`reference`。

在`wait`执行后1s，理论上`wait`的作用域应该已经被GC回收了，然而并没有......

假如你使用jQuery，你可能会有这样的代码：

```js
function setupBot(name, selector) {
				$(selector).click(function actiator() {
								console.log( "Activating: " + name);
				});
}
setupBot( "closure Bot 1", "#bot_1");

无论何时，无论何地（我怎么唱起来了？），计时器，事件处理函数，Ajax请求，跨窗口信息传递，web workers，或者任何异步或同步的任务，当你讲函数作为回调执行时，就做好闭包自然出现的准备吧！

第三中，我们提到*IIFE*函数，有人说这就是个闭包很好的例子：

```js
var a = 2;
(function IIFE() {
				console.log( a );
})();
```
其实不然，这里的代码可能能起到作用，但是这并不能说明它是一个闭包很好的展现例子，这里只是简单的实现了词法作用域的查询而已，最重要的是，函数`IIFE`并没有在其词法作用域之外被调用，所以它并不能称为一个闭包，尽管如此，它仍是跟闭包紧密联系着的。

### 循环与闭包

闭包最常使用的场景是循环的使用：

```js
for( let i = 1; i<=5; i++) {
				setTimeout( function timer() {
								console.log(i);
								},1000);
}
```
你可能与其目标是：1 2 3 4 5，但是你肯定不能如愿，啊哈哈！输出结果是：66666,可谓是六六大顺啊！

这就奇怪了，怎么就不是我们的预期结果呢？

很明显，6是因为循环结束时的i值，也就是说这里的定时回调函数是在循环完全结束后才开始运行的，为什么我们不能得到预期的结果呢？

我们预期得到每次循环的特定`i`值，但是作用域机制不是这么回事，我们在每个循环中定义的函数，其实都共享着同一作用域，这个作用域中只有唯一一个变量`i`。其实代码中循环中假如定时器，并传入回调函数，跟一个一个定义这个回调函数没有差别，根本没有什么循环存在。

回到我们的问题？我们到底少了什么东西！我们需要一个闭包，为每一个循环建立一个封闭作用域。

```js
(function() {
				for(var i = 1; i<=5;i++) {
								setTimeout(function timer() {
												console.log(i);
								},i*1000);
				}
}();
```

结果会怎么样呢？







啊哈！还是跟之前那个一样的吧？为啥？

确实，我们有了一个新的词法作用域，但是这个新的作用域里连跟毛的都没有，仅仅只是挂了壳而已，没什么卵用！我们需要在这个作用域里，给每个循环顶一个变量才会使得这个作用域有意义的。

```js
for(var i=1;i<=5;i++) {
				(function() {
								var j = i;
								setTimeout(function timer() {
												console.log( j );
								},j*1000);
				})();
}
```

Done!

```js
for (var i =1;i<=5; i++) {
				(function(j) {
								setTimeout(function timer() {
												console.log( j );
								},j*1000);
				}(i);
}
```
### 再回首块级作用域

我们通过IIFE函数新建了一个作用域，回顾我们之前*ES6*中块级作用域的相关知识，我们不难想到`let`，其实可以这么做：

```js
for( let i=1; i<=5;i++) {
				setTimeout( function timer() {
								console.log( i );
				};
}
```

简直啦！

### 模块

通过回调函数，我们接触了闭包的一个使用途径，但是它却不是这么简单的应用而已！

*模块化*

```js
function CoolModule() {
				var something = 'cool';
				var another = [1,2,4];
				function doSomething() {
								console.log( something );
				}
				function doAnother() {
								console.log( anther.join('!' );
				}
//no closure now!

				return {
								doSomething: doSomething,
								doAnother: doAnother
				}
}

var foo = CoolModule();
foo.doSomething(); //cool
foo.doAnother(); //1 ! 2 ! 3
```
Javascript中，我们称它为模块，我们仔细分析下上面的代码。

首先，`CoolModule()`是个函数，它调用实现模块实例化，没有它的执行，内部函数是不会暴露出来的。紧接着`CoolModule`返回一个对象，返回的对象是通过字面量定义的，对象仅保存着指向内部函数的指引，用这个办法使得，数据和变量等保存私有和封闭，可以说，返回的值是公用的API。

返回的对象赋值给变量`foo`，这样我们可以通过`foo.doSomething`来访问这些属性了。

**注意**：其实，我们不一定要返回一个对象来实现方法的公用，返回一个函数或者其他你想返回的形式也是可以的，这里`jQuery`就是很好的典范。

这里的闭包在哪儿呢？返回的方法对模块实例有封闭作用，来个简单的例子：

```js
function CoolModule() {
				a = [1,2,3];
				function doSomething() {
								console.log( a );
				}

				return {
								doSomething: doSomething
				}
}

var foo = CoolModule();
foo.a = 2;
foo.doSomething(); //???

```
最有一行会输出什么呢？这就是上面说的方法对模块实例的作用域的封闭作用。Did you get it?

简单来说，利用闭包实现模块需要：

- 外部函数的包裹，因此才能实现内部函数对实例作用域的封闭
- 返回

另外一个重要的点是可以通过定义个内部变量来维护你的API：

```js
function CoolModule() {
				var a = 2;
				function doSomething() {
								console.log( a );
				}

				var publicAPI = {
								doSomethingStuff: doSomething
				};

				return publicAPI;
}
```

### 现代模块化模式

直接上代码说话：

```js
var MyModule = (function Manager() {
				var modules = {},
				function define(name, deps, impl) {
								for (var i = 0; i<deps.length; i++) {
												deps[i] = modules[deps[i]];
								}
								modules[name] = impl.apply( impl, deps );
				}
				function get(name) {
								return modules[name];
				}
				return {
								define: define,
								get: get
				};
})();
```







