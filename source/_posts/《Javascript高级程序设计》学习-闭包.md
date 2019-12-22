---
title: 《Javascript高级程序设计》学习 | 闭包
date: 2019-11-26 10:18:45
categories: 
	- Javascript学习笔记
tags: 
	- Javascript
	- 前端
---

> 《Javascript高级程序设计》学习笔记三
>
> 这是我在学习Js红皮书的学习记录
>
> 本篇笔记主要记录了函数表达式、闭包方面的知识

<!-- more -->

### 函数表达式

定义函数的两种方式：

+ 函数声明

  ```javascript
  function functionName(arg0, arg1, arg2){
  	// dosomething
  }
  ```

+ 函数表达式

  ```javascript
  var functionName = function(arg0, arg1, arg2){
  	// dosomething
  };
  ```

虽然以上两种方式都创建了名为`functionName`的函数，但两者还是有区别的：

+ 函数声明会产生函数声明提升。在全部代码执行前，解析器会先读取函数声明并添加到执行环境中，即相当于函数都是在代码一开始就声明了的。这意味着可以把函数声明放在调用它的语句之后，调用时也不会报未定义的错误；

+ 函数表达式类似赋值语句，将一个匿名函数赋值给了一个变量。因此，与其他表达式一样，使用前必须先赋值。

### 函数递归

递归通常是通过调用自身实现的，如下求阶乘的函数：

```javascript
function factorial(num){
	if(num <= 1){
        return 1;
    }else{
        return num * factorial(num - 1);
    }
}
```

但通过名字的直接调用自身可能会导致错误：

```javascript
var anotherFactorial = factorial;
factorial = null;
alert(anotherFactorial(4)); // 出错
```

上述代码把函数保存在另一个变量中，然后将`factorial`变量置空，那么接下来在调用函数时，由于函数中的`factorial`不再是函数了，所以会导致错误。此时，使用`arguments.callee`代替函数中的`factorial`可以解决问题，这是一个指向正在执行函数的指针（即指向自己）。

但严格模式下对于`arguments.callee`无法访问，可以通过以下方式：

```javascript
var factorial = (function f(num){
    if(num <= 1){
        return 1;
    }else{
        return num * f(num - 1);
    }
});
```

以上代码创建了一个名为`f()`的❓<u>**命名函数表达式**</u>❓并赋值给变量`factorial`，此时即使把函数赋值给了另一个变量，名字`f`仍然有效。

### 闭包

#### 原理

闭包是一个非常强大的功能，是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式就是在函数内部创建另一个函数，由于作用链的关系（内部函数的作用链包含外部函数的作用链），内部的函数可以访问外部函数中的变量。同时，即使内部函数被返回了并在其他地方调用，它仍然可以访问到原来的变量。

书中的例子如下（这个函数的功能是用于创建基于对象不同属性的比较函数，用于排序）：

```javascript
function createComparisonFunction(propertyName){
    return function(object1, object2){
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        
        if(value1 < value2){
            return -1;
        }else if(value1 > value2){
            return 1;
        }else{
            return 0;
        }
    }
}
```

之所以会产生内部函数返回后仍然可以访问外部函数变量这种神奇的现象，就要理解作用域链的相关知识：

+ 当某个函数被调用时，会创建一个执行环境（execution context），其中包含相应的作用域链。然后使用`arguments`和其他命名参数的值初始化函数的活动对象（或变量对象，activation object）。活动对象存在于作用域链中，并且当前函数的活动对象排在第一位，外部函数的活动对象排在第二位…直至全局执行环境的活动对象。
+ 当函数执行，需要读写变量时，就会到作用域链中查找对应的变量。作用域链本质上相当于一个指向变量对象的指针列表，只是引用但不实际包含变量对象。
+ 全局环境的活动对象始终存在，而一般情况下函数执行完毕后，其局部活动对象就会被销毁。
+ 但对于闭包，这个在一个函数内部定义的函数会将外部函数的活动对象添加到它的作用域链上，这样内部的函数才可以访问外部函数的变量。但在其被返回后，这个被返回的匿名函数的作用域链仍然引用着外部函数的活动对象，导致了外部函数的活动对象不会被销毁。也就是说，外部函数返回后，其执行环境的作用域链虽然销毁了，但它的活动对象仍然会留在内存中，直到匿名函数被销毁（置空），引用解除。

