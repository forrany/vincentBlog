---
layout:     post
title:      "JavaScript中的几种设计模式"
subtitle:   
date:       2018-12-16 12:00:00
author:     "Vinecnt Ko"
header-img: "img/post-bg-unix-linux.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 前端开发
    - JavaScript
    - 设计模式
---

##  单例模式

```javascript
 
var person = {
	name : "Tom",
	age: 12
}

var person1 = {
	name : "toony",
	age : "11"
}

```

这就是单例模式
 - 作用： 实现分组，把描述同一个事物的属性和方法放在了同一个空间中，起到了分组的作用，避免了相同变量名之间的干扰
 - 在单例模式中，我们把对象名叫**命名空间**
 - 单例模式是我们项目中常用的模式，因为可以利用它进行模块化开发。

```javascript
//同样都用change方法，为了避免污染
var sideBar = {
	change : function (){}
}

var headBar = {
	change : function (){}
}

```

## 工厂模式
为了实现”低冗余，高聚合“的方式，为了批量生产，于是有了工厂模式。 

```javascript

function creatPerson(name,age){
	var obj = {};
	obj.name = name;
	obj.age = age;
	obj.writeJs = function () {
		console.log('ha' + obj.name + 'can write JS');
	}
	return obj;
}

```
于是，不用像单例模式那样，进行多次的重复的写内容了

```javascript
var person1 = creatPerson("tom",15);
var person2 = creatPerson("tony",22);

```
**特点**：
 1. 把实现相同功能的代码放到一个函数中，以后再想实现这个功能，不需要重新编写代码，只要运行代码即可   >>>我们把这个叫**函数的封装**    ==低耦合，高内聚==

### 面向对象的编程语言
 - 封装   封装已经实现，就是函数
 - 继承   通过原型链实现
 - 多态   当前方法和类的多种姿态，包括 重载和重写

#### 多态之重载
方法名相同，参数类型、个数不同。
但是，JS中没有重载，因为当重复写的时候，会被覆盖

```javascript
function sum (num,num){}
function sum (num) {}
```
这是不行的，因此JS是没有重载的(overload)！！
但是可以通过if语句判断，来模拟实现重载(overload)
**但是JS有多态，因为有重写**

### 构造函数模式

```javascript
function CreatPerson (name,age) {
	this.name = name;
	this.age = age;
	this.writeJs = function (){
		console.log("my name is " + this.name + "I can write JS"); 
		}

}

var tom = CreatPerson("tom",12);
```
这就是构造函数设计模式，注意方面里面的this是在调用的时候，才知道指向谁； 而前面的属性上的this，则是在构造函数时，指向实例。 因为构造函数，遇到new 的时候，会自己创建一个this对象，最后再自动返回this对象。

这种，在内建对象中，也有类似的模式，比如创建数组：
```javascript
var arr = [];
var arr1 = new Array();
```
不管哪组方式，arr都是 Array的实例。
但是注意： 不同实例的方法虽然名称相同，但是是不同的东西，那是各自实例的私有属性，单独的个体。
#### 这个不是单例模式，而是构造函数模式
有的时候，面试官认为这个是单例模式，实际上，这并不是，这个是构造函数模式，也创造了单独的对象，但是在W3C中是可以查询的！
 1. JS中，所有类都是函数数据类型的，它与函数没有很大的区别。只不过在new的时候，会自动创建一个this对象，最后再自动返回。 (注意，实例都是object类型)

 2. 构造函数中，this.xxx中的this都是指向实例本身。 在方法中，this需要等待调用才有所指代

 3. p1和p2中都是同一个类的实力，都有相同的方法，但是那是私有属性，是单独个体，不相等。

#### 还有一些特点(扩展)
- var p1 = new CreatPerson 在构造函数中，如果没有参数，括号可以省略.  但是函数执行是不可以省略的
- 如果提前return了一个对象，则会返回新的对象；如果返回一个基本数据类型，则没有任何影响
- 测一个实例是否是一个类的实力  `instanceof`  

