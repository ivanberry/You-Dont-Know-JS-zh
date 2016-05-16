---
generator: pandoc
---

You-Dont-Know-JS
----------------

Scope
-----

1.  变量的存储和获取
2.  变量存储在哪儿？
3.  如何找到它们呢？

以上就设计到一个语言的设计中非常重要的一点：
**作用域**，作用域就是这么一系列规则，规范存储和获取变量。但是，还是有问题，**作用域**是怎么定义来的呢？

对JS而言，它是一种“动态语言”或者说“解释性语言”，但本质上它其实是“编译性语言”，只是同传统编译性语言不同，它属于“低级别”的编译性语言。不管怎么说，JS引擎做了许多“编译相关”的事情，传统编译性语言中，经常会经过一下三个步骤来执行程序：

1.  **Tokenizing/Lexing**:
    字面量，在这个阶段，会拆分语句为字面量，`var a = 2;`,被拆分为`var`,`a`,`=`,`2`,`;`，其中的空格根据它是否有意义选择性拆分或者忽略掉。
2.  **Parding**： 渲染为元素树(**AST**)。
3.  **Code-Generation**:
    处理AST使得其成为可以运行的代码。（本质上就是把`var a = 2`）通过AST后，成为一组指令，让计算机真正的创建一个变量`a`，并分配内存空间，接着`2`存储其中。

对JS，我们简单而言：`var a = 2`在执行前，需要一个编译，这个编译事件不长，甚至是即使完成的。

### 理解作用域

我们通过模拟一下，实际过程中，代码运行时涉及的几个关键“人物”来学习作用域：

1.  引擎：负责编译和执行的整个过程
2.  编译器：引擎的一个好友，总是帮着引擎做着“脏活累活”（Parsing And
    Code-generation）
3.  作用域：引擎的又一个好友，收集和维护所有定义的标志符（变量），并通过系列严格的规则来体现，当前执行的代码如何访问到它们。`****`

整个学习JS的过程中，我们首先要把自己当作是**引擎**和它的朋友，实现它们之间的问答，模拟完整的过程。

### Back & Forth

当你看到`var a = 2;`时，你肯定是觉得这就是一个语句，但是引擎和它的小伙伴并不是这么想的，引擎同学把它看称两个部分，一部分是编译器管理的编译过程，另一部分是在执行过程，让我们来以上述语句做例来说明这么一个过程：

一个很合理的假设：编译器会产出这样子的代码：为一个变量分配内存空间，打上标记`a`，把`2`赋值给`a`，然而着却不是很精确的描述。

1.  我擦，这里有个`var a`，编译器同学就问作用域：Hey，你手上有个叫做`a`的变量吗？假如有，编译器就直接忽略`var a`，不然，编译器就叫作用域同学在当前作用域下声明一个新的变量，叫做`a`。
2.  编译器会引擎后续对`a = 2`的执行产出可执行代码，引擎首先问：Hey，Scope，你现在(当前作用域)手上是不是有个叫做`a`的变量？有，就取用，没有的话引擎就会找其他的作用域（一下会提到作用域的嵌套）。

总结来说：一个变量的赋值过程，在两个区域联合发生，首先，编译器申明变量（没有就在当前作用域申明），引擎需找变量，有就赋值，完毕。

### 编译有话说？

在为引擎提供可执行代码的过程中，它会查询是否存在一个变量`a`，这个过程称为*查询作用域*，但是，查询方式的不同会影响查询的结果，这里是一个"**LHS**"的查询方式，另外对应的是“**RHS**”，方式。这个L,R是什么鬼？左边？右边？那是谁的左右边啊？

没错，就是指左右边，它们是指赋值操作的左右边，这里就是`=`两边咯。
也就是说，当一个变量在赋值操作左边时，我们执行的是"LHS"查询，而在赋值右边时，我们执行的是“RHS”查询，而准确的说，“RHS”是查询变量存在的值，而“LHS”则是查询边另它本身.

<div class="sourceCode">

``` {.sourceCode .js}
console.log( a );
```

</div>

