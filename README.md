# javascript-对象的属性的延伸学习

## 前言

在学习[vue数据绑定](http://www.jianshu.com/p/d3a15a1f94a0)的较底层原理时，被setter和getter困惑了很久，一路追根溯源，通过阅读《你不知道的javascript》和红宝书理解了迷惑我的setter、getter。

## 首先了解什么是属性描述符
[](http://xurenjie.cn:3000/img/dog/dog.png)
在ES5之前，javascript语言没有提供可以检验属性特性的方法，是否只读？不知道；是否可配置？不知道；是否能用for in枚举？不知道。

ES5之后就有了如下的属性描述符：
```
 var Dogger = {
    breed: '柴犬'
 }

 Object.getOwnPropertyDescriptor( Dogger , "breed" );

```
输出：
```
 {
     value: "柴犬",
     writable: true,
     configurable: true,
     enumerable: true
 }

 
```
在这，创建了一个品种为柴犬的dogger。[[Value]]特性将设置为"柴犬",之后操作对breed的任何修改将反映到这个位置。对，没错，这个**getOwnPropertyDescriptor( Dogger , "breed" )**函数就是我们要的检验属性特性的方法。

**默认：**在创建普通对象属性时，属性描述符会使用默认值，即可写可配可枚举。（都为true）

下面分别介绍一下、这几个属性
##writeable
决定是否修改属性的值，是否可以指定新的值给它
```
    Dogger.breed = '哈士奇'
```
很好理解，当writeable为false的时候，其实定义了一个空的setter(等会会提),这个操作将无效,在严格模式下会抛出一个TypeError的错误。
[](http://xurenjie.cn:3000/img/dog/dog4.png)
##configurable
与configurable紧密相连的就是**defineProperty( )**这个方法了，当configurable: false 将不可使用‘好基友’defineProperty( )来配置。后面还会介绍一个会受影响的**delete**。
```
var Dogger = {
    breed: '柴犬'
 }

Object.defineProperty( Dogger, "breed", {
    value: '哈士奇',
    writable: true,
    configurable: false,
    enumerable: true
} )

Dogger.breed // "哈士奇" | 哈哈！我变成了一只哈士奇
Object.defineProperty( Dogger, "breed", {
    value: '柴犬',
    writable: true,
    configurable: true,
    enumerable: true
} ) // TypeError
```
这只作死的柴犬在通过**defineProperty( )**把自己配置成哈士奇之后，顺便把**configurable**修改为**false**，这样之后**defineProperty( )**不管是否严格模式都将报TypeError的错误,这是单向操作，无法撤销。 一失足成千古恨～
[](http://xurenjie.cn:3000/img/dog/dog1.png)
### 例外
还是可以通过writable的方式修改breed的嘛～，不过这里有一个方法可以让dogger彻底绝望，使breed无法修改，也就是这个例外：这个时候**defineProperty( )**还是可以使用的（如下），只可以修改writable，configurable需要与刚才的false一致。

```
Object.defineProperty( Dogger, "breed", {
    value: '哈士奇',
    writable: false,
    configurable: false,
    enumerable: true
} )
```
这样之后柴犬永远变成了只哈士奇。
[](http://xurenjie.cn:3000/img/dog/dog2.png)
### 关于delete
有人说我用delete删除这个breed属性不就好了？


```
delete Dogger.breed
```
之后打印dogger发现它还是一只哈士奇。如下：

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete)的解释如下

> delete 操作符会从某个对象上移除指定属性。成功删除的时候回返回 true，否则返回 false。
Non-configurable properties cannot be removed. This includes properties ofbuilt-in objects like Math, Array, Object and properties that are created as non-configurable with methods like Object.defineProperty( ).
>When in strict mode, if delete is used on a direct reference to a variable, a function argument or a function name, it will throw a SyntaxError.
>Any variable defined with var is marked as non-configurable. In the following example, salary is non-configurable and cannot be deleted. In non-strict mode, the delete operation will return false.


delete只是用来直接删除对象（可删除的）属性，当breed属性是Dogger的最后引用者，对这个属性执行delete操作，这个为引用的对象就可以被垃圾回收了，不要看成一个释放内存的工具，而是删除属性的操作，仅此。
##enumerable
当且仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false。
属性特性 enumerable 定义了对象的属性是否可以在 **for...in 循环和 Object.keys( ) **中被枚举。

放上MDN的代码片段：

> var o = {};
>Object.defineProperty(o, "a", { value : 1, enumerable:true });
>Object.defineProperty(o, "b", { value : 2, enumerable:false });
>Object.defineProperty(o, "c", { value : 3 }); // enumerable defaults to false
>o.d = 4; // 如果使用直接赋值的方式创建对象的属性，则这个属性的enumerable为true

>for (var i in o) {    
>  console.log(i);  
>}
>// 打印 'a' 和 'd' (in undefined order)

>Object.keys(o); // ["a", "d"]

>o.propertyIsEnumerable('a'); // true
>o.propertyIsEnumerable('b'); // false
>o.propertyIsEnumerable('c'); // false

## [ [ Get ] ]和[ [ Put ] ]
### [ [ Get ] ]

```
Dogger.breed
```

如上，对一个对象进行访问时有一个很重要的细节。Dogger.breed是一次属性访问，但并不是仅仅在Dogger中查找breed，其实看起来更像是在语言规范中执行了Dogger的[ [ Get ] ]操作，看上去像[ [ Get ] ]( )。
1. 在对象中查找有没有这个属性。
2. 在该对象的原型链上查找有没有这个属性。
3. 都没找到返回undefined（**注意：如果那个属性值恰好为undefined时，虽然返回值一样，但是底层发生的事是不一样的**）

### [ [ Put ] ]
[ [ Get ] ]对应[ [ Put ] ]操作，一旦给对象属性赋值就触发设置和创建这个属性发生的事情是这样的：
1. 首先确定是否存在这个属性。breed是否存在
2. 存在，是否是setter。是setter就调用setter，是否是setter来给breed赋值
3. writable是否为false，false则无效。breed的writable是否为false
4. 设置值为该属性的值

## Getter和Setter
在《 javascript高级程序设计 》中成为访问器属性，也称为访问描述符，getter和setter是两个隐形的函数，getter为读取属性值的函数，setter为设置属性值的函数，在访问这个阶段我们关注的是四个属性：
1. set
2. get
3. configurable
4. enumerable

这时候我们用Dogger的例子来了解一下这些特性

```
var Dogger = {
    get breed() {
        return  '柴犬'
    }
}
Object.defineProperty(
  Dogger
  "breed-type",
  {
    get: function() {
      return this.breed + '品种'
    }
  }
)
Dogger.breed // '柴犬'
Dogger.breed-type // '柴犬品种'

```
没毛病，不管是隐式调用还是显式确实能够让我们定义属性，自动调用隐藏函数，返回值为属性访问的返回值
[](http://xurenjie.cn:3000/img/dog/dog3.png)
如果这时候，我们想用赋值操作给Dogger改变属性会怎么样？

```
Dogger.breed = '哈士奇'

Dogger.breed // '柴犬'

```

由于只定义了breed的getter，所以对它的值进行设置时**set操作会忽略赋值操作**（也不会报错）。其实就算定义了setter，自定义
的getter还是只会返回getter设置的值。

因此你去改变属性的值时，你还需要定义一个setter，通常来说，他们是成双成对的。不写严格模式会报错。

setter其实就是我们最常用的赋值操作
```
var Dogger = {
    get breed() {
        return  '柴犬'
    }
    set breed(val) {
        this._breed_ = val
    }
}

Dogger.breed = '哈士奇'
Dogger.breed // '哈士奇'
```
这样一来，赋值操作就可以改变啦！我们把赋值操作存储给新建的_breed_ ，只是一种惯例，通过setter可以改变对变量访问值的处理规则。

### 如果不用_breed_，setter/getter的调用执行时机
```
class Dogger {
    constructor (name, breed) {
        this.name = name;
        this.breed = breed;
    }
    set breed (breed) {
        console.log("setter");
        this.breed = breed;
    }
    get breed () {
        console.log("getter");
        return this.breed;
    }
}

var dogger = new Dogger("忠犬八公", '柴犬');
```
1. 代码报错了！！！这是因为，在构造函数中执行this.breed = breed的时候，就会去调用set breed，在set breed方法中，我们又执行this.breed = breed，进行无限递归，最后导致**栈溢出(RangeError)**。
2. 因此，原来只要this.breed中的属性名和set breed/get breed后面的breed一致，对this.breed就会调用setter/getter，也就是说setter/getter是hook函数，而真实的存储变量是_breed_，我们可以在代码中直接获取它。
# 参考
> 1. 你不知道的javascript
> 2. javascript高级程序设计
