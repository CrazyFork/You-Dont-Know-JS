
# notes:

## types & grammar
* ch2
    * Number.NAN !== Number.NAN // true
    * Object.is 的代码实现
        ```
        +0 === -0 // true
        Object.is(-0, +0) // false
        Object.is(-0, -0) // true
        Object.is(Number.NAN, Number.NAN) // true
        ```
    * Number.EPSILON, 是浮点数的最小精度, 判断浮点数可以用差值来比较

* ch3 
    * 有的看懂了,  有的没看懂, 感觉也没啥用, 没看懂的部分

* ch04
     * 这章讲的类型转换, 有的地方太高深了, 感觉也一般用不上, 但是如果出问题的话, 这个地方将是个很好的参考

    * > For regular objects, unless you specify your own, the default toString() (located in Object.prototype.toString()) will return the internal [[Class]] (see Chapter 3), like for instance "[object Object]".

    * JSON.stringfy 的规则还挺有意思, 虽然没啥用
        * >JSON stringification has the special behavior that if an object value has a toJSON() method defined, this method will be called first to get a value to use for serialization.
        * > toJSON() should return the actual regular value (of whatever type) that's appropriate, and JSON.stringify(..) itself will handle the stringification.

    * coersion

    * number
         `ToPrimitive` 先寻找 valueOf 然后 toString 方法, 没有就  type error

    * falsey:
        * false: `undefined, null, 0, NAN, ""`


    * > ~x is roughly the same as -(x+1). That's weird, but slightly easier to reason about.

    *  number parsing
        * >always pass 10 as the second argument

        * 这是因为如果字符串中有 09 这样的字符, 以前规定的行为是以 8 进制去parse, 9 在8进制没有,所以会返回0

        * parseInt  会将目标对象转换成字符串, 然后去parse, 这意味着会先调用目标对象的 toString 方法

    * 后面的  equality 没有读, 感觉没啥太大价值.



## this & object prototypes

* ch01:
    * so what is this ?

*  ch02:
    * this 这个东西指向的到底是什么, 在 `Determining this` 这一节 解释的十分清楚, 总共四个规则

    * 

* ch03:
    * > In objects, property names are always strings.


    * > As you can see, the last delete call failed (silently) because we made the a property non-configurable

    *
    ```
    preventExtensions // 不允许添加属性
    seal //不允许添加,更改属性配置
    Freeze // 不允许添加,更改属性配置, 也不允许更改属性值

    Object.keys(..) returns an array of all enumerable properties, whereas 
    Object.getOwnPropertyNames( myObject ); //  returns an array of *all* properties, enumerable or not.
    ```

* ch04:
    * >

* ch05:
    * Setting & Shadowing Properties
        * `=` & `Object.defineProperty` 的区别, 
            * `=` 如果 prototype chain 上面 有, 且是 writable=false, 会报错, 否则会动态添加一个 prop.
            * `Object.defineProperty` 会忽略上边的规则
        
        * Shadowing  Properties: 这个意思就是说 prototype chain 上面有,  然后用 `=` 重新覆盖了原有属性的行为.

            ```js
            var anotherObject = {
                a: 2
            };

            var myObject = Object.create( anotherObject );

            anotherObject.a; // 2
            myObject.a; // 2

            anotherObject.hasOwnProperty( "a" ); // true
            myObject.hasOwnProperty( "a" ); // false

            myObject.a++; // oops, implicit shadowing!

            anotherObject.a; // 2
            myObject.a; // 3

            myObject.hasOwnProperty( "a" ); // true
            ```
    *  prototype:
        ```
        // pre-ES6
        // throws away default existing `Bar.prototype`
        Bar.prototype = Object.create( Foo.prototype );

        // ES6+
        // modifies existing `Bar.prototype`
        Object.setPrototypeOf( Bar.prototype, Foo.prototype )
        ```

        ```js
        // 判断该object 是否是 另一个object的  __proto__
        Object.prototype.isPrototypeOf()
        ```

    * `__proto__` vs `prototype`
        * 标注上来讲 `__proto__` 和 `[[prototype]]` 是一样的, 是真正的 js prototype chain. `Object.create` 和 `Object.setPrototypeOf` 修改的都是这个 `__proto__` 属性的指向

        * 而 `prototype` 这个属性,  指的是 class 或者用 function 作为class 用的function对象上的属性值, 这个属性值会在调用 new Function(), 生成对象的时候
        通过
        ```js
        let a = Object.create(Function.prototype)
        Function.apply(a, arguments)
        return a
        ```
        这样的一种操作方式, 生成对象的一个实例, 其实就是一个临时的一个属性 , 绑定在 Function上, 在用 `new` 关键字创建对象的时候才会用到.

        * 理解 function 的 prototype 属性一定要先理解 new 操作符, 书里边有解释, 但是似乎找不到了.

            ```js
            function Person{this.name = 'hi'}
            Person.prototype.sayHi = function(){console.log(this.name)}
            // what new essentially does is 
            let a = Object.create(Person.prototype)
            Person.appy(a, args)
            ```

    * ` instanceof` 关键字:

        ```js
        Function B(){}
        a instanceof  B // 这句话的意思就是说 a 的 [[prototype]]  chain 里边有没有 B.prototype 这个对象, 有就返回  true

        // 跟这个是同理的
        B.prototype.isPrototypeOf(a)
        ```
        
    * [[ prototype]] 工具方法

        ```js
        // 获得 a 的 [[prototype]]
        Object.getPrototypeOf( a );
        
        // create a empty object.
        Object.create(null)
        ```

    * `__proto__` 实现

        ```js
        Object.defineProperty( Object.prototype, "__proto__", {
            get: function() {
                return Object.getPrototypeOf( this );
            },
            set: function(o) {
                // setPrototypeOf(..) as of ES6
                Object.setPrototypeOf( this, o );
                return o;
            }
        } );
        ```