很明显这里对的`a`的查询是`RHS`查询，我们需要的是变量`a`的值，而不是容器本身。

<div class="sourceCode">

``` {.sourceCode .js}
a = 2;
```

</div>

这里就是明显的对比，`LHS`查询对`a`本身的值是多少毫不关心，只是找到存储变量`a`的内存区域，然后赋值`2`给它，至于，它原来是什么值，什么类型的值，一点都不在意。

我们在来点：

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    console.log(a);
}

foo(2);
```

</div>

加载代码后，调用`foo(...)`,这是一个`RHS`查询，因为我们是要得到`foo`本身的值，并`()`，即执行它，紧接着，`2`作为实际参数传递给`foo()`形式参数`a`,即`a = 2`的过程实际是一个`LHS`,只是得到`a`这个容器，而不关心它当前状态下，存储的值。接下来`a`作为参数传递给`console.log()`时又是一个`RHS`,取得了变量存储，也就是`2`。其实对于`console.log(...)`也是一个查询的过程，首先`RHS`得到`console`对象，接着是对象的方法`log(...)`,又是`RHS`。

有一个观点，我们定义函数时可以能是这样子`var foo = function({...})`，那这在查询`foo`应该是一个`LHS`查询呀？

上面的观点没有问题，但是不同的是，**编译器**
在`Code-Generation`过程中几乎同时处理了声明和赋值定义，所以当**引擎**执行代码时，没有必要对函数`foo`进行赋值，因此，对于上述函数的讨论不是很必要。

### 对话

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    console.log(a);
}

foo(2);
```

</div>

说明：E =&gt; 引擎， S =&gt; 作用域

> E: S，我有一个对于`foo`的`RHS`查询，你听说过它吗？

> S: 有，有，有这么个鬼，编译器大叔刚才不久声明的，它是个函数。

> E执行`foo`...

> E: S，对`a`的`LHS`查询，有听过吗？

> S: 当然有，编译器把它作为一个`foo`形式参数一起定义的。

> S: 真是棒棒哒，赋值`a = 2`中...

> E: 我需要一个`console`的`RHS`查询。

> S: 找到了，它是一个内建的对象。

> E: 查询`log`...它是一个方法

> E:
> Hey，S，我又需要一个对`a`的`RHS`查询，我记得好像有这么个东西的，不过还是问问你。

> S: 是的，还是之前那个。

> E: 真是给力，`log(...)`中把`2`传给`a`。

我们可以模拟下这个：

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    var b =a ;
    return a + b;
}

var c = foo(2);
```

</div>

### 作用域嵌套

我们说过作用域就是通过标志符来查询值的一系列规则，实际中，作用域不可能只有一个的，和函数可以嵌套类似，作用域也可以嵌套，而引擎的查询规则是：当前作用没有找到的情况下，去上一级作用域查询，一级一级向上，知道全局作用域为止。

\`\`\`js function foo(a) { console.log( a + b ); } var b = 2 ;

foo(2); \`\`\`

> E: foo作用域同学，你那有个叫`b`的小子吗？我这有个`RHS`查询

> fooS: 木有，木有，没有听说过它

> E: fooS外的作用域(这里是Global)，你那有吗？

> S outer of foo: 有

从上列可以看出，嵌套作用域的大致规则，引擎从当前作用查询变量，找到了那就是它，找不到的话就去上一级查询，直到最外层作用域，比如`window`，到达最外层后就停止查询，它也不得不停止了，根本就没有地方去了的。

\#\#\# 错误

说明了这么久的`LHS`和`RHS`，到底有什么用呢？

两种不同的查询方式，在查询未申明的变量时会返回不同的错误信息。

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    console.log(a + b);
    b = a;
}
 
foo(2);
```

</div>

代码中第一次对`b`的查询是`RHS`，当前作用域找不到它，外层甚至最最外层同样也没有，这个时候引擎就要报错了，错误为`ReferenceError`，这是很关键的一点。

