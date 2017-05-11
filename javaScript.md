# JavaScript

## 栈 （栈内存）
> `栈-stack：为自动分配的内存空间，它由系统自动释放`<br>

![stack](img/stack.png)<br>
 栈是一种特殊的线性表，限定删除和插入数据元素只能7在一端进行。<br>
 如球盒一样,第一个放入的球在栈底,最后一个球放入的在栈顶,如需拿到第一个放入的求,只能从栈顶一个一个拿出到达栈底。<br>
 栈空间：<b>先进后出、后进先出——Last in Fist out (LIFO)线性表</b>
 
 
## 堆 
> `堆-heap：则是动态分配的内存，大小不定也不会自动释放.`<br>

 如书架，需要取书时，只需知道书的名字就可以直接拿到，无需像球盒一样从最顶部拿出取到需要的球,如json中取数据只要知道`键 key`就可以拿到，顺序并不影响我们，只关心书名。
 

## 浅拷贝
>在定义一个数组或对象时，变量存放的<b>只是放在堆内存数据的一个地址(指针)</b><br>
当拷贝,操作引用类型时,这时候传递只是地址，子对象访问属性时，会根据地址回到父对象指向的堆内存中，父子对象发送了关联,父子指向的同一内存空间.如需让父子不存在关联可通过基础类型来进行赋值。
```javascript
var a = {
    key1:"11111"
}
function Copy(p) {
    var c = {};
    for (var i in p) { 
        c[i] = p[i];
    }
    return c;
}
a.key2 = ['aa','bb'];
var b = Copy(a);
b.key3 = '33333';
alert(b.key1);     //1111111
alert(b.key3);    //33333
alert(a.key3);    //undefined
```

## 深拷贝
>在浅拷贝中如果`b.key2.push('zz')`后a、b的key2值还是产生关联。<br>
解决key2中关联可用递归来解决这个问题
```
console.log(b.key2) //[aa,bb,zz]
console.log(a.key2) //[aa,bb,zz]
```
```javascript
function Copy(p, c) {
var c = c || {};
    for (var i in p) {
        if (typeof p[i] === 'object') {
            c[i] = (p[i].constructor === Array) ? [] : {};
            Copy(p[i], c[i]);
        } else {
            c[i] = p[i];
        }
    }
    return c;
}    
a.key2 = ['aa','bb'];
var b={};
b = Copy(a,b);	   
b.key2.push("cc");
alert(b.key2);    //aa bb cc
alert(a.key2);    //aa bb

```



## 执行上下文(call stack)

>执行代码时都进入执行上下文,他会形成一个作用域。会以堆栈的方式处理它们。栈底永远都是全局上下文，栈顶就是当前正在执行的上下文

```javascript
var name = 'zChange';
function changeName(){
    var oldname = "zz";

    function helloword(){
        var newname = oldname;
        oldname = name;
        name = oldname;
    }
    helloword();
}
changeName();
```
>函数中，遇到return能直接终止可执行代码的执行，因此会直接将当前上下文弹出栈。

```
1.全局上下文入栈 
2.changename入栈 遇到changename() 激活changename 创建他自己的上下文.
3.helloword入栈 遇到helloword() 激活helloword 创建他自己的上下文.

4.helloword出栈 在helloword没有遇到其他能生成上下文的情况，因此这段代码执行完毕，helloword弹出
5.changename出栈 继续执行changename中可执行的代码，也没遇到能生成上下文的情况，这段代码执行完毕，changename弹出
6.就只剩下全局全局上下文了，全局上下文在浏览器窗口关闭后出栈。
```
<b>每次某个函数被调用，就会有新的执行上下文创建，即使是调用自身函数</b>
![call_stack](img/call_stack.png)<b>

## 执行上下文生命周期
>创建阶段：创建变量对象、建立作用域、this的指向z

>代码执行阶段：变量赋值、函数引用、执行其他代码

<b>知识点：1.变量对象 2.作用域链 3.this</b>

![life_cycle](img/life_cycle.png)<br>

### 变量对象
创建变量对象=><br>
1.建立argument对象<br>

2.检查上下文的函数声明,function关键字，在变量中以函数名建立个属性，属性值指向该函数所在的内存空间,如果函数属性已经存在，那么这个函数会被新的覆盖<br>