* ch06:
    * 第五章读完了感觉就没有必要读第六章了, 道理都懂, 但是实在没看懂 说的一大堆的结论的意义是什么, 还有尽管 class 机制掩盖了 js 中的 object delegation 行为, 但实际上从用户 的 mental model 上回更加容易理解其代码行为.


## Scope & Closures

* ch01:
    * compiler 和 engine 的一个大致原理, 基本上js被下载下来会经过engine(v8 engine for instance)解析(parse)成AST, 然后在执行阶段
        去动态翻译AST, 在运行的时候变量会根据AST执行的时候动态创建变量的Scope, 这个东西是啥数据结构实现的就不知道了, (stack ?)

    * LHS & RHS
        > "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".
    * 
* ch02:
    * eval:
        * eval 会将字符串代码在所在位置执行, 可以动态创建代码执行的一种元编程机制
        * (1 && eval)("var a = 3"), indirect evaluation 将在 global mode 执行
        * eval 在 strict mode 中不会修改外层的 lexical scope, 会在自己的域中创建 variable.

            ```js
            function t(a){
                "use strict";

                eval(a)
                console.log(b)
            }   
            t("var b = 3") // undefined, or a error been throw.
            ```
* ch03:
    * 这章主要讲了 let, var scpoe 的区别, 还有 functional scope 在js里边应用的一些典型范式
    * scope 的执行机制就是越到里层, 里边的scope会包含和shadow外层的scope.
    * 

* ch04: Hoisting
    * 这章主要讲了 js 变量提升的机制, 基本上的意思就是说 js 中变量提升会把 var 声明提到 function scope 最上面. 后面的赋值按照
        正常的逻辑理解就可以了. 可以说 js 变量提升这个机制还是蛮恶心的. 需要注意的规则有

        * function first: 这意味着任何 `function <name>(){}` 的声明永远会在 var 前面, 而且后面同样 name 的function 会覆盖
            前面的 function 声明, 这里需要注意的是, 即使在 if 这种 condition 语句中, function 的声明会skip掉 condition 语句
            直接使用后面的. 而 `var f = function name(){}` 这种 variable 声明的function 不会被提升到最上面, 指的是f这个变量.


        * `var a = function(){}` 注意这里边 js 看到的是两个statement, `var a` & `a=function(){}`, 这意味着在对应scope上执行
            a() 会导致typeerror, 因为 `var a` 只是声明了, 默认是undefined.
            
* ch05: Scope Closure
    * 这个就没有必要理解了, 感觉自己其实理解的不错了

* apA: dynamic scope
    *  >lexical scope is write-time, whereas dynamic scope (and this!) are runtime. 









todo:



# You Don't Know JS (book series)

This is a series of books diving deep into the core mechanisms of the JavaScript language. The first edition of the series is now complete.