然而，对于非严格模式下的`LHS`查询，当在各作用域都找不到`b`，这个时候区别就来了，引擎兄弟不会报错，因为在最外层的作用域兄弟会在全局下新建一个`b`，以满足我们的引擎兄弟。然而，在ES5的严格模式下，不会有自动创建不存在的变量罗，而且同样也会报`ReferenceError`了。

我一个`RHS`查询的值对它执行某种不可能的操作时，比如运行一个不是函数的值，或者值是`null`或者`undefined`时，引擎会抛出一个不同的错误`TypeError`。

总的而言，`ReferenceError`是作用域判断的错误，而`TypeError`则是作用域没错，但是对它执行了一个不可能的操作。

### 总结

作用域是什么？ `RHS`: `LHS`:

javascript的编译交流：

词法作用域
----------

关于作用域的工作原理有两种模型可以说明，一种是最常用的`词法作用域`，另外一种是`动态作用域`，像`Bash`,`Perl`等还有使用。我们这里深入了解下`词法作用域`的相关知识。

经典的编译方法`lexing`或者`tokenzing`就是遍历源代码字符串，通过某种方式赋予它们语义。而词法作用域就是在这个阶段的作用域，也就是说词法作用域就是基于变量和块作用域**认证**的，对作者而言，就是书写的时候，它大多数时间都是在`lexer`阶段决定了的。

### 关于词法作用域的更直白的定义，需要了解

我们看看例子：

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    
    var b = a * 2;
    
    function bar(c) {
        console.log(a, b, c);
    }
    
    bar(b * 3);
}

foo(2);
```

</div>

上述代码中，存在三个作用域，我们可以把它们想象成三个包裹的气泡：

1.  **气泡1**：全局作用域，只有一个标识符`foo`
2.  **气泡2**：`foo`作用域，三个标识符`a`,`b`,`bar`
3.  **气泡3**：`bar`作用域，一个标识符`c`

`bar`作用域完全包裹在`foo`作用域中，这是因为，我们就是在`foo`作用域中定义了`bar`，这些作用域是严格的嵌套，一个作用域不会同时存在于，或者同时部分存在于外部作用域中，就如同，没有一个方法（函数）能部分存在于两个包含函数中一样，它们都是严格包含，嵌套的。

就是这样，作用域才能明确的告知引擎去哪儿查询对应的标识符。上述代码块中，当引擎执行到`console.log(a, b, c)`时，它需要查询三个变量`a`,`b`,`c`，首先它会在当前作用域中查询，即`bar`函数作用域中。假如找不到`a`，那么就想上一级的作用域查询它，即`foo`函数作用域中，同样，其他变量值都是遵照这样的规则查询的，注意一点，引擎第一次找到了某标识符，就会停止查询，无论外围作用域是否存在同样的标志符它都不在关心了。内层的标识符会**遮盖**外层的相同标志，就是这样，变量查询总是从最内层的作用域开始，知道第一匹配后停止查询。

全局变量会自动添加到全局环境（浏览器中的`window`），这时，我们可以不通过它的`词法`名来访问它，通过全局对象的一个属性来访问。这有什么用呢？假如我们需要在内层作用域中访问全局的某相同标识符，因为变量`shadow`的存在，直接访问时不行的，这个时候通过全局对象的属性就能访问到。

不管方法从哪儿被调用，或者怎么调用，它的词法作用域只定义在它声明的地方。

词法作用域的规则只适用于`first-class`标识符，对于`foo.bar.baz`的查询，它会按照词法作用域方法来查询`foo`，然后就是对象属性的访问规则开接管对`bar`,`baz`的查询。

### **欺骗**

介绍了两种对词法作用域`欺骗`的方法，但是都是没弃用的，我们就了解了解下，代码中可以完全避免使用的，它们最大的问题是会使得Javascript本身的编译过程的优化变得毫无意义，完全避免使用它们吧。

函数和块级作用域
----------------

`Javascript`中会创建作用域的方法有哪些呢？只有函数吗？

### 函数作用域

`Javascript`大多数情况下是基于函数的作用域。也就是说，当你申明一个函数的同时，一个新的作用域就创建了。好像没有其他的办法来创建一个新的作用域，这种说法大部分时间是没有问题的，但它不是完全正确的。

我们看看函数作用域：

<div class="sourceCode">

``` {.sourceCode .js}
function foo(a) {
    var b = a;
    
    //some code 
    
    function bar() {
        //...
    }
    
    //more code 
    
    var c = 3;
}
```

</div>

上面的代码块中，`foo`作用域包含有`a`, `b`, `bar`,
`c`等变量。它申明的顺序是否重要我们暂时不讨论。`bar(...)`也有自己的作用域，而全局作用域中只存在一个标识符`foo`。因为`a`,`b`,`c`,`bar`是在`foo`作用域申明的，因此在全局中对它们的直接访问肯定是要回报`ReferenceError`，因为它们根本在全局是不存在的呀。当引擎兄弟文全局作用域小伙伴的时候，它是懵逼的，从来没有见过。

<div class="sourceCode">

``` {.sourceCode .js}
bar(); //fails

