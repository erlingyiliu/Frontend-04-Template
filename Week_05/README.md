学习笔记

## JS 表达式|运算符和表达式

###  Expression Grammar

#### Tree VS Priority 语法树和优先级

我们进行四则运算的时候，知道乘除的优先级比加减高，想要改变运算顺序还要加括号，括号的优先级比乘除更高。所以我们构造语法树的时候，需要注意这种情况，优先级更高的优先形成更小一级的语法结构。

所以一个简单的 1 + 2 * 3 的语法树大概是这个样子的：

```
		+
	1		*
		2		3
```

这个语法树并不严谨，但是可以大概说明，优先级会影响到语法树的构成。实际上 JavaScript 语言使用产生式来描述 JavaScript 的运算优先级。

我们来看看运算符的优先级

##### Member Expression

Member 运算的优先级最高，表现形式有

+ a.b
+ a[b]
+ foo\`string\`
+ super.b
+ super["b"]
+ new.target
+ new Foo()

##### New Expression

+ new Foo

```js
// Example
new a()(); // 执行顺序是 new a()，然后执行第二个 ();
new new a();	// 执行顺序是 new a()，然后执行第一个 new
```

#####  Call Expression

+ foo()
+ super()
+ foo()['b']
+ foo().b
+ foo()\`abc\`

```js
// Example
/**
* 下面这几种情况的 Member Expression，会降级到 Call Expression
*/
new a()['b']; // 先执行 new a()，然后执行['b'],然后 new a()['b'] 的优先级是 Call Expression
foo().b
foo()`abc`
foo()['b']
```

> 语法结构能够表达的内容是多于运算符优先级表达的内容。上面的点运算符，本身也可以有不同的优先级，用它前面的语法结构来决定自己的优先级。所以用优先级来解释运算符并不是一个严谨的说法。真正严谨的还是使用产生式，一级一级的语法结构，来描述运算符的优先级。



### Left handside Expression &  Right handside Expression 

```js
// Example
a.b = c;
a + b = c; // 不可以，
```

> 这是因为 a.b 是一个 Left Handside，a + b 是一个 Right Handside，只有 Left Handside 才可以放到 = 左边，这个是各种编程语言都会使用的一个概念。在 JavaScript 中没有定义 Right Handside Expression，因为所有的 Expression 不属于 Left handside，就是 Right Hand

不能放到表达式左边的：

+ Update 自增表达式

  + a ++
  + a --
  + -- a
  + ++ a

  ```js
  // Expression
  ++ a ++; // 不合法的
  ```

+ Unary 单目运算符

  + delete a.b
  + void foo()
  + typeof a
  + \+ a
  + \- a
  + ~ a : 可以将整数按位取反，如果不是整数强制传唤成整数
  + ! a
  + await a

+ Exponental 

  + ** : 幂运算符

  ```js
  // Expression
  3 ** 2 **3 
  // 等同于
  3 ** (2 ** 3);
  ```

+ Multplicative 

  + \* 	/ 	%

+ Additive

  + \+ 	-

+ shift

  +  <<	  >>	 >>>

+ Relationship

  + \< 	\>	<=	>=	instanceof	in

+ Equality

  +  ==
  +  !=
  +  ===
  +  !===

+ Bitwise

  + &	|	^

+ Logical

  + &&
  + ||

+ Conditional

  + ? : 三目运算符，当 ? 前的表达式为 true 时，: 后面的表达式不执行，是短路逻辑。



### Expression Runtime



#### Type Convertion 类型转换

+ a + b

+ "false" == false

+ a[o] = 1;

+ 七种基本类型之间进类型转换

  |           | Number         | String           | Boolean  | Undefined | Null | Object | Symbol |
  | --------- | -------------- | ---------------- | -------- | --------- | ---- | ------ | ------ |
  | Number    | -              |                  | 0 false  | x         | x    | Boxing | x      |
  | String    |                | -                | "" false | x         | x    | Boxing | x      |
  | Boolean   | true 1  false0 | "true"   "false" | -        | x         | x    | Boxing | x      |
  | Undefined | NaN            | "Undefined"      | false    | -         | x    | x      | x      |
  | Null      | 0              | "null"           | false    | x         | -    | x      | x      |
  | Object    | valueof        | valueof toString | true     | x         | x    | -      | x      |
  | Symbol    | x              | x                | x        | x         | x    | Boxing | -      |

+ Unboxing 拆箱转换

  + ToPremitive 

    + ToPremitive 发生在表达式定义的方方面面，一个对象上有三个方法会影响到 ToPremitive，分别是 toString、valueof、Symbol.toPrimitive,如果定义了 Symbol.toPrimitive，它就会忽略 toSting 和 valueOf。没有定义则会根据提示来决定调用 toString 和 valueOf 的先后顺序。

      ```js
      // Expression
      var o = {
        toString() {return "2"},
        valueOf() {return 1},
        [Symbol.toPrimitive]() { return 3}
      }
      console.log("x" + o); // 在 o 的 Symbol.toPrimitive 存在的情况下，结果为x3，如果不存在结果为 x1，优先调用 valueOf。
      var x = {};
      
      x[o] = 1; // 不存在 Symbol.toPrimitive 的情况下，优先调用 toString()；
      
      ```

      

  + ToString VS value of

  + Symbol.toPrimitive

+ Boxing 装箱转换

  | 类型    | 对象                    | 值          |
  | ------- | ----------------------- | ----------- |
  | Number  | New Number(1)           | 1           |
  | String  | New String("a")         | "a"         |
  | Boolean | New Boolean(true)       | true        |
  | Symbol  | new Object(Symbol("a")) | Symbol("a") |

  装箱转换是自动进行的。



#### Reference 引用类型

a.b 访问一个属性，取出的不是属性的值，而是一个引用，这个引用不是 JavaScript 语言的 7 中基本类型之一，而是标准中的类型 Reference。

一个 Reference 分成两个部分：

+ Object
+ key

一个 Reference 类型取出的 是一个 Object 和一个 key，记录了 Member 运算的前半部分和后半部分。

在 JavaScript 中 delete 和 assign 这样的基础设施，运行时就会用到 Reference 类型，如果是加法或减法运算，我们就会把 Reference 直接解引用，像普通的变量一样去使用。

```js
// delete
delete a.b;

