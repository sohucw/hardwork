> 前言 之前面试的时候,面对候选人的筛选 (都对vue react 框架掌握的差不多)
考察基础的js都不是特别熟练. 如果你是候选人 框架能力&js基础都很好 那应该是不错的人选.
### 常见的面试题分析

- 直接来几道面试题 看看你能否答对

```
    var foo=1;
    function bar( ) {
      foo=10;
      return function foo() { };
    }
    bar( );
    alert(foo);
    **********分割线***********
    var foo = 1;
    function bar() {
        foo = 10;
        return;
        function foo() {}
    }
    bar();
    console.log(foo)
    **********分割线***********
    var a = 1;
    function fn() {
        console.log(a);
        var a = 2;
    }
    fn()
    console.log(a);
    **********分割线***********
    var a = 1;
    function fn(a) {
        console.log(a);
        a = 2;
    }
    fn(a)
    console.log(a);

```
如果你能够都答对可以忽略本文的阅读.免得浪费时间

- 接下来我要带大家分析下为什么是这样的结果
分析之前你需要明白的是
创建应用程序的时候,总免不了要声明变量和函数 解析器（interpreter）是如何以及从哪里找到这些数据（变量，函数）的，
当我们引用一个变量时，在解析器内部又发生了什么？

我们知道 变量和执行上下文相关 那么它就应该知道数据储存在哪里以及如何访问这些数据，这种机制被称为**变量对象（variable object）**。
变量对象（简称为 VO）是与某个执行上下文相关的一个特殊对象，并储存了一下数据
1. 变量（var, VariableDeclaration）
2. 函数声明（FunctionDeclaration, 缩写为FD）
3. 函数形参
也就是所有的执行中的变量都会存储在 这个**vo**中


- 处理上下文代码的几个阶段 本文最核心的部分了 处理执行上下文代码分为两个阶段:
	-  **进入执行上下文**
	    * 当进入执行上下文时（在代码执行前），VO 就会被下列属性填充
	         1. 函数的所有形参（如果是在函数执行上下文中）
              每个形参都对应变量对象中的一个属性，该属性由形参名和对应的实参值构成，如果没有传递实参，那么该属性值就为 undefined
             2. 所有函数声明（FunctionDeclaration, FD）
              每个函数声明都对应变量对象中的一个属性，这个属性由一个函数对象的名称和值构成，如果变量对象中存在相同的属性名，则完全替换该属性。
             3. 所有变量声明（var, VariableDeclaration）
              每个变量声明都对应变量对象中的一个属性，该属性的键/值是变量名和 undefined，如果变量名与已经声明的形参或函数相同，则变量声明不会干扰已经存在的这类属性。

            ```
             举例说明：
            function test(a, b) {
              var c = 10;
              function d() {}
              var e = function _e() {};
              (function x() {});
            }

            test(10);
            分析: 当进入 test 的执行上下文，并传递了实参 10，(VO)AO(VO 函数上下文中的变量对象) 对象如下：
            在函数上下文中，变量对象（VO）不能直接被访问到，此时活动对象（Activation Object，简称 AO）扮演着 VO 的角色。
            就是 VO(functionContext) === AO;
            在这个时候 AO对象的值是

            AO(test) = {
              a: 10,
              b: undefined,
              c: undefined,
              d: <reference to FunctionDeclaration "d">
              e: undefined
            };

            注意：AO 并不包含函数 x，这是因为 x 不是函数声明，而是一个函数表达式（FunctionExpression，简称为 FE），函数表达式不会影响 VO。
            同理，函数 _e 也是函数表达式，就像我们即将看到的那样，因为它分配给了变量 e，所以可以通过名称 e 来访问。
            函数声明与函数表达式的异同.(有时间我在进行分析)

            接下来就到了第二个阶段 代码执行阶段

            ```
	- **执行代码**
	     * 继续以上一例子，到了执行代码阶段，AO/VO 就会修改为如下形式

            ```
               AO['c'] = 10;
               AO['e'] = <reference to FunctionExpression "_e">;

               这里需要注意的是
               函数表达式 _e 仍在内存中，它被保存在声明的变量 e 中。
               但函数表达式 x 却不在 AO/VO 中，如果尝试在其定义前或者定义后调用 x 函数，这时会发生“x未定义”的错误
            ```
* 如果你看懂了我上面的分析 接下来我带大家分析几个例子