3.检查上下文中的变量声明，找到一个,就在变量对象中建立个属性，值未undefined,如果属性已经存在,为了防止同名函数被修改成undefined,则会跳过。原属性不会覆盖。<br>
![vo](img/vo.png)<br>

```javascript
function test() {
    console.log(foo);
    console.log(bar);

    var foo = 'Hello';
    console.log(foo);
    var bar = function () {
        return 'world';
    }

    function foo() {
        return 'hello';
    }
}

test();
//funtion (){return 'hello'}
//undefined
//Hello
```
> 可理解为
```javascript
function test() {
    function foo() {
        return 'hello';
    }
    console.log(foo);
    var bar;
    console.log(bar);
    var foo = 'Hello';
    console.log(foo);

    var bar = function () {
        return 'world';
    }
}
```
```javascript
// 创建阶段
// VO 为 Variable Object的缩写，即变量对象
VO = {
    arguments: {...},
    foo: <foo reference>,
    bar: undefined
}
// 这里有一个需要注意的地方，因为var声明的变量当遇到同名的属性时，会跳过而不会覆盖

// 执行阶段
// AO 为 Active Object 活动对象
VO -> AO  //变量对象转向活动对象
VO = {
    arguments: {...},
    foo: 'Hello',
    bar: <bar reference>
}
```


### 作用域or作用域链
#### 作用域：<b>作用域就是变量与函数可访问的范围，即作用域控制着变量与函数的可见性和生命周期。</b>
或者可说为是一套规则，这规则用来管理引擎如何在当前作用域以及嵌套子作用域中根据标识符名称进行变量查找。

<b>作用域是在编译阶段确定规则的</b>
作用域可分为`全局作用域`和`局部作用域` 

#### 作用域链：<b>在执行上下文创建时，会创建变量对象的的一个作用域链（scope chain）。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是一个函数，则将其活动对象作为变量对象。</b>
<b>作用域链是在执行上下文创建阶段生成的</b>

```javascript
var a = 20;
function test() {
    var b = a + 10;
    function innerTest() {
        var c = 10;
        return b + c;
    }
    return innerTest();
}
test();

//innerTestEC 执行上下文
innerTestEC = {
    VO: {...},  // 变量对象
    scopeChain: [VO(innerTest), VO(test), VO(global)], // 作用域链
    this: {}
}

```
![scope](img/scope.png)<br>
>例子中，全局、函数test、函数innerTest的执行上下文先后被创建

## 垃圾回收
>垃圾回收机制：函数的执行上下文,在执行完毕后，生命周期结束，该函数执行的上下文就会失去引用，其占用的内存空间也会被垃圾回收器释放，也就是说当一个值，在内存中失去引用，垃圾回收器将其回收释放

## 闭包
>闭包：有权访问另一个函数作用域中的变量的函数。

<b>闭包的存在就会阻止垃圾回收</b>
```javascript
var fn = null;
function foo() {
    var a = 2;
    function innnerFoo() { 
        console.log(a);
    }
    fn = innnerFoo; // 将 innnerFoo的引用，赋值给全局变量中的fn
}

function bar() {
    fn(); // 此处的保留的innerFoo的引用
}

foo();
bar();

innnerFoo = {
    VO: {...},  // 变量对象
    scopeChain: [VO(innnerFoo), VO(foo), VO(global)], // 作用域链
    this: {}
}

// 通过fn=innnerFoo 将innnerFoo的作用域链保存到全局变量fn中。
// 所以在后面bar()时，fn照样能访问foo中的局部变量a
```
![innerfoo_Scope](img/innerfoo_Scope.png)<br>
<b>所以通过闭包,我们可以在其他执行上下文中，访问到函数内部变量</b>就是在bar执行上下文中能访问到foo的a变量.

>虽然闭包被保存到全局变量中，但是作用域链并不会发生改变,在闭包中能访问到的变量,仍然还是作用域链上能查到的变量

```javascript
var fn = null;
function foo() {
    var a = 2;
    function innnerFoo() { 
        console.log(c);//这里会报错。
        console.log(a);
    }
    fn = innnerFoo; 
}
function bar() {
    var c = 123;
    fn();
    //此时innnerFoo的作用域链是没有发生变化的。所以访问不到c变量;
}
foo();
bar();
```
![error_Scope](img/error_Scope.png)<br>