// assign
a.b = ...;
a.b += ...;
```



## JS 语句

### Statement 

Grammar

+ 简单语句
+ 组合语句
+ 声明

Runtime

+ Completion Record 语句执行结果记录
+ Lexical Environment 作用域

#### Completion Record

Completion Record 记录语句执行的结果

Completion Record 组成

+ [[type]]：normal，break，continue，return， or throw
+ [[value]]：基本类型
+ [[target]]: label (在语句前加一个标识符和 ":"，)

#### 简单语句

+ Expression Statement

  表达式语句：一个表达式加一个 ";"

+ Empty Statement

  空语句：单独的一个分号，没什么大用，只是从语言的完备性上来讲，允许出现空语句。

+ Debugger Statement

  `Debugger;` 调试的时候使用的语句。

+ Throw Statement

  抛出异常，如果不想真的发生错误，可以主动使用用 `throw Expression statement` 来抛出一个异常。

+ Continue Statement

  与循环语句相匹配，

+ Break Statement

  与循环语句相匹配，switch 语句；

+ Return Statement

  在函数中使用 return 返回一个值

> 简单语句中只有 Expression Statement，可以驱动计算机进行计算，其他的语句都是流程控制的作用，或者说没啥用。

#### 复合语句

+ Block Statement

  块语句：是完成语句的树状结构的重要的基础设施。

+ If Statement

  分支结构：条件语句：

+ Switch Statement

  多分支结构：

+ Iteration Statement

  循环语句，while 循环、do...while 循环、for 循环、for await of 循环

+ With Statement

  with 语句，通过 with 打开一个对象，把这个对象的所有的属性直接放进作用域里面。不确定性高，现代的 JavaScript 规范里面拒绝使用 with 语句。

+ Labelled Statement

  在简单语句、复合语句前面加一个 label，相当于给语句取了一个名字。只有和 Iteration Statement 语句配合使用才会有意义。

+ Try Statement

  Try{}catch{} finally{}， try 后面的 {} 不可以省略

##### Block Statement

可以容纳多个语句。Block Statement 返回的 Completion Record 对象的 type 属性一般是 normal，value 和 target 不明确。

##### Iteration Statement

+ while() {}
+ do ... while() {}
+ for(...;...;...) ...
+ for(... in ...) ...
+ for(... of ...) ...
+ for await (of)
+ in 在 for 循环中是不允许出现 in 操作符的。

##### break continue 和 labelled Statement。

break continue 配合标签可以在多重循环中使用。可以跳出多重循环，

#####  try

```
try {
	
} catch(e) {

} finally {

}
```

注意一下 return 你在 try 中 return 了返回值， finally 中的代码也会继续执行。



#### 声明

+ Function Declaration
+ Generator Declaration
+ AsyncFunction Declaration
+ AsyncGenerator Declaration
+ VariableStatement: 变量语句 var
+ Class Declaration
+ Lexical Declaration let const 声明变量语句。

| function        | class |
| --------------- | ----- |
| function*       | const |
| async function  | let   |
| async function* |       |
| var             |       |

上表中左侧的声明只认作用域，没有先后关系，

##### 预处理（pre-process）

预处理是指在一段代码执行前，JavaScript 引擎会对代码本身做一次预先处理。

```js
var a = 2;
void function () {
  a = 1;
  return ;
  var a;
}();

