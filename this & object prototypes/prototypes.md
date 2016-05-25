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




























