console.log(a, b, c); //all 3 fails
```

</div>

理所当然，在`foo`作用域中，它们都是能被访问到的，因为`foo`作用域小伙伴知道它们的存在。

基于函数作用域的变量机制使得它们能在在函数中使用和复用，这能很好的发挥`Javascript`的`dynamic`特性。

### 作用域的隐藏

传统定义函数的方法是:
声明一个函数,在函数体内添加相关的语句,然而,它的另一方面和很是有用的,有一部分代码块,我们在它外围用一个函数声明包含它,这就会达到**隐藏**的作用.这就是说,我们把这一部分代码,绑定到了这个函数的作用域了,而不是代码块之前闭合的作用域中.

我们知道了这个方法,但是它有什么用吗?

理由们:

-   接口低暴露

<div class="sourceCode">

``` {.sourceCode .js}
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );

    console.log( b * 3 );

    }

function doSomethingEles(a) {
    return a -1;
    }

var b;

doSomething( 2 );
```

</div>

这里我们的变量`b`,
`doSomthingElse`看起来应该是`doSomethig`的私有变量和方法,然而现在的做法是将它们暴露在`doSomethig`作用域之外,这可能会导致不可预知的问题,随着代码量的增大,可能在**10000**行的代码区域,我们突然又需要另外一个变量`b`,我么不加思索定义后,可能会影响这里的值.我们看看下面的改善:

<div class="sourceCode">

``` {.sourceCode .js}
function doSomething(a) {
  function doSomethingElse(a) {
    return a - 1;
  }

  var b;
  b = a + doSomethingElse( a + 2);

  console.log( (b * 3) );
}
doSomething( 2 );
```

</div>

如上,我们就将`b`和``` doSomethingElse``私有 ```为`doSomethig`的变量,和它同级的作用域是不会对这二位不会产生什么影响了,Nice!

-   冲突避免

同名变量的不同值,不同用法可能会产生冲突,我们可以利用**隐藏**的方法来实现好的处理.

<div class="sourceCode">

``` {.sourceCode .js}
function foo() {
  function bar(a) {
    i = 3; //changing the `i` in the enclosing scope's for-loop
    console.log(a + i );
  }

  for ( var i = 0; i < 10; i++) {
    bar( i * 2 ); //oops, infinite loop ahead!
  }
}
foo(2);
```

</div>

这里,循环里的`i`会被`bar`作用域的`i`覆盖,这里就会导致了死循环了.这里`bar`需要一个局部的变量`i`,所以我们在函数作用域中使用`var i = 3;`来解决这个问题.或许,换另外变量的名字可以解决这个问题.

### 全局`命名空间`

另外一种避免全局冲突的方法是在全局声明一个`命名空间`,它可能是一个对象,所有的变量和方法都是作为这个对象的属性来定义,使用时,也是作为这个对象的属性来访问,这就模拟出了`命名空间`了,可以有效避免冲突.

<div class="sourceCode">

``` {.sourceCode .js}
var MyNameSpace = {
  awesome: "stuff",
  doSomething: function() {
    //do something
  },
  doSomethingElse: function() {
    //something else
  }
  a: ""

}
```

</div>

作为作用域的函数
----------------

任何的代码块我们都可以用一个函数来包裹以实现私有,然而它也是不完美的,也存在一些问题:

-   需要一个命名的函数,这意味着这个函数本身对上一级作用域的污染

我们可以用什么更好的办法来实现:

1.  不要函数名的函数
2.  能自动执行

Oops, Javascript offers One pretty NICE one!

<div class="sourceCode">

``` {.sourceCode .js}
var a = 2;
(function foo() {
  var a = 3;
  console.log(a); // 3
})();