<a href="http://www.ebooks.com/1993212/you-don-t-know-js-up-going/simpson-kyle/"><img src="up %26 going/cover.jpg" width="75"></a>&nbsp;
<a href="http://www.ebooks.com/1647631/you-don-t-know-js-scope-closures/simpson-kyle/"><img src="scope %26 closures/cover.jpg" width="75"></a>&nbsp;
<a href="http://www.ebooks.com/1734321/you-don-t-know-js-this-object-prototypes/simpson-kyle/"><img src="this %26 object prototypes/cover.jpg" width="75"></a>&nbsp;
<a href="http://www.ebooks.com/1935541/you-don-t-know-js-types-grammar/simpson-kyle/"><img src="types %26 grammar/cover.jpg" width="75"></a>&nbsp;
<a href="http://www.ebooks.com/1977375/you-don-t-know-js-async-performance/simpson-kyle/"><img src="async %26 performance/cover.jpg" width="75"></a>&nbsp;
<a href="http://www.ebooks.com/2481820/you-don-t-know-js-es6-beyond/simpson-kyle/"><img src="es6 %26 beyond/cover.jpg" width="75"></a>

Please feel free to contribute to the quality of this content by submitting PR's for improvements to code snippets, explanations, etc. While typo fixes are welcomed, they will likely be caught through normal editing processes, and are thus not necessarily as important for this repository.

**To read more about the motivations and perspective behind this book series, check out the [Preface](preface.md).**

## Titles

* Read online (free!): ["Up & Going"](up\%20&\%20going/README.md#you-dont-know-js-up--going), Published: [Buy Now](http://www.ebooks.com/1993212/you-don-t-know-js-up-going/simpson-kyle/) in print, but the ebook format is free!
* Read online (free!): ["Scope & Closures"](scope\%20&\%20closures/README.md#you-dont-know-js-scope--closures), Published: [Buy Now](http://www.ebooks.com/1647631/you-don-t-know-js-scope-closures/simpson-kyle/)
* Read online (free!): ["this & Object Prototypes"](this\%20&\%20object\%20prototypes/README.md#you-dont-know-js-this--object-prototypes), Published: [Buy Now](http://www.ebooks.com/1734321/you-don-t-know-js-this-object-prototypes/simpson-kyle/)
* Read online (free!): ["Types & Grammar"](types\%20&\%20grammar/README.md#you-dont-know-js-types--grammar), Published: [Buy Now](http://www.ebooks.com/1935541/you-don-t-know-js-types-grammar/simpson-kyle/)
* Read online (free!): ["Async & Performance"](async\%20&\%20performance/README.md#you-dont-know-js-async--performance), Published: [Buy Now](http://www.ebooks.com/1977375/you-don-t-know-js-async-performance/simpson-kyle/)
* Read online (free!): ["ES6 & Beyond"](es6\%20&\%20beyond/README.md#you-dont-know-js-es6--beyond), Published: [Buy Now](http://www.ebooks.com/2481820/you-don-t-know-js-es6-beyond/simpson-kyle/)

## Publishing

These books are being released here as drafts, free to read, but are also being edited, produced, and published through O'Reilly.

If you like the content you find here, and want to support more content like it, please purchase the books once they are available for sale, through your normal book sources. :)

If you'd like to contribute financially towards the effort (or any of my other OSS work) aside from purchasing the books, I do have a [patreon](https://www.patreon.com/getify) that I would always appreciate your generosity towards.

<a href="https://www.patreon.com/getify">[![patreon.png](https://s13.postimg.org/k9nkc5thz/become_a_patron_button.png)](https://www.patreon.com/getify)</a>

## In-person Training

The content for these books derives heavily from a series of training materials I teach professionally (in both public and private-corporate workshop format): "Deep JavaScript Foundations", "Rethinking Async", and "ES6: The Right Parts" workshops.

If you like this content and would like to contact me regarding conducting training on these, or other various JS/HTML5/node.js topics, please reach out to me through email: getify @ gmail

## Online Video Training

I also have some JS training material available in on-demand video format. I teach courses through [Frontend Masters](https://FrontendMasters.com), like my [Deep JavaScript Foundations](https://frontendmasters.com/courses/javascript-foundations/) workshop. You can find [all my courses here](https://frontendmasters.com/kyle-simpson/).

Some of those courses are also distributed on other platforms, like Pluralsight, Lynda.com, and O'Reilly Safari Online.

## Contributions

Any contributions you make to this effort **are of course greatly appreciated**.

But **PLEASE** read the [Contributions Guidelines](CONTRIBUTING.md) carefully before submitting a PR.

## License & Copyright

The materials herein are all (c) 2013-2017 Kyle Simpson.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivs 4.0 Unported License</a>.
