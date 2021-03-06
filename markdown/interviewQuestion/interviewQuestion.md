# 前端知识汇总


## 第一部分. js

### 1. js数据类型

- 基本数据类型：Boolean、Number、String、undefined、Null、Symbol (ES6新增，表示独一无二的值)

- 引用数据类型：Object、Array、Function

### 2. 数据类型判断

- typeof ：typeof返回一个表示数据类型的字符串，返回结果包括：Number、Boolean、String、Symbol、Object、undefined、Function等7种数据类型，但不能判断Null、Array等
- instanceof ：instanceof 是用来判断A是否为B的实例，表达式为：A instanceof B，如果A是B的实例，则返回true,否则返回false。instanceof 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性，但它不能检测Null和          undefined
- constructor ：constructor作用和instanceof非常相似。但constructor检测 Object与instanceof不一样，还可以处理基本数据类型的检测。不过函数的 constructor 是不稳定的，这个主要体现在把类的原型进行重写，在重写的过程中很有可能出现把之前    的constructor给覆盖了，这样检测出来的结果就是不准确的。
- Object.prototype.toString.call() ：是最准确最常用的方式。

```javascript

Object.prototype.toString.call();               // [object String]

Object.prototype.toString.call(1);              // [object Number]

Object.prototype.toString.call(true);           // [object Boolean]

Object.prototype.toString.call(undefined);      // [object Undefined]

Object.prototype.toString.call(null);           // [object Null]

Object.prototype.toString.call(new Function()); // [object Function]

Object.prototype.toString.call(new Date());     // [object Date]

Object.prototype.toString.call([]);             // [object Array]

Object.prototype.toString.call(new RegExp());   // [object RegExp]

Object.prototype.toString.call(new Error());    // [object Error]

```

### 3. call, apply, bind的区别，怎么实现call,apply方法?

>在js中，每个函数的原型都指向Function.prototype对象（js基于原型链的继承）。因此，每个函数都会有apply，call，和bind方法，这些方法继承于Function。
>它们的作用是一样的，都是用来改变函数中this的指向。

1. 方法定义：

- apply：调用一个对象的一个方法，用另一个对象替换当前对象。例如：B.apply(A, arguments);即A对象应用B对象的方法。
- call：调用一个对象的一个方法，用另一个对象替换当前对象。例如：B.call(A, args1,args2);即A对象调用B对象的方法。

2. call 与 apply 的相同点：

- 方法的含义是一样的，即方法功能是一样的；
- 第一个参数的作用是一样的；

3. call 与 apply 的不同点：

- 两者传入的列表形式不一样
- call可以传入多个参数；
- apply只能传入两个参数，所以其第二个参数往往是作为数组形式传入

4. bind的传参方式和call一样，只不过它的不同之处是，apply和call方法调用之后会立即执行，而bind方法调用之后会返回一个新的函数，它并不会立即执行，需要我们手动执行。

5. 存在的意义：
- 实现（多重）继承
   
```javascript

Function.prototype.myBind = function(content) {
    if(typeof this !='function'){
        throw Error('not a function')
    }
    let _this = this;
    let args = [...arguments].slice(1) 
    let resFn=function(){
        return _this.apply(this instanceof resFn?this:content,args.concat(...arguments))
    }
    return resFn
};
 
 /**
 * 每个函数都可以调用call方法，来改变当前这个函数执行的this关键字，并且支持传入参数
 */
Function.prototype.myCall=function(context=window){
      context.fn = this;//此处this是指调用myCall的function
      let args=[...arguments].slice(1);
      let result=content.fn(...args)
      //将this指向销毁
      delete context.fn;
      return result;
}
/**
 * apply函数传入的是this指向和参数数组
 */
Function.prototype.myApply = function(context=window) {
    context.fn = this;
    let result;
    if(arguments[1]){
        result=context.fn(...arguments[1])
    }else{
        result=context.fn()
    }
    //将this指向销毁
    delete context.fn;
    return result;
}


```

### 4. this指向问题

>由于 JS 的设计原理: 在函数中，可以引用运行环境中的变量。因此就需要一个机制来让我们可以在函数体内部获取当前的运行环境，这便是this。因此要明白 this 指向，其实就是要搞清楚 函数的运行环境，说人话就是，谁调用了函数。
> this的值是在执行的时候才能确认，定义的时候不能确认。 因为this是执行上下文环境的一部分，而执行上下文需要在代码执行之前确定，而不是定义的时候。

- 对于直接调用 foo 来说，不管 foo 函数被放在了什么地方，this 一定是 window

- 对于 obj.foo() 来说，我们只需要记住，谁调用了函数，谁就是 this，所以在这个场景下 foo 函数中的 this 就是 obj 对象

- 在构造函数模式中，类中(函数体中)出现的this.xxx=xxx中的this是当前类的一个实例

- call、apply和bind：this 是第一个参数

- 箭头函数this指向:箭头函数没有自己的this，看其外层的是否有函数，如果有，外层函数的this就是内部箭头函数的this，如果没有，则this是window。


### 5. 原型 / 构造函数 / 实例

- 原型(prototype): 一个简单的对象，用于实现对象的 属性继承。可以简单的理解成对象的爹。在 Firefox 和 Chrome 中，每个JavaScript对象中都包含一个__proto__ (非标准)的属性指向它爹(该对象的原型)，可obj.__proto__进行访问。


- 构造函数: 可以通过new来 新建一个对象 的函数。


- 实例: 通过构造函数和new创建出来的对象，便是实例。 实例通过__proto__指向原型，通过constructor指向构造函数。


### 6. 原型链

> 概念：原型链是由原型对象组成，每个对象都有 __proto__ 属性，指向了创建该对象的构造函数的原型，__proto__ 将对象连接起来组成了原型链。是一个用来实现继承和共享属性的有限的对象链。

- 属性查找机制: 当查找对象的属性时，如果实例对象自身不存在该属性，则沿着原型链往上一级查找，找到时则输出，不存在时，则继续沿着原型链往上一级查找，直至最顶级的原型对象Object.prototype，如还是没找到，则输出undefined；

- 属性修改机制: 只会修改实例对象本身的属性，如果不存在，则进行添加该属性，如果需要修改原型的属性时，则可以用: b.prototype.x = 2；但是这样会造成所有继承于该对象的实例的属性发生改变。


### 7. 继承（原型链继承）

> 概念：在 JS 中，继承通常指的便是 原型链继承，也就是通过指定原型，并可以通过原型链继承原型上的属性或者方法。原型链是实现继承最原始的模式，即通过prototype属性实现继承。将父类的实例作为子类的原型。常用的有6种继承方式：  

- 原型链继承
- 借用构造函数继承
- 组合继承（组合原型链继承和借用构造函数继承）（常用）
- 原型式继承
- 寄生式继承
- 寄生组合式继承（常用）

### 8. 执行上下文(EC)和执行栈

- 执行上下文: 就是当前 JavaScript 代码被解析和执行时所在环境的抽象概念， JavaScript 中运行任何的代码都是在执行上下文中运行。执行上下文的生命周期包括三个阶段：创建阶段→执行阶段→回收阶段
- 执行栈: JavaScript 引擎创建了执行栈来管理执行上下文。可以把执行栈认为是一个存储函数调用的栈结构，遵循先进后出的原则。