console.log(a); // 2
```

</div>

我们拆解来看看,这里发生了一些什么神奇的事情:

1.  `(function foo(){...})();` VS `function foo(){...}`

函数表达式和函数声明的差别,辨别二者的方法很简单,假如`function`出现在语句的最前端,那么它就是一个函数声明,其它都是函数表达式了.而它们之间的区别是函数标志的绑定对象,第一个例子中`foo`绑定到了上级作用域中,而第二个例子中,它仅仅只是绑定在自己的函数作用域中.也就是说`foo`仅仅只能在它自己的作用域中访问到,这就实现了`foo`对外部作用域的隐藏,避免对外部作用域的隐藏.

-   我们真的需要它有名字吗?

我们都对作为回调参数的函数:

<div class="sourceCode">

``` {.sourceCode .js}
setTime( function() {
  console.log("I wanted 1 second!");
}, 1000);
```

</div>

这里,作为回调参数的函数木有名字哟,这就是**匿名函数**,我们的函数表达式可以是匿名的,但是函数声明就不行.当然,匿名函数也不是完美的,它有自己的问题:

1.  难以调试
2.  无法调用本身,可能需要被废弃的`arguments.callee`实现

有名的函数表达式同样强大有力,更重要的是它没有了匿名函数的一些缺陷,所以最佳实践:
总是给你的函数表达式来个名字.

### IIFI(立即调用函数)

\_\_

<div class="sourceCode">

``` {.sourceCode .js}
var a = 2;
(function foo() {
var a = 3;
console.log( a ); // 3
})();

console.log( a );
```

</div>

这里我们有一个`foo`函数定义被`()`包裹起来了,这是个什么鬼呢?而且它后面还接着一个括号,其中第一个圆括号使得函数定义成为了一个表达式,而第二个括号是对它的执行,当然,立即调用函数不需要一个函数名字,但是有的话会有以上我们提到的好处,但是却没有什么坏处可言的,所以,总是给IFFI函数来个函数名是很好的实践.

<div class="sourceCode">

``` {.sourceCode .js}
var a = 2;
(function IIFE() {
  var a = 3;
  console.log( a );
})();

console.log( a );
```

</div>

还有一类普遍使用IIFE函数,有传入参数的立即调用函数:

<div class="sourceCode">

``` {.sourceCode .js}
var a = 2;
(function IFFE(global) {
  var a = 3;
  console.log( a );
})(window);

