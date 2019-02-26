---
title: JS中的对象
tags: 
 - JavaScript
---
js支持面向对象编程，但不使用类或者接口。
### 创建对象

#### 工厂模式

```javaScript
function createPerson(name ,age, job) {
	var o = new Object()
	o.name = name
	o.age = age
	o.job - job
	o.sayName = function() {
		alert(this.name)
	}
	return O
}

var person1 = createPerson('nini', 29, 'DOc')
```

解决了创建多个相似对象问题，没有解决对象识别问题（无法判断一个对象的类型）

#### 构造函数

```javaScript
function Person(name, age, job) {
	this.name = name
    this.age = age
    this.job = job
   	this.sayName = function() {
		alert(this.name)
    }
}
var person1 = new Person('nico',29,'Doc')    
var person2 = new Person('fel',20,'ss')    
```

用构造函数创建的对象，可以判断其类型，但是用构造函数中的每个方法都要在实例上重新创建一遍。

基于上段代码运行下面

```javaScript
person1.sayName === person2.sayName //false
```

#### 原型模式

创建的每个函数都有一个prototype属性，这个属性是一个指针，指向的是一个对象，这个对象包含了一类特定类型的所有实例共享的方法和属性。


```javaScript
function Person() {
}
	Person.prototype.name = 'nico'
	Person.prototype.age = 29
	Person.prototype.job = 'Doc'
	Person.prototype.sayName = function() {
		alert(this.name)
	}
var person1 = new Person()    
var person2 = new Person() 

person1.sayName === person2.sayName //true
```
1. 理解原型对象
创建一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，该属性指向函数的原型对象，在默认情况下所有的原型对象都会添加一个constructor属性，也就是构造函数，这个属性包含一个指向prototype所在函数的指针。前面的代码中，Person.prototype.constructor指向Perosn。
创建自定义的构造函数后，其原型对象默认只会取得constructor属性，其他方法，从Object继承而来。
由构造函(数创建的对象中包含一个指针，这个指针指向构造函数的原型对象。也就是__proto__
每当读取某个对象的某个属性时，首先会在对象实例自身搜索此属性并返回属性的值，如果没有，会继续搜索指针指向的原型对象。
存储在原型对象中的值不能够通过对象实例重写。
2. 访问对象属性的几个方法
in操作符会在通过对象访问给定属性时返回true，无论属性存在于实例中还是原型中。
hasOwnProperty,只有在通过对象访问给定属性，并且此属性存在于实例中才会返回true。
for in,遍历整个对象，返回的是整个对象可以访问到的所有可遍历的属性。(\[[Enumeravle]] 的值为true)
Object.keys(),返回对象自身所有并且可以遍历的属性。
Object.getOwnPropertyNames()，返回对象自身所有属性（包括不可遍历的）。
续上面的代码，添加：
```JavaScript
person1.loc = 'xxx-yyy'
person1.id = '00x2'
person1.sayHi = function () {
      alert('hello')
}

for(var porp in person1) {
	console.log(prop) 
}
/* 打印结果，会遍历原型中的属性，但是没有不可遍历的属性
** loc
** id
** sayHi
** name
** age
** job
** sayName
*/ 

console.log(Object.keys(Person.prototype)) 
// ["name", "age", "job", "sayName"] 没有返回不可遍历的属性

console.log(Object.getOwnPropertyNames(Person.prototype))
// ["constructor", "name", "age", "job", "sayName"] 有返回不可遍历的属性
```

#### 构造函数模式和原型模式的结合
构造函数用于定义实例属性，原型模式用于定义方法和共享的属性。
#### 动态原型模式
通过判断某个应该存在的方法是否有效，来决定是否需要初始化原型。
```JavaScript
function Person(name, gae, job) {
	this.name = name
	this.gage = age
	this.job= job

	if(typeOf this.sayNmae != 'function') {
		Person.prototype.sayName = function() {
			alert(this.name)
		}
	}
}
```

#### 寄生构造函数
函数的作用仅仅是封装创建对象的代码,这个模式可以在特殊的情况下用来为对象创建构造函数。假如我们想创建一个具有额外方法的数组，并且不直接修改Array构造函数。
```JavaScript
function SpecialArr() {
	var values = new Array()
	values.push.apply(values, arguments)
	values.toPipedString = function() {
		return this.join('|')
	}
	return values
}
var colors = new SpecilaArr('red', 'blue' ,'green')
alert(colors.toPipedString()) // red|blue|green
```
通过构造函数创建的对象和构造函数的原型属性之间没有关系，构造函数返回的对象与构造函数外部创建的对象并没什么不同。
#### 稳妥构造函数模式
没有公共属性，不引用this的对象。
### 继承
#### 原型链
默认构造函数、原型对象、实例的关系：每个构造函数都有一个原型对象，原型对象包好一个指针指向构造函数，实例存在内部指针指向原型对象。
如果我们让原型对象等于另外一个类型的实例，那么此时的原型对象包好一个指针指向另一个原型对象，相应的，另一个原型中也包含一个指向另外一个构造函数的指针。同样的，这里的另外一个原型也可以是其他类型的实例，那么这种关系就会构成实例与原型的链条，层层递进下去。所有引用类型默认都继承了Object，所以原型链的最终指向了Object的原型。
```JavaScript
function Sup() {
	this.property = true
}
Sup.prototype.getSup = function() {
	return this.property
}

function Sub() {
	this.subproperty = false
}
Sub.prototype = new Sup()
Sub.prototype.getSub = function() {
	return this.subproperty
}

var instance = new Sub()
console.log(instance.getSup()) //true
```
1. 确定原型和实例的关系
instace instanceOf Contructor 判断instance是不是Constructor构造函数的实例。
Contructor.prototype.isProtptypeOf(instance)判断实例instance是不是原型对象Contructor.prototype所派生的实例。
2. 注意定义方法
 - 子类型中有时候需要添加超类型中不存在的方法，或者是修改超类型中的方法。给原型添加方法的代码必须放在替换原型对象语句的后面。
 - 修改超类型的代码或者是修改超类型方法的代码，不能用字面量表达。因为这样会重写原型链。
3. 实例属性会变成原型属性
因为引用类型值的原型属性会被所有实例共享，所以我们将实例属性定义在构造函数中，在通过原型链继承的时候，原型实际上会变成另外一个类型的实例，所以，原先的实例属性就会变成这一时刻的原型属性。

#### 借用构造函数
在子类型中调用超类型的构造函数，可以实现各实例之间不会共享原型的属性，并且可以在子类型构造函数中向超类型构造函数传参。
```JavaScript
function People(w) {
	this.w = w
}
function Stu(name) {
	People.call(this, 'student')
	this.name = name
	this.fre = ['x', 'y']
}

var s1 = new Stu('nico')
s1.fre.push('z')
var s2 = new Stu('rudy')
console.log(s1.fre) // ['x', 'y', 'z']
console.log(s2.fre) // ['x', 'y']

console.log(s1.w) // student
```
缺点：因为构造函数的通病，方法都在构造函数中定义，函数复用是没有实现的，在超类型的原型中定义的方法子类型并不可见。很少单独使用此方法。 
#### 组合继承
组合原型链和构造函数来实现继承。用原型链来实现原型属性和方法的继承，用构造函数实现实例属性的继承。
```JavaScript
function People(w) {
	this.w = w
}
People.prototype.sayW = function() {
	alert(this.w)
}
function Stu(name) {
	People.call(this, 'student')  //第二次调用People()
	this.name = name
	this.frends = ['tina','wang']
}
Stu.prototype = new People()  //第一次调用People()
Stu.prototype.contructor = Stu
Stu.prototype.showFrends = function() {
	console.log('frends: ' + this.frends)
}

var stu1 = new Stu('nico')
stu1.frends.push('tim')
console.log(stu1.frends) //['tina','wang','tim']
stu1.sayW() //student
stu1.showFrends() //['tina','wang','tim']

var stu2 = new Stu('wuwu')
console.log(stu2.frends) //['tina','wang']
stu2.sayW() //student
stu2.showFrends() //['tina','wang']
```
这种模式中两次调用了超类型构造函数，第一次调用超类型构造函数是，Stu.prototype会得到一个属性W；他们是People的实例属性，现在位于Stu的原型中，当调用Stu的构造函数时，会在此调用People构造函数，这一次有在新对象上创建了实例属性w。于是此时的W属性就屏蔽了原型中的同名属性。
#### 寄生组合继承
通过借用构造函数来继承属性，通过原型链来继承方法
```JavaScript
// 原型式继承
function object(o) {
	function F(){}
	F.prototype = o
	return o
}

function inheritPrototype(subType, superType) {
	var prototype = object(superType.prototype) //创建对象
	prototype.constructor = subType // 增强对象
	subtype.prototype = prototype // 指定对象
}
```
这种模式下只调用了一次超类型的构造函数，所以子类型的原型中就不会有多余的属性（组合继承中描述的W属性）