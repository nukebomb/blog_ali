---
title: jQuery源码阅读笔记【1】
tags: 
  - jQuery
  - JavaScript
---
### 写在前面
jQuery是JavaScript的工具库，极大的简化了JavaScript的开发体验。其核心特性：
 - 实现了兼容众多浏览器
 - 易于使用的API，链式调用语法
 - 高效的css选择器
 - 便捷的插件扩展

使用jQuery也算有一段时间，在对jQuery的API有一定利用机会后，更想知道其中背后的逻辑与原理，所以决定通过阅读jQuery的源码来提升JS的功底的同时，学习jQuery的编程思想。

### 两个整体特性
jQuery具有两个整体的显而易见的特性：
- 使用$('XXX')，可以创建一个jQuery对象；
- 链式调用：$('#idname').removeAttr(name).addClass(className).

#### 链式调用
对于链式调用的实现，其实很好理解，就是每次对对象操作完成后返回此对象，也就是每个操作的函数中最后return this,这样就实现了链式调用。
``` JavaScript
var Person  = function (age) {
  this.age = age
}
Person.prototype = {
  addone: function () {
    this.age ++
    return this
  },
  addtwo: function () {
    this.age += 2
    return this
  }
}
var p1 = new Person(21)
p1.addone().addtwo() // p1.age = 24
```
#### 匿名函数自调用
在jQuery的源码是全部包含在一个匿名函数的自调用中的，
``` JavaScript
(funciton (window, undefined) {
  ...

function (selector, context) {
    // The jQuery object is actually just the init constructor 'enhanced'
    // Need init if jQuery is called (just allow error to be thrown if not included)
    return new jQuery.fn.init(selector, context);
}
jQuery.fn = jQuery.prototype = {
}

...

})(window);
```
通过自调用匿名函数，创建了一个函数的作用域，在这个中作用域中的代码和名称不会和外部的变量，函数，产生冲突。这种隔绝，确保了jQuery的代码不会与其他代码产生干扰，并且不会影响到全局变量，
1) 匿名函数的末尾分号
js实际上对代码每一行添加分号是必选的，我个人习惯是不添加的，至于原因后面会考虑再做<font color=orange>一篇总结</font>,但是对于自调用匿名函数来说，如果省略之前或者之后分号，是有可能引起语法错误的：

``` JavaScript
var t = 0
(function() {})()
// TypeError: 0 is not a function
```

因为匿名函数并没有添加分号，所以0的下一括号会被解析成0这个函数的运行，所以会报0 不是一个函数的错误。
2) 添加window对象：

 - window 成为局部变量，在jQuer中访问window不需要回退到顶层作用域;
 - 压缩代码
 
 
 ```
 (function(a,b){})(window)
 ```


3) 传入undefined

除了像传入window一样，可以减少回退，快速访问的作用以外，更重要的原因是因为在浏览器早起版本中，undefined是可以被重新赋值的，所以为了保证undefined的就是undefined，并且不受到外界改变的影响，所以自调用匿名函数设置了参数undefined。

#### 无new创建对象

jQuery并没有通过new显示的实例化，而是直接调用其初始化函数

``` JavaScript
function (selector, context) {
    return new jQuery.fn.init(selector, context);
    jQuery.prototype = {
      ...
      init: function() {
        ...
        return this
      },
      first: function() {
        ...
        return this
      }
      ...
    }
}
```

init的方法就放在了jQuery.prototype原型中，但是此时访问jQuery.first()会报错，是因为此时通过new jQuery.prototype.init()返回的实例实际上市init这个函数的实例，各个实例实例之间的this当然也是毫无关系的，完全隔离的。所以源代码在后面加上了一句 ``` jQuery.prototype.init.prototype = jQuery.prototype```,这样jQuery的原型传递给init的原型，并且是引用传递，不需要担心这个循环引起的性能问题。