### 9. 作用域与作用域链

- 作用域: 执行上下文中还包含作用域链。作用域其实可理解为该上下文中声明的变量和声明的作用范围。可分为 块级作用域 和 函数作用域( 也可理解为：作用域就是一个独立的地盘，让变量不会外泄、暴露出去。也就是说作用域最大的用处就是隔离变量，不同作   用域下同名变量不会有冲突。)
- 作用域链：作用域链可以理解为一组对象列表，包含 父级和自身的变量对象，因此我们便能通过作用域链访问到父级里声明的变量或者函数。我们知道，我们可以在执行上下文中访问到父级甚至全局的变量，这便是作用域链的功劳。

### 10. 闭包

> 概念：定义在函数内部的函数。里面的函数可以访问外面函数的变量，外面的变量的是这个内部函数的一部分。（其他说法：闭包属于一种特殊的作用域，称为 静态作用域。它的定义可以理解为: 父函数被销毁 的情况下，返回出的子函数的[[scope]]中仍然保留着> 父级的单变量对象和作用域链，因此可以继续访问到父级的变量对象，这样的函数称为闭包。）

- 作用：1.使用闭包可以访问函数中的变量。2.可以使变量长期保存在内存中，生命周期比较长。
- 缺点：闭包不能滥用，否则会导致内存泄露，影响网页的性能。
- 应用场景：1.函数作为参数传递。2.函数作为返回值

```javascript

/**
*JavaScript中的函数会形成了闭包。
*闭包是由函数以及声明该函数的词法环境组合而成的。该环境包含了这个闭包创建时作用域内的任何局部变量。
**/

function foo(){
  var num = 1
  function compute(){
    num++
    return num
  }
  return compute
}

var fn = foo()
fn()

// num 变量和 compute 函数就组成了一个闭包

```

### 11. 浅拷贝与深拷贝

浅拷贝：

a. 概念：浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。修改时原对象也会受到影响。

b. 方法：

- 利用 = 赋值操作符实现浅拷贝。

- 数组的浅拷贝一般使用 slice、concat。

- 数组浅拷贝 - 遍历 。

- 对象浅拷贝 - Object.assign()。

- 对象浅拷贝 - 扩展运算符

深拷贝：

a. 概念：深拷贝就是在拷贝数据的时候，将数据的所有引用结构都拷贝一份。简单的说就是，在内存中存在两个数据结构完全相同又相互独立的数据，将引用型类型进行复制，而不是只复制其引用关系。修改时原对象不再受到任何影响。

b. 方法：

- 利用 JSON 对象中的 parse 和 stringify。

```
JSON.parse(JSON.stringify(obj))深拷贝弊端：
1.时间会变成字符串形式，而不是时间对象。
2.如果obj里面有function,undefined，则序列化的结果会把function,undefined丢失
3.如果obj里面有NaN,Infinity和-Infinity，则序列化的结果会变成null
4.JSON.stringify()只能序列化对象的可枚举的自有属性
5.如果obj里有RegExp、Error对象,则序列化的结果只能得到空对象

```

- 利用递归来实现每一层都重新创建对象并赋值。