console.log(a); // 2

var a = 3;
void function () {
	a = 1;
  return ;
  const a;
}();
console.log(a); // 抛出错误
```

所有的声明都是有预处理机制的，都能够把变量变成一个局部变量。const let 声明的变量在声明前使用会抛出错，而且可以被 try 捕捉到。



##### 作用域

在代码中变量的生效范围，就是作用域。

var 和 function 的作用范围，都是整个的函数体，不管你把 var 写在哪。const 就在所在的  {} 中的，如果 let 和 const 配合 for 循环使用，那么 const 生效的范围，是整个循环，也可以在函数中使用多个 {} 把函数分成小结，不会产生干扰。



## JS 结构化 | 宏任务和微任务

### JS 执行粒度（运行时）

+ 宏任务

  传递给 JavaScript 引擎的任务，

+ 微任务（Promise）

  JavaScript 引擎内部的任务 

+ 函数调用（Execution Context)

+ 语句/声明（Completion Record）

+ 表达式（Reference）

+ 直接量/变量/this ......

#### 宏任务和微任务

 JavaScript 引擎其实是一个静态的库，使用 JavaScript 引擎的时候，会把一段代码传给它。

```js
var x = 1;
var p = new Promise(resolve => resolve());
p.then(() => x = 3);
x = 2;
```

我们塞给 JavaScript 引擎一段代码，这个段代码产生两个异步任务，第一段是 x=1；p=...；x=2; 第二段就是 x=3；这两个异步任务，我们称为 MicroTask，在 JavaScript 标准里称为一个 job，把这段代码塞给引擎，并且进行执行的整个过程，称为一个 MacroTask，

#### 事件循环

事件循环也是跳出 JavaScript 引擎的的概念，是我们如何使用 JavaScript 引擎的过程，事件循环 event loop 本身是来自 node 里面的一个概念。

事件循环有三个部分，第一个部分就是获取代码，第二个部分就是把这个代码给执行掉，第三分布就是等待，然后继续获取代码，这个等待的过程有可能是等待一个事件，一个时间。

## JS 结构化 | JS 函数调用

```js
import {foo} from "foo.js";

var i = 0;
console.log(i);
foo();
console.log(i);
i++;
```

```js
function foo() {
  console.log(i);
}
export foo;
```

这两段代码中，有 5 次使用到了变量 i，这 5 个 i 是同一个么？

不是， foo() 中的 i，是 foo() 环境里面的 i，foo() 环境里面没有定义就会报错。

一个更复杂的例子：

```js
import {foo} from "foo.js";

var i = 0;
console.log(i);
foo();
console.log(i);
i++;
```

```js
import {foo2} from "foo2.js";

var x = 1;
function foo() {
  console.log(x);
  foo2();
  console.log(x);
}

export foo;
```

```js
var y = 2;
function foo2(){
	console.log(y);
}
export foo2;
```

这三段代码都可以调用；这5 段代码形成了一个栈式的调用关系。

函数调用本身也会形成一个 stack。

```
	Execution Context			    				Execution Context										Execution Context
			i:0																x:1																	y:2
																	===>
													Execution	Context Stack
```

上面方框中，是一个从左到右 stack；每个 Stack 里面保存的东西我们把它称作 Execution Context(执行上下文)，我们执行一个语句的时候，所需要的信息都会保存在这个 Execution Context 中。保存 Execution Context 的数据结构，我们就把它称作：Execution Context Stack 执行上下文栈。

当我们执行到当前语句的时候，栈有一个栈顶元素，栈顶元素就是我们当前能访问到的所有的变量，这些变量我们叫 Running Execution Context，代码里所需要的一切信息，都要从 Running Execution Context 中取出来，

### Execution Context

Execution Context 里面有 7 大件，但是没有一个 Execution Context 是 7 件齐全的。

+ code evaluation state 

  用于 async 和 generator 函数，它是代码执行到哪了的保存信息。

+ Function

  由 Function 来初始化

+ Script or Module

  两者只存其一，只不同部分的代码，要么是 script 中的，要么是 module 中的。

+ Generator

  只有 Generator 函数需要，Generator 函数每次执行所生成的隐藏在背后的 Generator 字段

+ Realm

  保存所有使用内置对象的领域。

+ Lexical Environment

  执行代码中所需要访问的环境，保存变量的

+ Variable Environment

  var 声明变量会声明到哪个部分。

Execution Context 分成了几个种类

+ ECMAScript Code Execution Context
  + code evaluation state
  + Function
  + Script or Module
  + Realm
  + Lexical Environment
  + Variable Environment
+ Generator Execution Contexts
  + code evaluation state
  + Function
  + Script or Module
  + Realm
  + Lexical Environment
  + Variable Environment
  + Generator

#### Lexical Environment

Lexical Environment 在老版本里面，只存变量，ES2018 以后的标准里，它保存所有执行时候存的东西。

+ this
+ new.target
+ super
+ 变量

#### Variable Environment

Variable Environment 是历史遗留的包袱，仅仅用于处理 var 声明

```js
{
  let y = 2;
  eval('var x = 1;');
}