console.log( a );
```

</div>

关于IIFE我们先就说这么多,其实它还有的许多的应用的.

块作用域
--------

函数作用域是普遍的作用域单元，但是其他形式的作用域的存在可能会使得代码的维护更好，和其他语言不通，Javacript中不存在块级作用域：

<div class="sourceCode">

``` {.sourceCode .js}
for (var i = 0;i < 10; i++) {
    console.log( i );
}
```

</div>

上述代码中，我们在`for`循环中定义了`i`，本意应该是只在循环中使用它，一旦循环结束或者退出，它就没有对应的定义了，但是在Javascript中，它依然能被访问到，它被定义到了包含它的函数作用域。

<div class="sourceCode">

``` {.sourceCode .js}
var foo = true;
if(foo) {
    var bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
```

</div>

同样，这里本意是只在`if`块中可以访问的`bar`变量，同样会暴露到上级函数作用域中，这就给代码的维护带来很多的麻烦。然而，我们的Javascript却没有给我们提供块级作用域的声明，至少现在是这样子的，那我们有没有办法来`伪造`一个块级作用域呢？

### with

这个东西是个恶魔，它带来的好处小于坏处，直接弃用

### try/catch

这是一个很少人知道的技巧，ES3中，`catch`中定义的变量是只属于`catch`这个块的。

<div class="sourceCode">

``` {.sourceCode .js}
try {
    undefined(); //错误的调用
}
catch(err) {
    console.log( err ); //works
}

console.log( err ); //errors 
```

</div>

如上，
`catch`块外访问`err`会报错，然而我们很多的语法检测都会提示统一作用域中多次声明的`err`，这本质是不对的，因为每个`catch`块都是一个块作用域，是不会被影响到的。但是这个语法好像用的不是很多，我们先放置下。

### let

ES6给我们来了一个原生的块级作用域的定义，这使得Javascript中没有块级作用域将成为过去，作为和`var`同级的变量定义关键字，它能绑定声明的变量到当前作用域中，是一个很完美的块级作用域变量声明：

    var foo = true;
    if (foo) {
        let bar = foo * 2;
        bar = something( bar );
        console.log( bar );
    }

    console.log( bar ); // RefereceError

通过`let`我们可以简单的实现跨级作用域，然而，对于以下需要提到的变量提升，`let`不会有变量提升：

<div class="sourceCode">

``` {.sourceCode .js}
{
    console.log( a ); // ReferenceError!
    let a = 2;
    console.log( a ); // 2
}
```

</div>

具体关于变量提升，我们慢慢道来！

### 垃圾回收机制

### `let`循环

块级作用域更常用于`for`循环中：

<div class="sourceCode">

``` {.sourceCode .js}
for (let i = 0; i < 10; i++) {
    console.log( i );
}
console.log( i ); // ReferenceError;
```

</div>

这就是我么期望的，`i`只在我们的循环中存在，在外部作用域中是没有定义的。当然，重构代码时候一定要注意`let`不是万能的，它不能代替我们的`var`，运用是需要留心。

### `const`

同样是ES6中引入，同样也能创建块级作用域，它更适用于声明常量，任何对它值得修改都会报错的。

<div class="sourceCode">

``` {.sourceCode .js}
var foo = true;
if (foo) {
    var a = 2; 
    const b = 3; // `if`块级作用域变量
    a = 3; //正常的赋值
    b = 3; //error!
}
console.log( a ); // 3
console.log( b ); //RefereceError!
```

</div>

总结
----

函数作用域是Javacript中最常用的作用域单元，定义在函数的中变量和函数在其他同级函数中是隐藏的，不可访问的，但是函数作用域不是唯一的作用域单元：

-   ES3: try/catch
-   ES6：let and const

变量提升
--------

我们现在对作用域有相当的认识了，不管函数作用域还是块级作用域它们都遵守相同的规则：在某一作用域中声明的变量只存在于这个作用域。但是，具体是如何绑定的呢？

### 先有鸡还是先有蛋？

我们很容易以为，我们的代码都是从上到下，一行一行的执行的，但是有些情况可能就不是这么回事了：

<div class="sourceCode">

``` {.sourceCode .js}
a = 2;
var a;
console.log( a );
```

</div>

那这里我们能得到什么呢？按照我们上述的普遍认为的规则，我们可能会得到`undefinded`，，但结果却不然，是`2`。

<div class="sourceCode">

``` {.sourceCode .js}
console.log( a );
var a = 2;
```

</div>

这个呢？

基于以上，有可能认为这个是`2`，但是`console.log(...)`执行在`a`定义之前，应该是`ReferenceError`，然而结果却不然，是`undefined`，这到底是什么鬼啊？到底变量声明在前还是赋值在前呢？

要说明这个问题，我们可以回忆下第一章说的关于引擎兄弟和它小伙帮的故事了，引擎兄弟在执行代码之前需要一个编译的过程，而编译中有一个步骤就是要先找到某变量的声明并关联它的作用域。

因此，我们可以这么理解：所有的声明，包括变量和函数的声明在其他所有代码之前。

但你看到`var a = 2;`时，我们可能认为这是一个语句，然而对于`Javascript`而言却不是，它是两个语句`var a;`和`a = 2;`首先是声明变量，接着才是赋值操作。所以我们的第一个代码块应该是这个样子：

首先声明变量：

<div class="sourceCode">

``` {.sourceCode .js}
var a;
```

</div>

然后是赋值和其他操作：

<div class="sourceCode">

``` {.sourceCode .js}
a = 2;
console.log( a );
```

</div>

同样地，第二段代码就是：

<div class="sourceCode">

``` {.sourceCode .js}
var a;
```

</div>

紧接着：

<div class="sourceCode">

``` {.sourceCode .js}
console.log( a );
a = 2;
```

</div>

这样子就能得到我们开始的值了。对于这种机制，我们称为**变量提升**，需要注意的是，仅仅只有变量声明会有提升，而它的赋值会出现在它本身就在出现的位置，就如以上的代码一样。

<div class="sourceCode">

``` {.sourceCode .js}
foo();
function foo() {
    console.log( a ); //undefined
    var a = 2;
}
```

</div>

`foo`声明会提升到代码顶部，所以我们能调用`foo(...)`,同样，`a`声明也被提升，但是赋值却在`console.log(...)`之后，所以我们得到`undefined`而不是`ReferenceError`，就是这么简单，实际的代码是这么个样子：

<div class="sourceCode">

``` {.sourceCode .js}
function foo() {
    var a;
    console.log( a ); //undefined
    a = 2;
}
foo();
```

</div>

函数声明会被提升，但是函数表达式是不会被提升的：

<div class="sourceCode">

``` {.sourceCode .js}
foo(); //not ReferenceError, but TypeError!
var foo = function bar() {
    //
};
```

</div>

这里，变量`foo`声明被提升，所以它不是`ReferenceError`，但是`foo`还没有任何的值，因此`foo()`就是对调用`undefined`，这是一个`TypeError`，非法操作。

我们还要注意到，尽管是一个函数表达式，`bar`早上一级同样不是有效的：

<div class="sourceCode">

``` {.sourceCode .js}
var foo;
foo(); //TyepErro
bar(); //ReferenceErro
foo = function() {
    var bar =  function() {...}
    //
}
```

</div>

变量和函数都会被提升，存在重复声明时,函数会先有变量的提升：

<div class="sourceCode">

``` {.sourceCode .js}
foo(); //1
var foo;
function() {
    console.log( 1 );
}
foo = function() {
    console.log( 2 );
};
```

</div>

定义引擎而言，上面的代码是这个样子的：

<div class="sourceCode">

``` {.sourceCode .js}
function foo() {
    console.log( 1 );
}
foo();
var foo = function() {
    console.log( 2 );
};
```

</div>

注意到`var foo;`不见了，这是因为函数提升先有变量提升，所以它被引擎忽略了。变量的声明会别忽略，但是函数的重复声明会有覆盖效果：

<div class="sourceCode">

``` {.sourceCode .js}
foo(); //3
function foo() {
    console.log( 1 );
}

var foo = functin() {
    console.log( 2 );
};

function foo() {
    console.log( 3 );
}
```

</div>

### 总结

引擎兄弟会把我们眼中的`var a = 2;`看成两部分，一部分`var a;`提升至代码块顶端，而另一部分则是在它出现的原来位置`a = 2;`这直接导致不管变量声明的位置在哪儿，声明都会被提到当前作用域的前端，这就是所谓的**变量提升**

变量会提升，但是赋值，甚至函数表达式的赋值都不会有提升。

作用域封闭（闭包）
------------------

我们很好的学习并理解了作用域，现在我们要开始对作用域封闭来一个完整的学习了。

闭包无处不在！！！

闭包自然而然就出现了，它依靠于词法作用域，它就是那样子出现了，没有什么可以的安排，我们有时候不知道，仅仅是因为我们对它**视而不见**。

> 闭包是指函数词法作用域之外函数及其词法作用域能被访问的现象。