## this
>this的指向是在调用时确定的。

>在一个函数上下文中，this由调用者提供，由调用函数的方式来决定。如果调用者函数，被某一个对象所拥有，那么该函数在调用时，内部的this指向该对象。如果函数独立调用，那么该函数内部的this，则指向undefined。但是在非严格模式中，当this指向undefined时，它会被自动指向全局对象。

### 作为方法调用
```javascript
//例子1
var name="XL";
var person={
    name:"xl",
    showName:function(){
        console.log(this.name);
    }
}
person.showName();//xl  --因为person去调用showName所以this指向person

var showNameA=person.showName;
showNameA();   //XL --因为是showNameA去调用showName的,showNameA是在window下 所以this指向window 

//例子2
var personA={
    name:'XL',
    showNameA:function(){
        console.log(this.name);
    }
}

var personB={
    name:'xl',
    showNameB:personA.showNameA
}
personB.showNameB();
//xl --因为showNameA是在personB中进行调用的所以this指向的是personB

```


### 作为构造函数调用
```javascript
var name ="XL"
function  Person(name){
    this.name='xl';
}
var personA=Person();
// console.log(window.name);//xl
// console.log(personA.name)//报错
//当personA去调用Person()指向的是window所以this.name就是window.name，重新赋值了name=xl而不是XL

var personB=new Person();
console.log(personB.name);//xl
//new操作符会实例化对象。

```
### new操作符
>先创建一个空对象o，将o的原型链继承fn的原型
>用`call`或`apply`把fn的this重新指向为o,即完成o.name=name操作

```
new操作符调用构造函数会经历4个阶段
1.创建一个新对象。
2.将构造函数的this指向这个新对象
3.指向构造函数的代码，为这个对象添加属性，方法。
4.返回新对象
```

```javascript
//模拟new操作符
function newperson(fn,arg){
    var o = {};
    o.__proto__=fn.prototype
    fn.call(o,arg);
    return o;
}
var name = "XL"
function  Person(name){
    this.name = 'xl';
}
var p = newperson(Person);
console.log(p.name);//xl
```

### call/apply方法
>用处：主要用来改变`this`的指向<br>
>区别：`call`是一个一个的传递，`apply`是以数组形式传递

```javascript
 var name = 'XL';
  var person = {
      name:"xl",
      changeName:function(){
          console.log(this.name)
      }
  }
 var p = person.changeName;
 p();//XL  全局window下
 p.call(person)//xl  this重新指向为person
```


>超时调用的代码`setTimeout`和`setInnerval`都是在`window`下运行。非严格模式指向的是window，严格模式下指向undefined；
```javascript
function newbind (fn,obj){
    return fn.apply(obj,arguments);
}

var a = 10;
var obj = {
    a: 20,
    getA: function() {
        setTimeout(newbind(function() {
            console.log(this.a)
        },this), 1000)
    }
}

obj.getA();//10

//newbind通过apply吧this指向obj。输出10;
```

通过apply\call实现继承
```javascript
var father = function(work,sex){
    this.work = work;
    this.sex = sex;
    this.hello = function(){
        console.log(this.work,this.sex,"hello");
    }
}
var son = function(name,sex,height){
    father.apply(this,arguments);
    this.height=height;
}

son.prototype.message = function(){
    console.log(this.work,this.sex,this.height);
}

var son = new son('zz','man',180);
son.message();//zz man 180
son.hello();//zz man hello

```
<b>借助apply方法,将父及的构造函数指向一次，相当于将两father中的代码,在son中复制一份,其中this的指向为son中new出来的实例对象</b>


## 函数参数传递方式:按值传递
>函数参数在进入函数时，实际是被保存在了函数的变量对象中，因此，这个时候发生了一次复制。

传入参数是按值传递，而不是引用传递，只不过传递一个引用类型时，传递的是引用类型保存在变量中的引用（指针）而已。
```javascript
var person = {
    name:'aa'
}
function setName(obj){
    obj = {};//将传入的引用指向另外一个值
    obj.name = 'bb';
}
setName(person);
person.name;//aa
```
如果是引用传递的话name会被改变成bb，在这里将obj的引用执行了一个{}所以obj.name='bb' 是不会影响到person.name的