[----------具体深拷贝和浅拷贝函数封装,请点击此处](https://blog.csdn.net/weixin_38008863/article/details/87902901)


### 12. require与import的区别

相同点：
- import和require都是被模块化使用

不同点：
- require是CommonJs的语法（AMD规范引入方式），CommonJs的模块是对象；import是es6的一个语法标准（浏览器不支持，本质是使用node中的babel将es6转码为es5再执行，import会被转码为                        require），es6模块不是对象
- require是运行时加载整个模块（即模块中所有方法），生成一个对象，再从对象上读取它的方法（只有运行时才能得到这个对象,不能在编译时做到静态化），理论上可以用在代码的任何地方；import是编译时调用，确定模   块的依赖关系，输入变量（es6模块不是对象，而是通过export命令指定输出代码，再通过import输入，只加载import中导的方法，其他方法不加载），import具有提升效果，会提升到模块的头部（编译时执行）export和    import可以位于模块中的任何位置，但是必须是在模块顶层，如果在其他作用域内，会报错es6这样的设计可以提高编译器效率，但没法实现运行时加载
- require是赋值过程，把require的结果（对象，数字，函数等），默认是export的一个对象，赋给某个变量（复制或浅拷贝）；import是解构过程（需要谁，加载谁）

备注：

- require 支持动态导入，属于同步导入，是值拷贝，导出值变化不会影响导入值。
- import  不支持动态导入，正在提案 (babel 下可支持), 属于 异步 导入, 指向内存地址，导入值会随导出值而变化。

### 13. commonjs与ES6的module的区别

- 两者的模块导入导出语法不同，commonjs是module.exports，exports导出，require导入；ES6则是export导出，import导入。	
- commonjs是运行时加载模块；ES6是在静态编译期间就确定模块的依赖。
- ES6在编译期间会将所有import提升到顶部；commonjs不会提升require。
- commonjs模块是拷贝；es6模块是引用。commonjs导出的是一个值拷贝，会对加载结果进行缓存，一旦内部再修改这个值，则不会同步到外部；ES6是导出的一个引用，内部修改可以同步到外部。
- 两者的循环导入的实现原理不同，commonjs是当模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。所以，输入变量的时候，必须非常小心；ES6 模块是动态引用，如   果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。
- commonjs中顶层的this指向这个模块本身；而ES6中顶层this指向undefined。


### 14. 防抖与节流

> 防抖与节流函数是一种最常用的 高频触发优化方式，能对性能有较大的帮助。

- 防抖 (debounce): 将多次高频操作优化为只在最后一次执行，通常使用的场景是：用户输入，只需再输入完成后做一次输入校验即可。

```javascript

/**
 * 防抖 (debounce)将多次高频操作优化为只在最后一次执行
 * 
 * @param {Function} fn 需要防抖函数
 * @param {Number} wait  需要延迟的毫秒数
 * @param {Boolean} immediate 可选参，设为true，debounce会在wait时间间隔的开始时立即调用这个函数
 * @return {Function}
 * 
 */
 const debounce= (fn, wait, immediate) =>{
    let timer = null

    return function() {
        let args = arguments
        let context = this

        if (immediate && !timer) {
            fn.apply(context, args)
        }

        if (timer) clearTimeout(timer)
        timer = setTimeout(() => {
            fn.apply(context, args)
        }, wait)
    }
}

```

- 节流(throttle): 每隔一段时间后执行一次，也就是降低频率，将高频操作优化成低频操作，通常使用场景: 滚动条事件 或者 resize 事件，通常每隔 100~500 ms执行一次即可。

```javascript

/**
 * 节流(throttle)将高频操作优化成低频操作，每隔 100~500 ms执行一次即可
 * 
 * @param {Function} fn 需要防抖函数
 * @param {Number} wait  需要延迟的毫秒数
 * @param {Boolean} immediate 可选参立即执行，设为true，debounce会在wait时间间隔的开始时立即调用这个函数
 * @return {Function}
 * 
 */
 const throttle =(fn, wait, immediate) =>{
    let timer = null
    let callNow = immediate
    
    return function() {
        let context = this,
            args = arguments

        if (callNow) {
            fn.apply(context, args)
            callNow = false
        }

        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(context, args)
                timer = null
            }, wait)
        }
    }
}

```

### 15. AST

抽象语法树 (Abstract Syntax Tree)，是将代码逐字母解析成 树状对象 的形式。这是语言之间的转换、代码语法检查，代码风格检查，代码格式化，代码高亮，代码错误提示，代码自动补全等等的基础。

### 16. babel编译原理

babel是一个转译器，感觉相对于编译器compiler，叫转译器transpiler更准确，因为它只是把同种语言的高版本规则翻译成低版本规则，而不像编译器那样，输出的是另一种更低级的语言代码。
但是和编译器类似，babel的转译过程也分为三个阶段：parsing、transforming、generating，以ES6代码转译为ES5代码为例，babel转译的具体过程如下：

- babylon 将 ES6/ES7 代码解析成 AST
- babel-traverse 对 AST 进行遍历转译，得到新的 AST
- 新 AST 通过 babel-generator 转换成 ES5

此外，还要注意很重要的一点就是，babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而新标准引入的新的原生对象，部分原生对象新增的原型方法，新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill来解决。

### 17. 函数柯里化

在一个函数中，首先填充几个参数，然后再返回一个新的函数的技术，称为函数的柯里化。通常可用于在不侵入函数的前提下，为函数 预置通用参数，供多次重复调用。

```javascript

const add = function add(x) {
	return function (y) {
		return x + y
	}
}

const add1 = add(1)

add1(2) === 3
add1(20) === 21

```
### 18. ES6/ES7常用属性及方法

声明：

- let: 声明变量，块级作用域、不存在变量提升、暂时性死区、不允许重复声明
- const: 声明常量，只读无法修改

解构赋值：

- class / extend: 类声明与继承
- Set / Map: 新的数据结构

异步解决方案：

- Promise的使用与实现
- generator： yield: 暂停代码 ； next(): 继续执行代码

ES7新增：

- Array.prototype.includes()方法，用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回true，否则返回false
- 求幂运算符（**），它与 Math.pow(a, b)相同

```javascript

//Array.prototype.includes()

['a', 'b', 'c', 'd'].includes('b')     // true

//求幂运算符（**）

3 ** 2            // 9
//效果同：
Math.pow(3, 2)   // 9

```

### 19. Array 常见操作方法

- map: 遍历数组，返回回调返回值组成的新数组

- forEach: 无法break，可以用try/catch中throw new Error来停止

- filter: 过滤

- some: 有一项返回true，则整体为true

- every: 有一项返回false，则整体为false

- join: 通过指定连接符生成字符串

- push / pop: 末尾推入和弹出，改变原数组， 返回推入/弹出项

- unshift / shift: 头部推入和弹出，改变原数组，返回操作项

- sort(fn) / reverse: 排序与反转，改变原数组

- concat: 连接数组，不影响原数组， 浅拷贝

- slice(start, end): 返回截断后的新数组，不改变原数组

- splice(start, number, value...): 返回删除元素组成的数组，value 为插入项，改变原数组

- indexOf / lastIndexOf(value, fromIndex): 查找数组项，返回对应的下标

- reduce / reduceRight(fn(prev, cur)， defaultPrev): 两两执行，prev 为上次化简函数的return值，cur 为当前值(从第二项开始)

- flat： 数组拆解， flat: [1,[2,3]] --> [1, 2, 3]

### 20. Object 常见操作方法

- Object.keys(obj): 获取对象的可遍历属性(键)

- Object.values(obj): 获取对象的可遍历属性值(值)

- Object.entries(obj): 获取对象的可遍历键值对

- Object.assign(targetObject,...object): 合并对象可遍历属性

- Object.is(value1,value2): 判断两个值是否是相同的值

详细参考，请点击 [js对象方法大全](https://blog.csdn.net/qq_26562641/article/details/88575516?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1)


## 第二部分. 浏览器与服务端

### 1. http协议

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。属于应用层的 面向对象的协议，由于其简捷、快速的方式，适用于分布式超媒体信息系统。


### 2. HTTPS的工作原理

非对称加密与对称加密双剑合璧，使用非对称加密算法传递用于对称加密算法的密钥，然后使用对称加密算法进行信息传递。这样既安全又高效

### 3. 三次握手

  在TCP/IP协议中,TCP协议提供可靠的连接服务,采用三次握手建立一个连接：

- 第一次握手：建立连接时,客户端发送syn包(syn=j)到服务器,并进入SYN_SEND状态,等待服务器确认；SYN：同步序列编号(Synchronize Sequence Numbers)
- 第二次握手：服务器收到syn包,必须确认客户的SYN（ack=j+1）,同时自己也发送一个SYN包（syn=k）,即SYN+ACK包,此时服务器进入SYN_RECV状态； 
- 第三次握手：客户端收到服务器的SYN＋ACK包,向服务器发送确认包ACK(ack=k+1),此包发送完毕,客户端和服务器进入ESTABLISHED状态,完成三次握手.完成三次握手,客户端与服务器开始传送数据


### 4. 在浏览器地址栏键入URL，按下回车之后会经历以下流程

- DNS 解析（浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址）
- 建立TCP连接 (三次握手）
- 发送请求，分析 url，设置请求报文(头，主体)
- 服务器返回请求的文件 (html)
- 释放 TCP连接（四次挥手）
- 浏览器渲染 html 文本并显示内容 

### 5. get和post请求的区别

- get : 缓存、不安全、请求长度受限不能超过4k、会被历史保存记录
- post: 不会缓存、安全、请求数据大小不受限、更多编码类型


### 6. 常见状态码

- 1xx: 接受，继续处理
- 200: 成功，并返回数据
- 201: 已创建
- 202: 已接受
- 203: 成为，但未授权
- 204: 成功，无内容
- 205: 成功，重置内容
- 206: 成功，部分内容
- 301: 永久移动，重定向
- 302: 临时移动，可使用原有URI
- 304: 资源未修改，可使用缓存
- 305: 需代理访问
- 400: 请求语法错误
- 401: 要求身份认证
- 403: 拒绝请求
- 404: 资源不存在
- 500: 服务器错误

### 7. Websocket

>
> 概念：WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。它是一个 持久化的协议， 基于 http ， 服务端可以 主动 push
>

```javascript

new WebSocket(url)

ws.onerror = fn

ws.onclose = fn

ws.onopen = fn

ws.onmessage = fn

ws.send()

```
### 8. Web Worker

web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。
现代浏览器为JavaScript创造的 多线程环境。可以新建并将部分任务分配到worker线程并行运行，两个线程可 独立运行，互不干扰，可通过自带的 消息机制 相互通信。

```javascript
/*基本用法*/
// 创建 worker
const worker = new Worker('work.js');

// 向主进程推送消息
worker.postMessage('Hello World');

// 监听主进程来的消息
worker.onmessage = function (event) {
  console.log('Received message ' + event.data);
}

```

限制：

- 同源限制
- 无法使用 document / window / alert / confirm
- 无法加载本地资源

### 9. Promise

>概念： Promise 是异步编程的一种解决方案，比传统的异步解决方案【回调函数】和【事件】更合理、更强大。现已被 ES6 纳入进规范中。

Promise 的常用 API 如下：

- Promise.resolve(value) : 类方法，该方法返回一个以 value 值解析后的 Promise 对象
- Promise.reject : 类方法，且与 resolve 唯一的不同是，返回的 promise 对象的状态为 rejected。
- Promise.prototype.then : 实例方法，为 Promise 注册回调函数，函数形式：fn(vlaue){}，value 是上一个任务的返回结果，then 中的函数一定要 return 一个结果或者一个新的 Promise 对象，才可以让之后的then 回调接收。
- Promise.prototype.catch : 实例方法，捕获异常，函数形式：fn(err){}, err 是 catch 注册 之前的回调抛出的异常信息。
- Promise.race ：类方法，多个 Promise 任务同时执行，返回最先执行结束的 Promise 任务的结果，不管这个 Promise 结果是成功还是失败。
- Promise.all : 类方法，多个 Promise 任务同时执行，如果全部成功执行，则以数组的方式返回所有 Promise 任务的执行结果。 如果有一个 Promise 任务 rejected，则只返回 rejected 任务的结果。

```javascript

//基本用法

const promise = new Promise((resolve, reject) => {
    try{
       resolve('success')
    }catch(err){
        reject('error')
    }
  
})

promise
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })


```

### 10. Ajax的原理

简单来说就是 通过XmlHttpRequest对象向服务器发异步请求，从服务器获得数据，然后用 javascript 来操作DOM更新页面的技术。

```javascript

//get请求方式
function get(url, fn){
        var xhr=new XMLHttpRequest();
        xhr.open('GET', url, false);
        xhr.onreadystatechange=function(){
            if(xhr.readyState === 4){
                if(xhr.status === 200){
                    fn.call(xhr.responseText);
                }
            }
        }
        xhr.send();
    }

//post请求方式
function post(url, data, fn){
        var xhr=new XMLHttpRequest();
        xhr.open('POST', url, false);
        // 添加http头，发送信息至服务器时内容编码类型
        xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
        xhr.onreadystatechange=function(){
            if (xhr.readyState === 4){
                if (xhr.status === 200){
                    fn.call(xhr.responseText);
                }
            }
        }
        xhr.send(data);
    }

```

### 11. 跨域

>
>概念：跨域是指浏览器不能执行其他网站的脚本。它是由浏览器的同源策略（域名、协议、端口均为相同）造成的，是浏览器对JavaScript实施的安全限制。
>

- JSONP: 利用<script>标签不受跨域限制的特点，缺点是只能支持 get 请求。
- 设置 CORS: Access-Control-Allow-Origin：*
- postMessage

```javascript

// JSONP方式
function jsonp(url, jsonpCallback, success) {
  const script = document.createElement('script')
  script.src = url
  script.async = true
  script.type = 'text/javascript'
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}


```

### 12. 浏览器存储Cookie、 LocalStorage 与 SessionStorage

Cookie：

- 一般由服务器生成，可设置失效时间。如果在浏览器端生成Cookie，默认是关闭浏览器后失效
- 大小限制为4KB左右
- 每次都会携带在HTTP头中，如果使用cookie保存过多数据会带来性能问题
- 需要程序员自己封装，源生的Cookie接口不友好

LocalStorage：

- 除非被清除，否则永久保存
- 一般为5MB
- 仅在客户端（即浏览器）中保存，不参与和服务器的通信
- 原生接口可以接受，亦可再次封装来对Object和Array有更好的支持

SessionStorage：

- 仅在当前会话下有效，关闭页面或浏览器后被清除
- 一般为5MB
- 仅在客户端（即浏览器）中保存，不参与和服务器的通信
- 原生接口可以接受，亦可再次封装来对Object和Array有更好的支持

### 13. 安全

- XSS攻击: 注入恶意代码。 （解决方法：1.cookie 设置 httpOnly; 2.转义页面上的输入内容和输出内容）
- CSRF: 跨站点请求伪造。（解决办法：1.get 不修改数据；2.不被第三方网站访问到用户的 cookie； 3. 设置白名单，不被第三方网站请求；4.请求加校验）

### 14. 内存泄露

- 意外的全局变量: 无法被回收
- 定时器: 未被正确关闭，导致所引用的外部变量无法被释放
- 事件监听: 没有正确销毁 (低版本浏览器可能出现)
- 闭包: 会导致父级中的变量无法被释放
- dom 引用: dom 元素被删除时，内存中的引用未被正确清空

### 15.单页面应用（SPA）较多页面应用（MPA）的区别及优缺点

单页面应用（SPA）：
 通俗一点说就是指只有一个主页面的应用，浏览器一开始要加载所有必须的 html, js, css。所有的页面内容都包含在这个所谓的主页面中。但在写的时候，还是会分开写（页面片段），然后在交互的时候由路由程序动态载入，单页面的页面跳转，仅刷新局部资源。多应用于pc端。

多页面应用（MPA）：
  就是指一个应用中有多个页面，页面跳转时是整页刷新。

单页面的优点：
- 用户体验好，快，内容的改变不需要重新加载整个页面，基于这一点spa对服务器压力较小。 
- 前后端分离。 
- 页面效果会比较炫酷（比如切换页面内容时的专场动画）。 

单页面缺点：
- 不利于seo。 
- 导航不可用，如果一定要导航需要自行实现前进、后退。（由于是单页面不能用浏览器的前进后退功能，所以需要自己建立堆栈管理）。 
- 初次加载时耗时多。 
- 页面复杂度提高很多。

## 第三部分. Vue

### 1. 概念

Vue是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 2. 特点

- 单页面应用
- 双向数据绑定：视图改变 数据自动更新；数据更新 视图自动改变
- 渐进式：vue vue-router路由 vuex axios
- 框架：自己写的代码被框架调用（库：自己调用库的代码）
- 声明式

### 3. Vue2.x响应式数据原理(数据劫持)

Vue在初始化数据时，会对data进行遍历，并使用Object.defineProperty把这些属性转为getter/setter。
每个组件实例都对应一个watcher 实例，当页面使用对应属性时，首先会用getter进行依赖收集(收集当前组件的watcher)如果属性发生变化setter 触发时会通知相关依赖进行更新操作(发布订阅)。

> 注意：
>1. Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也就是 Vue 不支持 IE8 以及更低版本浏览器的原因。
>2. 由于 JavaScript 的限制，Vue 不能检测数组和对象的变化。

### 4. Vue3.x响应式数据原理

Vue3.x改用Proxy替代Object.defineProperty。因为Proxy可以直接监听对象和数组的变化，并且有多达13种拦截方法。并且作为新标准将受到浏览器厂商重点持续的性能优化。

### 5. 生命周期

- beforeCreate：即将创建。此阶段为实例初始化之后，此时的数据观察和事件机制都未形成
- created：创建完毕。在这个阶段vue实例已经创建，我们在同样打印一下data和DOM元素。
- beforemount：即将挂载。在上个阶段我们知道DOM还没生成，相关属性还是undefined，那么此阶段为即将挂载。
- mounted:渲染完毕。mounted是平时我们使用最多的函数了，一般我们的异步请求都写在这里。在这个阶段，数据和DOM都已被渲染出来。
- beforeUpdate:即将更新渲染。vue遵循数据驱动DOM的原则，当我们修改vue实例的data时，vue会自动帮我们更新视图。
- updated:更新渲染后。为了不使看到同-函数在不能阶段的效果，我注释掉beforeUpdate函数，添加update函数并绑定了刚才的click事件
- beforeDestroy:销毁之前。到上一步vue已经成功的通过数据驱动DOM更新，当我们不在需要vue操纵DOM时，就需要销毁Vue，也就是清除vue实例与DOM的关联，调用destroy方法可以销毁当前组件。在销毁前，会触发beforeDestroy钩   子函数。
- destroyed:销毁之后。在销毁后，会触发destroyed钩子函数。 

### 6. Vue指令

- v-model ：放在表单元素input、textarea、select>option上的，实现双向数据绑定
- v-text ： 展示对应的文本
- v-once ： 对应的标签只渲染一次
- v-show ： 是否能显示
- v-html : 把值中的标签渲染出来
- v-for : 循环显示元素
- v-bind : 用于绑定行内属性 简写成:
- v-if : 控制是否渲染该元素
- v-cloak : 需要配合css使用：解决小胡子显示问题
- v-pre : 跳过有这个指令的标签及其子元素的编译，按照原生代码编译


### 7. 事件修饰符

- .self只点元素本身时才触发事件
- .stop阻止冒泡事件
- .prevent阻止默认事件
- .once对应函数只触发一次
- .capture在捕获阶段触发二级绑定事件
- .passive优先执行默认事件（滚动行为）

### 8. 表单修饰符

- .number转化为数字，类似parse转化
- .trim去字符串前后空格

### 9. directives自定义指令

```javascript

// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})

// 如果想注册局部指令，组件中也接受一个 directives 的选项
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus()
    }
  }
}

//然后你可以在模板中任何元素上使用新的 v-focus 属性，如下：

<input v-focus>

```
 一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

 - bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。

 - inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。

 - update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。

 - componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。

 - unbind：只调用一次，指令与元素解绑时调用。

### 10. vue-router路由传参

提供了两种传参方式:

- query传参（问号传参）： 路由映射表不用改动 :to={path:'',query:{}}或者:to={name:'',query:{}}

- params传参（路径传参）： 在映射表中添加  /:变量  的形式； :to={name:'',params:{变量:''}}}



### 11. 接口请求一般放在哪个生命周期中？

接口请求一般放在mounted中，但需要注意的是服务端渲染时不支持mounted，需要放到created中。

### 12. Computed和Watch区别

- Computed本质是一个具备缓存的watcher，依赖的属性发生变化就会更新视图。
  适用于计算比较消耗性能的计算场景。当表达式过于复杂时，在模板中放入过多逻辑会让模板难以维护，可以将复杂的逻辑放入计算属性中处理。

- Watch没有缓存性，更多的是观察的作用，可以监听某些数据执行回调。当我们需要深度监听对象中的属性时，可以打开deep：true选项，这样便会对对象中的每一项进行监听。这样会带来性能问题，优化的话可以使用字符串形式监听，如果没有写到组件中，不要忘记使用unWatch手动注销哦。

### 13. v-if和v-show的区别

当条件不成立时，v-if不会渲染DOM元素，v-show操作的是样式(display)，切换当前DOM的显示和隐藏。

### 14. 组件中的data为什么是一个函数？

一个组件被复用多次的话，也就会创建多个实例。本质上，这些实例用的都是同一个构造函数。如果data是对象的话，对象属于引用类型，会影响到所有的实例。所以为了保证组件不同的实例之间data不冲突，data必须是一个函数。

### 15. Vue模版编译原理

简单说，Vue的编译过程就是将template转化为render函数的过程。会经历以下阶段：

- 生成AST树（解析模版，生成AST语法树(一种用JavaScript对象的形式来描述整个模板)）

- 优化

- codegen

### 16. 路由的两种实现原理

- Hash模式: window对象提供了onhashchange事件来监听hash值的改变,一旦url中的hash值发生改变,便会触发该事件。

- History 模式: popstate监听历史栈信息变化,变化时重新渲染; 使用pushState方法实现添加功能; 使用replaceState实现替换功能


### 17. keep-alive作用

keep-alive可以实现组件缓存，当组件切换时不会对当前组件进行卸载。
常用的两个属性include/exclude，允许组件有条件的进行缓存。
两个生命周期activated/deactivated，用来得知当前组件是否处于活跃状态。
keep-alive的中还运用了LRU(Least Recently Used)算法。

### 18. Vue2.x组件通信有哪些方式

- 父子组件通信: props 、$on($off注销)、$emit、$refs 获取实例的方式调用子组件的属性或者方法、$parent 子组件直接调用父组件方法、（Provide、inject 官方不推荐使用，但是写组件库时很常用）

- 兄弟组件通信: EventBus 、Vuex

- 跨级组件通信：Vuex、$attrs、$listeners、Provide、（Provide、inject 官方不推荐使用，但是写组件库时很常用）

### 19. SSR

> 概念：SSR也就是服务端渲染，也就是将Vue在客户端把标签渲染成HTML的工作放在服务端完成，然后再把html直接返回给客户端。

优点：

- SSR有着更好的SEO
- 首屏加载速度更快

缺点：

- 开发条件会受到限制
- 服务器端渲染只支持beforeCreate和created两个钩子，当我们需要一些外部扩展库时需要特殊处理，服务端渲染应用程序也需要处于Node.js的运行环境。
- 服务器会有更大的负载需求。

### 20. 性能优化

- 编码阶段
>1. 尽量减少data中的数据，data中的数据都会增加getter和setter，会收集对应的watcher
>2. v-if和v-for不能连用
>3. 如果需要使用v-for给每项元素绑定事件时使用事件代理
>4. SPA 页面采用keep-alive缓存组件
>5. 在更多的情况下，使用v-if替代v-show
>6. key保证唯一
>7. 使用路由懒加载、异步组件
>8. 防抖、节流
>9. 第三方模块按需导入
>10. 长列表滚动到可视区域动态加载 
>11. 图片懒加载
- SEO优化
>1. 预渲染
>2. 服务端渲染SSR

- 打包优化
>1. 压缩代码
>2. Tree Shaking/Scope Hoisting
>3. 使用cdn加载第三方模块
>4. 多线程打包happypack
>5. splitChunks抽离公共文件
>6. sourceMap优化
- 用户体验
>1. 骨架屏
>2. PWA
>3. 客户端缓存、服务端缓存


### 21. Vuex

> 概念: Vuex 是一个专为 Vue应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

结构:
- State ---单一状态

- Getter --- Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

- Mutation --- 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数

- Action --- Action 提交的是 mutation，而不是直接变更状态， Action 可以包含任意异步操作。

- Module --- Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割。


## 第四部分. React

### 1. 概念

React 是一个声明式，高效且灵活的用于构建用户界面的 JavaScript 库。使用 React 可以将一些简短、独立的代码片段组合成复杂的 UI 界面，这些代码片段被称作“组件”。

### 2. 生命周期

- componentWillMount: 在渲染前调用,在客户端也在服务端。

- componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

- componentWillReceiveProps: 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。

- shouldComponentUpdate: 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。 
  可以在你确认不需要更新组件时使用。

- componentWillUpdate:在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。

- componentDidUpdate: 在组件完成更新后立即调用。在初始化时不会被调用。

- componentWillUnmount:在组件从 DOM 中移除之前立刻被调用。


在新版本中，React 官方对生命周期有了新的 变动建议:

- 使用 getDerivedStateFromProps 替换 componentWillMount；
- 使用 getSnapshotBeforeUpdate替换 componentWillUpdate；
- 避免使用 componentWillReceiveProps；

新版的建议生命周期如下:

```javascript

class Component extends React.Component {
  // 替换 `componentWillReceiveProps` ，
  // 初始化和 update 时被调用
  // 静态函数，无法使用 this
  static getDerivedStateFromProps(nextProps, prevState) {}
  
  // 判断是否需要更新组件
  // 可以用于组件性能优化
  shouldComponentUpdate(nextProps, nextState) {}
  
  // 组件被挂载后触发
  componentDidMount() {}
  
  // 替换 componentWillUpdate
  // 可以在更新之前获取最新 dom 数据
  getSnapshotBeforeUpdate() {}
  
  // 组件更新后调用
  componentDidUpdate() {}
  
  // 组件即将销毁
  componentWillUnmount() {}
  
  // 组件已销毁
  componentDidUnMount() {}
}

```

使用建议:

- 在constructor初始化 state。

- 在componentDidMount中进行事件监听，并在componentWillUnmount中解绑事件。

- 在componentDidMount中进行数据的请求，而不是在componentWillMount。

- 需要根据 props 更新 state 时，使用getDerivedStateFromProps(nextProps, prevState)。

- 可以在componentDidUpdate监听 props 或者 state 的变化。

- 在componentDidUpdate使用setState时，必须加条件，否则将进入死循环。

- getSnapshotBeforeUpdate(prevProps, prevState)可以在更新之前获取最新的渲染数据，它的调用是在 render 之后， update 之前。

- shouldComponentUpdate: 默认每次调用setState，一定会最终走到 diff 阶段，但可以通过shouldComponentUpdate的生命钩子返回false来直接阻止后面的逻辑执行，通常是用于做条件渲染，优化渲染的性能。



### 3. React路由传参

```javascript

import queryString from 'query-string'
//方式一
this.props.history.push({
      pathname: '/course/platformCourse',
      search:   queryString.stringify({
         id: '0212',
         name:'你好'   
      })
    })

//方式二
this.props.history.push(`/cart?cartType=${cartType}`)

//获取参数并解析
queryString.parse(this.props.location.search)

```

### 4. 直接输出HTML方法

```javascript

<div  dangerouslySetInnerHTML={{ __html: item.info}}></div>

```

### 5. HOC(高阶组件)

> 概念： 高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。高阶组件（ HOC）是 React中的高级技术，用来重用组件逻辑。但高阶组件本身并不是 ReactAPI。它只是一种模式，这种模式是由 React自身的组合性质必然产生的。

可以实现的功能：

- 组合渲染
- 条件渲染
- 操作 props
- 获取 refs
- 状态管理
- 操作 state
- 渲染劫持


HOC在业务中的实际应用场景：

- 权限控制，通过抽象逻辑，统一对页面进行权限判断，按不同的条件进行页面渲染。

```javascript

function withAdminAuth(WrappedComponent) {
    return class extends React.Component {
		constructor(props){
			super(props)
			this.state = {
		    	isAdmin: false,
			}
		} 
		async componentWillMount() {
		    const currentRole = await getCurrentUserRole();
		    this.setState({
		        isAdmin: currentRole === 'Admin',
		    });
		}
		render() {
		    if (this.state.isAdmin) {
		        return <Comp {...this.props} />;
		    } else {
		        return (<div>您没有权限查看该页面，请联系管理员！</div>);
		    }
		}
    };
}

```

- 性能监控，包裹组件的生命周期，进行统一埋点。

```javascript

function withTiming(Comp) {
    return class extends Comp {
        constructor(props) {
            super(props);
            this.start = Date.now();
            this.end = 0;
        }
        componentDidMount() {
            super.componentDidMount && super.componentDidMount();
            this.end = Date.now();
            console.log(`${WrappedComponent.name} 组件渲染时间为 ${this.end - this.start} ms`);
        }
        render() {
            return super.render();
        }
    };
}

```

- 代码复用，可以将重复的逻辑进行抽象。

- 日志打点

- 双向绑定

- 表单校验

使用注意：

- 纯函数: 增强函数应为纯函数，避免侵入修改元组件。

- 避免用法污染: 理想状态下，应透传元组件的无关参数与事件，尽量保证用法不变。

- 命名空间: 为 HOC 增加特异性的组件名称，这样能便于开发调试和查找问题。

- 引用传递: 如果需要传递元组件的 refs 引用，可以使用React.forwardRef。

- 静态方法: 元组件上的静态方法并无法被自动传出，会导致业务层无法调用；解决:函数导出和静态方法赋值。


### 6. Redux

>概念： Redux 是一个 数据管理中心，可以把它理解为一个全局的 data store 实例。它通过一定的使用规则和限制，保证着数据的健壮性、可追溯和可预测性。它与 React 无关，可以独立运行于任何 JavaScript 环境中，从而也为同构应用提供了更好的数据同步通道。

核心理念：

- 单一数据源: 整个应用只有唯一的状态树，也就是所有 state 最终维护在一个根级 Store 中。

- 状态只读: 为了保证状态的可控性，最好的方式就是监控状态的变化。那这里就两个必要条件（Redux Store 中的数据无法被直接修改； 严格控制修改的执行）。

- 纯函数: 规定只能通过一个纯函数 (Reducer) 来描述修改。

 实现方式：

- Store: 全局 Store 单例， 每个 Redux 应用下只有一个 store， 它具有以下方法供使用: 
> - getState: 获取 state；
> - dispatch: 触发 action, 更新 state；
> - subscribe: 订阅数据变更，注册监听器；

- Action: 它作为一个行为载体，用于映射相应的 Reducer，并且它可以成为数据的载体，将数据从应用传递至 store 中，是 store 唯一的数据源。

- Reducer: 用于描述如何修改数据的纯函数，Action 属于行为名称，而 Reducer 便是修改行为的实质；

```javascript

/*store*/
// 创建
const store = createStore(Reducer, initStore)





/* Action*/

const action = {
	type: 'ADD_LIST',
	item: 'list-item-1',
}

// 使用：
store.dispatch(action)

// 通常为了便于调用，会有一个 Action 创建函数 (action creater)
funtion addList(item) {
	return const action = {
		type: 'ADD_LIST',
		item,
	}
}

// 调用就会变成:
dispatch(addList('list-item-1'))





/*Reducer*/
// @param {state}: 旧数据
// @param {action}: Action 对象
// @returns {any}: 新数据
const initList = []
function ListReducer(state = initList, action) {
	switch (action.type) {
		case 'ADD_LIST':
			return state.concat([action.item])
			break
		defalut:
			return state
	}
}

```
注意:
- 遵守数据不可变，不要去直接修改 state，而是返回出一个 新对象，可以使用 assign / copy / extend / 解构 等方式创建新对象；
- 默认情况下需要 返回原数据，避免数据被清空；
- 最好设置 初始值，便于应用的初始化及数据稳定；


结合React使用 React-Redux：

- Provider: 将 store 通过 context 传入组件中。

- connect: 一个高阶组件，可以方便在 React 组件中使用 Redux。

- 将store通过mapStateToProps进行筛选后使用props注入组件。

- 根据mapDispatchToProps创建方法，当组件调用时使用dispatch触发对应的action

- Reducer 的拆分与重构：随着项目越大，如果将所有状态的 reducer 全部写在一个函数中，将会 难以维护；可以将 reducer 进行拆分，也就是 函数分解，最终再使用combineReducers()进行重构合并。

- 异步 Action: 由于 Reducer 是一个严格的纯函数，因此无法在 Reducer 中进行数据的请求，需要先获取数据，再dispatch(Action)即可，下面是三种不同的异步实现:

```
redex-thunk
redux-saga
redux-observable

```





## 第五部分. Vue和React对比

### 1. 为什么说Vue 的响应式更新比 React 快？

- Vue (响应式+依赖收集)对于响应式属性的更新，只会精确更新依赖收集的当前组件，个组件都有自己的渲染 watcher，而不会递归的去更新子组件。
- React 在类似的场景下是自顶向下的进行递归更新的，也就是说，React 中假如 ChildComponent 里还有十层嵌套子元素，那么所有层次都会递归的重新render（在不进行手动优化的情况下），这是性能上的灾难。（因此，React 创造了Fiber，创造了异步渲染，其实本质上是弥补被自己搞砸了的性能）。他们能用收集依赖的这套体系吗？不能，因为他们遵从Immutable的设计思想，永远不在原对象上修改属性，那么基于 Object.defineProperty 或 Proxy 的响应式依赖收集机制就无从下手了（你永远返回一个新的对象，我哪知道你修改了旧对象的哪部分？）同时，由于没有响应式的收集依赖，React 只能递归的把所有子组件都重新 render一遍（除了memo和shouldComponentUpdate这些优化手段），然后再通过 diff算法 决定要更新哪部分的视图，这个递归的过程叫做 reconciler，听起来很酷，但是性能很灾难。

### 2.React和Vue的虚拟DOM区别

相同点：
- react和vue的虚拟dom都是一样的， 都是用JS对象来模拟真实DOM，然后用虚拟DOM的diff来最小化更新真实DOM。虽然两者对于dom的更新策略不太一样， react采用自顶向下的全量diff，vue是局部订阅的模式。 但是这其实和虚拟dom并无关系

不同点：
- vue会跟踪每一个组件的依赖关系, 不需要重新渲染整个组件树​；而对于React而言,每当应用的状态被改变时,全部组件都会重新渲染,所以react中会需要shouldComponentUpdate这个生命周期函数方法来进行控制。


### 3. 高阶组件(HOC)和Mixin的异同点

作用：

- Mixin和 HOC都可以用来解决 React的代码复用问题

Mixin:

- Mixin 可能会相互依赖，相互耦合，不利于代码维护

- 不同的 Mixin中的方法可能会相互冲突

- Mixin非常多时，组件是可以感知到的，甚至还要为其做相关处理，这样会给代码造成滚雪球式的复杂性

HOC:

- 高阶组件就是一个没有副作用的纯函数，各个高阶组件不会互相依赖耦合

- 高阶组件也有可能造成冲突，但我们可以在遵守约定的情况下避免这些行为

- 高阶组件并不关心数据使用的方式和原因，而被包裹的组件也不关心数据来自何处。高阶组件的增加不会为原组件增加负担

### 4. Hook有哪些优势

- 减少状态逻辑复用的风险
> Hook和 Mixin在用法上有一定的相似之处，但是 Mixin引入的逻辑和状态是可以相互覆盖的，而多个 Hook之间互不影响，这让我们不需要在把一部分精力放在防止避免逻辑复用的冲突上。在不遵守约定的情况下使用 HOC也有可能带来一定冲突，比如 props覆盖等> 等，使用 Hook则可以避免这些问题。

- 避免地狱式嵌套
> 大量使用 HOC的情况下让我们的代码变得嵌套层级非常深，使用 Hook，我们可以实现扁平式的状态逻辑复用，而避免了大量的组件嵌套。

- 让组件更容易理解
> 在使用 class组件构建我们的程序时，他们各自拥有自己的状态，业务逻辑的复杂使这些组件变得越来越庞大，各个生命周期中会调用越来越多的逻辑，越来越难以维护。使用 Hook，可以让你更大限度的将公用逻辑抽离，将一个组件分割成更小的函数，而不是强制> 基于生命周期方法进行分割。

- 使用函数代替class
> 相比函数，编写一个 class可能需要掌握更多的知识，需要注意的点也越多，比如 this指向、绑定事件等等。另外，计算机理解一个 class比理解一个函数更快。Hooks让你可以在 classes之外使用更多 React的新特性。


## 第六部分. Webpack

### 1. 什么是webpack，与Grunt和Gulp有啥不同

概念：

- Webpack是一个模块打包工具，在webpack里面一切皆模块通过loader转换文件，通过plugin注入钩子，最后输出有多个模块组合成的文件。
- Gulp/Grunt是一种能够优化前端的开发流程的构建工具

相同点：

- 三者都是前端构建工具，Grunt和Gulp在早期比较流行，现在webpack相对来说比较主流，不过一些轻量化的任务还是会用gulp来处理，比如单独打包CSS文件等。

不同点：

- Grunt和Gulp是基于任务和流（Task、Stream）的。类似jQuery，找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程。
- Webpack是基于入口的。Webpack会自动地递归解析入口所需要加载的所有资源文件，然后用不同的Loader来处理不同的文件，用Plugin来扩展webpack功能。　

### 2. Webpack的优缺点

优点：

- 专注于处理模块化的项目，能做到开箱即用，一步到位
- 可通过plugin扩展，方便、灵活　　
- 社区庞大活跃，经常引入新特性
- 良好的开发体验

缺点：

- 只能用于采用模块化开发的项目

### 3. webpack构建过程　

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- 1.初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数

- 2.开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译

- 3.确定入口：根据配置中的 entry 找出所有的入口文件

- 4.编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理

- 5.完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系

- 6.输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会

- 7.输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

### 4. 什么是entry,output

- entry入口，告诉webpack要使用哪个模块作为构建项目的起点，默认为./src/index.js

- output出口，告诉webpack在哪里输出它打包好的代码以及如何命名，默认为./dist

### 5. bundle、chunk、module是什么

- bundle: 是由webpack打包出来的文件

- chunk: 代码块，一个chunk由多个模块组合而成，用于代码的合并和分割

- module: 是开发中的单个模块，一个模块对应一个文件，webpack会从配置的entry中递归开始找出所有依赖的模块

### 6. 什么是loader, 什么是plugin, 两者的不同

概念：

- loader：模块转换器。用于把模块原内容按照需求转换成新内容。通过使用不同的Loader，Webpack可以要把不同的文件都转成JS文件,比如CSS、ES6/7、JSX等

- plugin：扩展插件。在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情，一个插件是含有apply方法的一个对象，通过这个方法可以参与到整个webpack打包的各个流程

不同：

- loader是使webpack拥有加载和解析非js文件的能力

- plugin 可以扩展webpack的功能，使得webpack更加灵活。可以在构建的过程中通过Webpack的api改变输出的结果

> loader是用来对模块的源代码进行转换,而plugin目的在于解决 loader 无法实现的其他事，因为plugin可以在任何阶段调用,能够跨Loader进一步加工Loader的输出

### 7. 常见的plugin和常见的loader有哪些，有什么作用

plugin:

- html-webpack-plugin 为html文件中引入的外部资源，可以生成创建html入口文件
- mini-css-extract-plugin 分离css文件
- clean-webpack-plugin 删除打包文件
- HotModuleReplacementPlugin 热更新应用
- copy-webpack-plugin 拷贝静态文件
- define-plugin：定义环境变量
- commons-chunk-plugin：提取公共代码
- terser-webpack-plugin 通过TerserPlugin压缩ES6代码

loader:

- file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件
- url-loader：和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去
- source-map-loader：加载额外的 Source Map 文件，以方便断点调试
- image-loader：加载并且压缩图片文件
- babel-loader：把 ES6 转换成 ES5
- css-loader：加载 CSS，支持模块化、压缩、文件导入等特性
- style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS。
- eslint-loader：通过 ESLint 检查 JavaScript 代码

### 8. 什么是模块热更新

Webpack的热更新又称热替换（Hot Module Replacement），缩写为HMR。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。devServer中通过hot属性可以空时模块的热替换。

```javascript

//通过配置文件
const webpack = require('webpack');
const path = require('path');
let env = process.env.NODE_ENV == "development" ? "development" : "production";
const config = {
    mode: env,
    devServer: {
        hot:true
    }
}
    plugins: [
        new webpack.HotModuleReplacementPlugin(), //热加载插件
    ],
module.exports = config;

```

### 9. webpack-dev-server和http服务器如nginx有什么区别

webpack-dev-server使用内存来存储webpack开发环境下的打包文件。并且可以使用模块热更新，他比传统的http服务对开发更加简单高效。

### 10. 在Webpack中如何做到长缓存优化

浏览器在用户访问页面的时候，为了加快加载速度会对用户访问的静态资源进行存储，但是每一次代码升级或更新都需要浏览器下载新的代码，最简单方便的方式就是引入新的文件名称。webpack中可以在output中指定chunkhash，并且分离经常更新的代码和框架代码。通过NameModulesPlugin或HashedModuleIdsPlugin使再次打包文件名不变。

### 11. Webpack如何配置单页面和多页面的应用程序

```javascript

    //单个页面
    module.exports = {
        entry: './path/to/my/entry/file.js'
    }
    //多页面应用程序
    module.entrys = {
        entry: {
            pageOne: './src/pageOne/index.js',
            pageTwo: './src/pageTwo/index.js'
        }
    }

```

### 12. 如何提高Webpack的构建速度

- 多入口情况下，使用CommonsChunkPlugin来提取公共代码

- 通过externals配置来提取常用库

- 利用DllPlugin和DllReferencePlugin预编译资源模块 通过DllPlugin来对那些我们引用但是绝对不会修改的npm包来进行预编译，再通过DllReferencePlugin将预编译的模块加载进来。

- 使用Happypack 实现多线程加速编译

- 使用webpack-uglify-parallel来提升uglifyPlugin的压缩速度。 原理上webpack-uglify-parallel采用了多核并行压缩来提升压缩速度

- 使用Tree-shaking和Scope Hoisting来剔除多余代码

### 13. 如何利用Webpack来优化前端性能（提高性能和体验）

- 压缩代码。删除多余的代码、注释、简化代码的写法等等方式。可以利用webpack的UglifyJsPlugin和ParallelUglifyPlugin来压缩JS文件， 利用cssnano（css-loader?minimize）来压缩css

- 利用CDN加速。在构建过程中，将引用的静态资源路径修改为CDN上对应的路径。可以利用webpack对于output参数和各loader的publicPath参数来修改资源路径

- 删除死代码（Tree Shaking）。将代码中永远不会走到的片段删除掉。可以通过在启动webpack时追加参数--optimize-minimize来实现

- 提取公共代码

### 14. Webpack4有哪些变化

- mode/–mode参数，新增了mode/--mode参数来表示是开发还是生产（development/production）。production 侧重于打包后的文件大小，development侧重于goujiansud

- 移除loaders，必须使用rules（在3版本的时候loaders和rules 是共存的但是到4的时候只允许使用rules）

- 移除了CommonsChunkPlugin (提取公共代码)，用optimization.splitChunks和optimization.runtimeChunk来代替

- 支持es6的方式导入JSON文件，并且可以过滤无用的代码

```javascript

let jsonData = require('./data.json')
import jsonData from './data.json'
import { first } from './data.json' // 打包时只会把first相关的打进去

```
- 升级happypack插件（happypack可以进行多线程加速打包）

- ExtractTextWebpackPlugin调整，建议选用新的CSS文件提取kiii插件mini-css-extract-plugin，production模式，增加 minimizer

## 第七部分. HTML && CSS(不做整理，推荐其他)

### 1. 前端基础面试题(HTML+CSS部分)

前端基础面试题(HTML+CSS部分): [https://zhuanlan.zhihu.com/p/28415923](https://zhuanlan.zhihu.com/p/28415923)

### 2. web前端面试中10个关于css高频面试题

web前端面试中10个关于css高频面试题,你都会吗?: [https://cloud.tencent.com/developer/article/1476141](https://cloud.tencent.com/developer/article/1476141)

### 3. 2019 CSS经典面试题

2019 CSS经典面试题（史上最全，持续更新中...）: [https://blog.csdn.net/weixin_33691817/article/details/91392503](https://blog.csdn.net/weixin_33691817/article/details/91392503)


## 参考资料

- [中高级前端大厂面试秘籍，为你保驾护航金三银四，直通大厂(上) - 掘金](https://juejin.im/post/5c64d15d6fb9a049d37f9c20#heading-30)
- [中高级前端大厂面试秘籍，寒冬中为您保驾护航，直通大厂(中) - React篇](https://juejin.im/post/5c92f499f265da612647b754#heading-4)
- [JavaScript 面试 20 个核心考点 ](https://mp.weixin.qq.com/s?__biz=MzU3MDAyNDgwNA==&mid=2247484912&idx=1&sn=4b91963d14bfc9f5d7799525bebf423c&scene=19#wechat_redirect)
- [React高频面试题梳理，看看面试怎么答？（上） ](https://mp.weixin.qq.com/s?__biz=MzU3MDAyNDgwNA==&mid=2247484938&idx=1&sn=5707d470d5a3c29db0f29a683ce7d02a&scene=19#wechat_redirect)
- [前端面试题整理—Webpack+Git篇](https://www.cnblogs.com/theblogs/p/10781273.html)