with({a:1}) {
  eval('var x;');
}
console.log(x);
```

第一个{} 中`eval('var x = 1;');` 会把 var 声明到一个更大的范围，function body、script、module 中，

第二个 with 语句，中的 `eval('var x;');`  会穿透 with 语句，声明到了一个更大的范围。

#### Environment Record

Environment 是一个链式结构，链式结构的每一个点，称为一个 Environment Record。Environment Record 有一个继承关系，是一个家族，Environment Record 的基类就是 Environment Records，它下面有三个子类：Declarative Environment Records、Global Environment Records、Object Environment Records，直接去看 Global 和 Object 都是给特殊场景用的，Object 是给 with 用的，在这里我们大量生成和产生的是 Declarative Environment Records，{} 块语句就会生成一个 Declarative Environment Records，Function Environment Records、和 module Environment Records，都有各自特殊的地方。

```
																		Environment Records
										|											|														 |
							Declarative 								Global										Object
					Environment Records					Environment Records			Environment Records
	
				Fucntion						Module
	Environment Records		Environment Records
```

#### Function - Closure

JavaScript 里每一个函数都会生成一个闭包，根据闭包的经典定义，闭包由两部分组成，Environment Record 和 Code。Environment 部分包括一个 Object 和一个变量的序列来组成。

在 JavaScript 中每个函数都会带一个定义时所在 Environment  Records，它会把 Environment Records 保存到自己的函数对象身上，变成一个属性。

简单的例子：

```js
var y = 2;
function foo2() {
  console.log(y);
}
export foo2;
```

> 在上面的代码中，foo2 定义时有一个，外面有一个 y = 2，那么不管这个 foo2 最后被通过参数或者 export、import 这样的东西传到哪里去，它都会带上一个 y = 2 的 Environment Record，这个就是闭包，也是 Environment Record 能够形成链的关键的设施。

更复杂的例子：

```js
var y = 2;
function foo2() {
  var z = 3;
  return () => {
    console.log(y, z);
  }
}
var foo3 = foo2();
export foo3;
```

> foo3 在执行的时候，它的会有 foo2 执行时内部形成的一个闭包，有一个 z = 3 的 Environment Record，foo2() 所在环境的 y = 2，会被作为 z = 3 环境的上一级保存下来，这就是 Environment Record 形成的一个链式结构。我们还要注意，因为有箭头函数，所以 z = 3 环境执行时使用的 this 也保存了下来，所以这一条 Environment Record 里面有 z = 3, this = global 两条记录，加上更外层的 y = 2，一共有 3 条记录，return 后面的箭头函数里面，可以同时访问 y、z 和 this。

#### Realm

在 JS 中，函数表达式和对象直接量均会创建对象。

使用 "." 做隐式转换也会创建对象。

这些对象也是有原型的，如果我们没有 Realm，就不知道它们的原型是什么。 

我们去执行一个表达式的过程中，除了变量、this、new.target 这些东西，还需要一些其他的信息。比如说对象直接量{}，数组直接量[]，这些东西都会产生 object，产生的 object 也是有原型的，如果使用过 iframe 的话，在不同的 iframe 里面创建对象，它的原型是不一样的，这个原型需要一个东西去记录，JavaScript 就在标准里面定义了一个叫做 Realm 的东西。规定了在一个 JavaScript 引擎的实例里面，所有的内置对象会被放进一个 Realm 里面去，在不同的 Realm 实例之间是互相独立的，意味着我们使用 instanceof 可能会失效。进行装箱转换的时候，也会创建出来的对象也会使用 Realm。有了 Realm 之后我们就可以去执行这些对应的表达式描述它们的行为，而 Realm 会根据外部条件去创建不同的 Realm，不同的 Realm 实例之间，也可以互相传递对象。传递后 prototype 是不一致的。

