---
title: 《Javascript高级程序设计》学习 | 对象、原型
date: 2019-11-22 20:26:41
categories: 
	- Javascript学习笔记
tags: 
	- Javascript
	- 前端
---

> 《Javascript高级程序设计》学习笔记二
>
> 这是我在学习Js红皮书的学习记录
>
> 本篇笔记主要记录了对象创建、原型链方面的知识

<!-- more -->

类可以算是面向对象语言的一个标志，通过类的方式我们可以对于属性和方法进行良好的封装，并以此创建任意个具有相同属性和方法的对象。但ECMAScript中是没有类的概念的，在ECMAScript中，对象被定义为：

> 无序属性的集合，其属性可以包含基本值、对象或者函数。

因为每个属性都有名字，名字对应到一个值，ECMAScript的对象也可以看成一组名值对。**每个对象都是基于一个引用类型创建的**。

### 对象属性的特性

对象的属性在创建时都带有一些特征值（characteristic），JavaScript可以通过这些特征值定义属性的行为。_定义这些特性是为了实现_ _JavaScript_ _引擎用的，所以无法直接访问，为了表示特性是内部的，将其放在两队方括号中_。

ECMAScript属性分：数据属性和访问器属性两种属性。两者的差别见下文`_year`和`year`两个属性，前者为数据属性，后者为访问器属性。

+ **数据属性**：包含一个数据值的位置，在这个位置可以读取和写入值。数据属性包含以下4个特性：

  + [[ configurable ]]：表示能否通过delete删除属性、能否修改属性特性、能否把属性改为访问器属性，默认为`true`。
  + [[ enumerable ]]：表示能否通过`for-in`循环返回属性，默认为`true`。
  + [[ writable ]]：表示能否修改属性，默认为`true`。
  + [[ value ]]：包含属性的数据值，默认为`undefine`。

  修改属性默认特性的方法，通过ES5的`Object.defineProperty()`方法，该方法接收三个参数：属性所在的对象，属性的名字和一个描述符对象，如下：

  ```javascript
  var person = {};
  Object.defineProperty(person, "name", {
  	writable: false,
  	value: "Nicholas"
  });
  
  alert(person.name);    //"Nicholas"
  person.name = "Greg";
  alert(person.name);    //"Nicholas"
  ```

  以上例子把`name`属性改为只读，非严格模式下会忽略赋值操作，严格模式下会抛出错误。

  **对于configurable特性来说，一旦定义为false，就再也变不会true了，因为其表示不可配置。并且，此时如果调用`defineProperty()`方法修改除writable外的特性，都会导致错误**

  **在调用`defineProperty()`，如果不指定任何特性，那么都会默认变为false。**

+ **访问器属性**：不包含数据值，而包含一对`getter`和`setter`函数，有以下4个特性：

  + [[ configurable ]]：表示能否通过delete删除属性、能否修改属性特性、能否把属性改为数据属性，默认为`true`。
  + [[ enumerable ]]：表示能否通过`for-in`循环返回属性，默认为`true`。
  + [[ get ]]：读取属性时调用的函数，默认为`undefine`。
  + [[ set ]]：写入属性时调用的函数，默认为`undefine`。

  ```javascript
  var book = {
  	_year: 2004,   // 下划线只是一种记号，表明只能通过对象访问
  	edition: 1
  };
  
  Object.defineProperty(book, "year", {
  	get: function(){
  		return this._year;
  	},
  	set: function(newValue){
  		if(newValue > 2004){
  			this._year = newValue;
  			this.edition += newValue - 2004;
  		}
  	}
  });
  
  book.year = 2005;
  alert(book.edition); // 2
  ```

  使用访问器属性的常用方式就是如上所示的，通过设置一个值导致另一个值改变。

  **只指定getter意味着属性不能写，只指定setter意味着属性不能读**

通过`Object.defineProperties()`方法可以通过描述符一次定义多个属性，使用`Object.getOwnPropertyDescriptor()`方法可以取得给定属性的描述符。

### 创建对象

创建对象有以下几种方式：

+ `Object`引用类型的构造函数或字面量。缺点：代码无法重复利用，每次都要重新写一遍

+ 工厂模式。将创建的过程抽象和隐藏，封装在一个函数中，并在这个函数中返回创建的对象：

  ```javascript
  function createPerson(name, age, job){
  	var o = new Object();
  	o.name = name;
  	o.age = age;
  	o.job = job;
  	o.sayName = function(){
  		alert(this.name);
  	};
  	return o;
  }
  
  var person1 = createPerson("Nicholas", 29, "software Engineer");
  ```

  **优点：实现了创建多个相似对象的代码复用；缺点：没有解决对象识别问题（不知道这个对象的类型）**