```
console.log(a);
var a = 1;
console.log(a);
function a(){alert(2)}
console.log(a);
var a = 3;
console.log(a);
function a() {alert(4);}
console.log(a);

分析流程如下:
 1. 进入执行上下文(预解析)给vo对象定义变量
        1. 第二行有var定义的变量a，将其保存为a=undefined;
        2. 第4行有function声明和第二行的a同名此时由于函数的等级高于变量，于是就覆盖变量a，此时a= function a (){ alert(2); }
        3. 第六行又发现一个var定义的变量，名称与第四行的函数相同，但等级低，故a= function a (){ alert(2); }不变。
        4. 同理，由于第八行后定义，又为同一等级的函数，所以a= function a (){ alert(4); }
        5. 浏览器解析完成
    此时的
    vo= {
       a: <reference to FunctionDeclaration "function a() {alert(4);}">
    }

 2. 开始执行
          1. 第一行 a应该是打印function a() {alert(4);}
          2. 第二行表达式修改了a的值，使其为1，所以第三行输出a=1
          3. 第四行定义了一个函数，但没有执行，所以第五行输出还为1
          4. 第六行表达式又一次更改了a的值，现在a=3，此时的a为一个数字，第七行输出3
          5. 同理，第八行没有做更改，第九行还是输出3
**是不是觉得特别简单 **
```

* 转过头来我来和大家一起分析前面的几道面试题

```
    var a = 1;
    function fn() {
        console.log(a);
        var a = 2;
    }
    fn()
    console.log(a);

    * 解析
    1. 第一行 var定义的变量a=undefined
    2. fn就是函数本身
    vo定义如下:
    vo={
     a: undefined
     fn: <reference to FunctionDeclaration "function fn}">
    }
    * 开始执行代码
    1. 第一行 表达式将修改 a=1
    2. 第二到五行声明函数
    3. 到了fn() 开始执行函数此时函数为 一作用域，只要存在作用域 就先解析
    所有在函数fn内部
      1. 解析 a=undefined
      在fn内部 ao定义如下:
       ao(fn)= { a: undefined}
      2. 执行fn内部第一行  弹出 undefined 第二行 把a赋值为2  (这里的a是在fn中的局部变量 和外包的a全局变量不同)
    4. 最后一行打印全局变量a=1;

    说明 如果把
    function fn() {
            console.log(a);
            a = 2; (把var去掉  此事的a 变成了全局变量)
        }

```

```
分析这道题
var foo = 1;
function bar() {
    foo = 10;
    return;
    function foo() {}
}
bar();
console.log(foo)

* 解析阶段(进入执行上下文)

vo = {
    foo: undefined
    var: <Function bar>
}
* 执行代码阶段
1. 第一行 foo =1;
2. 第6行执行函数bar() (第二行到第5行的定义的函数)
   在函数bar的内部又有解析和执行阶段
   表示如下:
   vo(bar) = {
    foo: <Function "foo">
   }
   执行阶段
   在函数bar内部的第一行 foo=10 (这里需要注意的是 foo是bar的局部变量 不会影响 外面全局的变量foo)
3: 最后一行 console.log() foo=1;

```

```

var foo=1;
function bar( ) {
  foo=10;
  return function foo() { };
}
bar( );
alert(foo);
最后对于这道题 为什么结果是10 不是1  你只要明白 return function foo() { }; 其实是一个函数表达式 不是函数定义. 原因就在这里



var a = 1;
function fn(a) {
    console.log(a);
    a = 2;
}
fn(a)
console.log(a); 这道题关键在于fn(a) 其实就是 fn(1) 在函数内部 解析阶段 a=1 (这里a在函数内部 是形参 a=2 也是改变fn内的局部变量 不影响全局的a  )

```

* 有一点需要说明的是，我们知道，JS中没有块级作用域，只有函数包含的块才会被当做是作用域。诸如for、if等用花括号包含起来的内容是不算作作用域的。
也就是说，其中内容隶属于全局作用域，在全局范围内都可以访问到，如下

```
alert(fn);
if(true){
    var a = 1;
    function fn() {
        alert(123);
    }
}
```


### 总结

 你只要明白了 函数执行分2个阶段 (确定上下文  执行代码)  在确定上下文阶段定义vo(ao)对象值时候 的规则
 规则如下
 argument(函数的形参) > function声明 > var声明 (也就之前提高的变量提升Hoisting)

 * 函数的所有形参（如果是在函数执行上下文中）
 每个形参都对应变量对象中的一个属性，该属性由形参名和对应的实参值构成，如果没有传递实参，那么该属性值就为 undefined
 * 所有函数声明（FunctionDeclaration, FD）
 每个函数声明都对应变量对象中的一个属性，这个属性由一个函数对象的名称和值构成，如果变量对象中存在相同的属性名，则完全替换该属性。
 * 所有变量声明（var, VariableDeclaration）
 每个变量声明都对应变量对象中的一个属性，该属性的键/值是变量名和 undefined，如果变量名与已经声明的形参或函数相同，则变量声明不会干扰已经存在的这类属性。