```javascript
console.log(p1 instanceof CreatPerson);  //true
console.log(p1 instanceof(CreatPerson)); //true
console.log(p1 instanceof Array) ;  //false
console.log(p1 instanceof Object) ; //true
```

检测数据类型来说，typeof不能准确检测Array的具体类型(只能返回object)。 可以使用`istanceof`

```javascript
var arr = [];
console.log(arr istanceof Array);  //true
//也可以使用Object下的
console.log(Object.prototype.toString(arr)) //返回[object Array]
```

- 检测某一个实例是否有某个属性  `in`

```javascript
var obj = {
	name : "tom",
	age : 12
}
console.log("age" in obj) //true;
```
注意in无论是共有(prototype)还是私有的，都会返回。 因此，为了检测私有属性，有hasOwnProperty

- 检测某一个属性是否是私有属性 `hasOwnProperty`

```javascript
console.log(obj.hasOwnProperty('age')) //true;
```

- 检测某一个实例是否有某个属性  `in`


```javascript
function hasPubProperty (obj,property) {
	return property in obj && !obj.hasOwnProperty(property);
}
```


### 原型链模式


- 数组去重,链式写法

```javascript
Array.prototype.myUnique = function () {
	var obj = {};
	for(var i = 0; i < this.length; i++ ){
		var cur = this[i];
		if(obj[cur] == cur){
			this[i] = this[this.length -1];
			this.length--;
			i--;
			continue;
		}
		obj[cur] = cur;
	}
	return this;  //实现链式写法
}
```
- 写一个mySlice方法

```javascript
Array.prototype.mySlice = function (start,end) {
	var arr = [],
		rArr = [];
	start < 0? start += this.length : start;
	end < 0? end += this.length : end;
	end >= this.length? end = this.length : end;
	for(var i = 0; i< this.length - 1 ; i++){
		if(start <= i){
			arr[arr.length] = this[i];
		}
		if(end == i-1){
			break;
		}
	}
	return arr;		
}
```

#### 原型链批量设置共有属性的小问题

当有多项属性、方法需要在原型链上配置的时候，可以使用批量设置，但是注意，一定要设置好constuctor

```javascript
Myfun.prototype = {
	constructor: Myfun,
	this.show: function () {
		console.log(this.name);
	},
	...

}
```
![](http://ww1.sinaimg.cn/large/6f9f3683ly1fy8tozcht2j21510m415m.jpg)
**记住这张图中的内容，这种设置方式相当于新开辟的一个内存地址来保存prototype，如果没有设置contructor，这个时候，constructor 就会指向Object。**

另一方面，如果直接内置的类上批量添加，因为会覆盖原有的很多方法，因此浏览器会屏蔽这种设置方法

```javascript
Array.prototype = {}
//这种方法无效
```


## 原型继承
1. 最常用的方法；
2. 子类B想要继承父类A中所有的属性和方法（私有+共有)，让B的prototype = A的实例。`B.prototype = new A`
3. 原型继承特点，父类私有和共有的都变成了子类私有的方法和属性。
## 圣杯模式

```javascript
var inherit = (function () {
	var function F() {}
	return function (Origin,Target){
		F.prototype = Origin.prototype;
		Target.prototype = new F；
		Target.prototype.constructor = Target;
		Target.prototype.uber = Origin;
}
} ())

```
## 中间类继承
这个比较特殊，主要用在一个问题上:
> 当遇到argument[]这种类数组时，无法使用数组的方法，很蛋疼。比如想封装一个平均值的方法

```javascript
function avgFn() {
	var sum;
	Array.prototype.sort.call(arguments,function (a,b) {return a-b});  //升序排列，扔掉一个最大，一个最小
	Array.prototype.pop().call(arguments);
	Array.prototype.shift().call(arguments);
	for(var i = 0; i < arguments.length; i++){
		sum += arr[i]
	}
	return sum/arguments.length;
}
```
显得很麻烦，因为要通过调用Array的方法。所以，这里可以直接修改类数组(同样也是对象)的`__proto__` 属性
```javascript
arguments.__proto__ = Array.prototype;
...
```