+ 构造函数模式。像内置的Object和Array原生构造函数一样，我们可以自定创建构造函数（与普通函数唯一区别，通过new调用）

  ```javascript
  function Person(name, age, job){
  	this.name = name;
  	this.age = age;
  	this.job = job;
  	this.sayName = function(){
  		alert(this.name);
  	};
  }
  
  var person1 = new Person("Nicholas", 29, "software Engineer");
  ```

  通过这种方式创建对象会经历以下四步：

  + 创建一个新对象
  + 将构造函数的作用域赋给新对象，即this指向了新对象
  + 执行构造函数里的代码，为新对象添加属性
  + 返回新对象

  通过构造函数创建的对象，可以通过以下方式标识对象类型。（比工厂模式好的原因）

  ```javascript
  alert(person1.constructor == Person); //true
  alert(person1 instanceof Object);     //true
  alert(person1 instanceof Person);     //true
  ```

  **优点：可以将对象标识为特定的类型。仍有缺点：相同的方法，却在每个实例上都重新创建了一遍**。解决方法：

  1. 把函数定义移到外面，创建一个全局函数，在构造函数中，都将这个全局函数赋值给对象的对应属性。由于函数名是指针，所以这样一来通过指向同一个全局函数实现共享。但是这样一来，函数一多就会需要创建很多全局函数，并且没有封装性可言。
  2. 通过使用原型模式解决

### 原型模式

创建的每个函数都有一个`prototype`（原型）属性，这个属性是一个指针，指向一个对象，这个对象的用途是包含可以由特定类型所有实例共享的属性和方法。即`prototype`是通过调用构造函数创建的那个实例对象的原型对象

在使用构造函数时，通过将信息添加到构造函数的原型对象中，可以实现调用该构造函数创建的所有对象实例信息的共享。

```javascript
function Person(){}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "software Engineer";
Person.prototype.sayName = function(){
	alert(this.name);
};


var person1 = new Person();
person1.sayName(); //"Nicholas"

var person2 = new Person();
person2.sayName(); //"Nicholas"

alert(person1.sayName == person2.sayName); // true
```

#### 理解原型对象

无论什么时候，只要创建一个函数，就会根据一组特定的规则为函数创建一个`prototype`属性，指向函数的原型对象，同时，在默认情况下，所有原型对象也会自动获得一个`constructor`（构造函数）属性，这个属性包含一个指向`prototype`属性所在函数的指针。例如，前面的例子中，`Person.prototype.constructor`指向`Person`，因此，通过这个构造函数即`Person()`，我们还可以添加新的非共享的属性。

> （看到这里可能有点糊涂了，大概的意思就是我们之前定义的这个函数可以看作某个原型对象的构造函数`constructor`，而这个原型对象可以通过函数的`prototype`访问，直接添加到原型对象上的函数被共享，而构造函数中创建的是非共享的属性）

创建自定义构造函数后，其原型对象初始默认只会取得`constructor`属性，其它的继承自Object。（`constructor`属性也是共享的，这也是上文曾出现的`alert(person1.constructor == Person); //true`的原因）

同时，当用构造函数创建了一个新实例后，这个<u>新实例内部也有一个指针（内部属性）直接指向构造函数的原型对象</u>。**注意：实例的指针指向原型，不指向构造函数**。整个指向逻辑见下图：

{% asset_img 1.jpg %}

```javascript
alert(Person.prototype.isPrototypeOf(person1));             //true

alert(Object.getPrototypeOf(person1) == Person.prototype);  //true
alert(Object.getPrototypeOf(person1).name);                 //"Nicholas"
```

虽然实例对象中并没有属性，但我们之所以可以访问，是因为当代码读取属性是，如果没找到，就会向上搜索指针指向的原型对象，直到找到或没找到。使用`hasOwnProperty()`方法可以检测一个属性是否是存在实例中（该方法继承自Object）。

虽然可以通过实例对象访问保存在原型中的值，却不能重写，如果添加了重名属性，会屏蔽原型中的属性。因为此时直接找到了，不必到原型对象中找。（通过delete删掉实例对象中的属性可以恢复对于原型对象属性的访问）。

#### 简便写法

```javascript
function Person(){}

Person.prototype = {
    // constructor: Person
	name: "Nicholas";
	age: 29;
	job: "software Engineer";
	sayName: function(){
		alert(this.name);
	}
};
```

以上无注释的写法将`Person.prototye`设置成了一个新对象，虽然最终结果相同，但是有一点例外，`Person.prototye.constructor`属性不再指向`Person`了。因为我们重写了原型对象，所以`constructor`属性变成了这个新对象的`constructor`属性（指向Object构造函数）。此时，前文出现的构造函数判断将返回`false`。

如果构造函数属性很重要，可以加上注释这一行，特意设回适当的值。

#### 原型的动态性

我们可以随时在原型对象上添加属性，由于实例对象保存着指向原型对象的指针，这个新添加的属性就可以被找到。但是，如果我们重写了原型对象，那么就有可能出错，因为实例对象的指针指向的仍然是最开始的那个原型对象。