#### 副作用

##### 变量

我们回顾一下前文所说的，闭包现象的发生是因为保存了外部函数的整个活动对象，并且是通过作用域链引用这个对象。因此，其保存的不是一个特定时刻的特殊值，之后的调用只能取得外部函数任何变量的最终值。

```javascript
function createFunctions(){
	var result = new Array();
    for(var i = 0; i < 10; i++){
        result[i] = function(){
          return i;  
        };
    }
    return result;
}
```

上述代码中得到的函数数组都返回10，因为它们引用着同一个保存变量`i`活动对象，该对象中`i`的值为10。

当然，也有解决方法：

```javascript
function createFunctions(){
	var result = new Array();
    for(var i = 0; i < 10; i++){
        result[i] = function(num){
            return function(){
                return num;
            };
        }(i);
    }
    return result;
}
```

我们没有直接把闭包赋值给数组，而是定义了一个匿名函数，并把这个匿名函数立即执行后的结果赋给数组。由于函数参数是按值传递的，所以每次循环时，变量`i`的当前值被复制给变量`num`，而在匿名函数内部，又创建并返回了一个访问`num`的闭包，这样一来，`result`数组中每个函数就有自己的`num`变量副本了，不再是同一个！

##### this对象

首先看以下代码：

```javascript
var name = "The window";

var object = {
    name: "My Object",
    getNameFunc: function(){
        return function(){
            return this.name;
        };
    }
};

alert(object.getNameFunc()()); //"The window"(非严格模式下)
```

以上代码在对象中定义了一个方法，该方法返回一个匿名函数，而匿名函数又返回`this.name`。但我们可以看到，最终的结果为`The window`，即匿名函数并没有取得外部函数的`this`对象。

原因是：每个函数在被调用时都会自动取得两个特殊变量：`this`和`arguments`，内部函数在搜索这两个变量时，只会搜索到自己的活动对象，也就是说内部函数不可能访问到外部函数的这两个变量，也就不存在保存不保存的问题了。

不过，也是有方法实现闭包对于外部函数的`this`对象的访问的：

```
var name = "The window";

var object = {
    name: "My Object",
    getNameFunc: function(){
    	var that = this;
        return function(){
            return that.name;
        };
    }
};

alert(object.getNameFunc()()); //"My Object"(非严格模式下)
```

以上代码将外部函数的`this`赋值给一个变量`that`，这样内部函数在搜索`that`变量时就可以访问到这个外部函数定义的变量了，同时返回后仍然引用着`that`。

##### 内存

需要注意的是，基于前文的分析我们可以看到，闭包会引用外部函数的整个活动对象，只要匿名函数存在，活动对象所占内存不会被回收。因此闭包会比其他函数占更多的内存，过度使用闭包可能会导致内存占用过多。

### 闭包使用

#### 模仿块级作用域

```javascript
function outputNumbers(count){
	for(var i = 0; i < count; i++){
        alert(i);
    }	
    alert(i);
}
```

Javascript中没有块级作用域，以上代码中在循环外还可以访问到变量`i`。原因是：在块语句中定义的变量，实际上是在包含函数中而非块语句中创建的，即变量`i`是定义在包含函数的活动对象中的，因此从有定义开始，在函数内部随处都可以访问。

通过闭包，我们还可以模仿块级作用域：

```javascript
(function(){
    // 这里是块级作用域
})();
```

以上代码定义并立即调用了一个匿名函数并立即调用。将函数声明包含在一对圆括号中，表示实际上是一个函数表达式，即我们直接用函数表达式的值取代了函数名的位置。（注意一定要给函数体加括号，因为如果直接用`function`开头，会被认为是函数声明的开始，不允许后面立即跟圆括号，而函数表达式可以）

用这样的方式，相当于我们将变量定义在了匿名函数的活动对象中，而匿名函数在立即执行结束后就会被销毁了。而块级作用域又能访问到外部变量，因为它是一个闭包。

以上技术经常被用在全局作用域中以限制向全局作用域中添加过多的变量和函数，防止合作开发时过多的全局变量和函数导致的命名冲突。

#### 私有变量

严格来讲，JavaScript中没有私有成员的概念，所有对象属性都是公开的，但确实有一个类似私有变量的概念，即我们在函数中定义的变量在函数的外部是不能访问的。但结合我们上文提到的闭包，其作用就是在外部访问内部的变量，那么，我们是否可以利用闭包，实现用于访问私有变量的公有方法呢？答案是肯定的。这种方法称为特权方法，创建方式有两种：

##### 在构造函数中定义

```javascript
function MyObject(){
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction(){
        return false;
    }
    
    // 特权方法
	this.publicMethod = function(){
        privateVariable++;
        return privateFunction();
    }
}
```

上述特权方法作为闭包有权访问在构造函数中定义的所有变量和函数。在用上述构造函数创建对象实例后，除了使用特权方法，是没有其他办法访问到私有变量和方法的，因此，使用这种技巧可以隐藏那些不应该被直接修改的数据。

在构造函数中定义特权方法有一个缺点，就是针对每个实例都会创建同样的新方法，因此，可以使用静态私有变量来实现特权方法。

##### 在原型上定义静态私有变量

通过在私有作用域中定义私有变量和函数：

```javascript
(function(){
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction(){
        return false;
    }    
    // 构造函数
    MyObject = function(){
        
    };
    
    // 公有/特权方法
    MyObject.prototype.publicMethod = function(){
        privateVariable++;
        return privateFunction();
    };
})();
```

这个模式创建了一个私有作用域，并在其中定义了私有变量和私有函数，然后定义了构造函数和公有方法，公有方法在原型上定义。

**需要注意的是，这个模式在定义构造函数时使用的是函数表达式，因为函数声明只能创建局部函数，而我们需要在外部也可以使用这个构造函数。同时，表达式没有使用`var`也是出于同样的目的目的就是创建一个全局变量。（但也要知道，在严格模式下，给未经声明的变量赋值会导致错误，所以我们可以先提前声明好。）**

这个模式和在构造函数中定义特权方法的主要区别在于实现了代码复用，但私有变量是由实例共享的。因为我们把特权方法定义在了原型上，所有实例使用这同一个函数，而这个特权方法作为闭包保存着对包含作用域活动对象的引用，所有私有变量也就是静态且共享的了，在任何实例中改变会影响其他所有实例。

### 模块模式

#### 创建单例

前面的模式是为自定义类型创建私有变量和特权方法的，还有一种模块模式（module pattern）则是为单例创建私有变量和特权方法的。所谓单例，即只有一个实例对象，说白了就是以字面量方式创建的单例对象.

```javascript
var singleton = function(){
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction(){
        return false;
    }   
    // 特权/公有方法和属性
    return {
        publicVariable: true,
        publicMethod: function(){
            privateVariable++;
        	return privateFunction(); 
        }
    };
}();
```

看代码我们可以知道，这个模式返回了一个对象（对象字面量），返回的对象字面量只能访问公开的属性和方法，而公有方法作为闭包可以访问到私有变量和函数。注意到这里的函数立即执行不需要像上面一样套括号了，应该是这里是函数表达式的原因。

这种模式创建的单例都是Object的实例，因为是通过对象字面量的方式表示的。

#### 创建特定类型的单例

```javascript
var singleton = function(){
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction(){
        return false;
    }   
    
    // 创建特定类型对象
    var object = new CustomType();
    
    // 添加特权/公有方法和属性
    object.publicVariable = true,
    object.publicMethod = function(){
        privateVariable++;
     	return privateFunction(); 
    };
    
    // 返回对象
    return object;
}();
```
