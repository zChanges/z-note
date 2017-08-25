# Angular的个人理解（优化及坑）
Tags： angular.js angular

## 双向绑定实现方式(哪几种方式vue，ng)
### 1.发布订阅模式(Publish/Subscribe)
> 发布订阅模式也叫观察者模式，定义了对象一对多的关系，让多个观察者监听同一个对象，当这个对象发生变化时，观察者们就能接受接收到通知。
**现实中**：在淘宝中，多个买家在一家店铺中买鞋子，但是又各部分鞋子暂时缺货，这时候店铺就会叫买家关注我的店铺，等货到了就会一次通知他们。卖家店铺属于发布者，买家属于订阅者，当鞋子到了卖家店铺就会通过旺旺之类的工具发布消息给买家。

在双向绑定中就是，在数据发送变化时，我们发布一个叫`modelUpdate`的事件，当视图发生变化时，发布一个叫`uiUpdate`的事件，后面就是当想要发生什么动作的时候就去订阅这些事件就好了。

### 2.数据劫持（vue）
vue就是通过[Object.defineProperty()][1]来劫持对象属性的setter和getter的操作，在数据变动后去更新视图。
```javascript
var data = {
    name:'aaa'
}
Object.keys(data).forEach(function(key){
    Object.defineProperty(data,key,{
        enumerable:true,
        configurable:true,
        get:function(){
            console.log('get');
        },
        set:function(){
            console.log('监听到数据发生了变化');
            uiUpdate();
        }
    })
});
function uiUpdate(){
    console.log('更新视图……');
}
data.name;
data.name = 'bbb'
//打印了：
//get
//监听到数据发生了变化
//更新视图……
```
通过Object.defineProperty()在结合发布订阅者模式，当数据变化之后就可以通知订阅者执行相应的操作。
> **vue和angular的优势:**
当页面中有上千个绑定是，vue能通过`Object.defineProperty()`知道哪个页面数据发生了改变，然后值渲染着一个地方，而angular不能知道哪一块绑定发生了变化，比较暴力的检查当前作用域下的所有watcher，只要有东西变了，就吧所有的当前作用域里的watcher进行计算，变了就更新；

### 3.脏检查
每个双向绑定的元素都为有个$watch，每个$watch都记录了上一次的值，当发生事件的时候调用$digest()脏检测，对比新值和旧指是否一致，不一致就更新页面，然后再执行一次，直到数据不再变化，至少执行两边，因为第二遍来确认，前一边的变更是不是导致了其他的数据变更。$digest()一次最多执行10遍，超出后就会抛出异常。

像vue通过setter得到数据变更的通知的，但是angular.js脏检测机制并没有这个阶段，所以他只能通过某个事件来调用`$digest()`。
如：
1.DOM事件，ng-click、ng-change……
2.XHR响应事件（$http）
3.浏览器Location变更事件 ( $location )
4.Timer事件( $timeout , $interval )
如果引入了外部框架如jq出发了数据变更，可以通过手动调用`$apply()`或`$digest()`进行手动执行脏检测。

```javascript
function Scope() {
        this.$$watchers = [];//存储组测过的所有监听器。
    }

    //定义$watch方法。把他存储到 $$watchers数组中。
    Scope.prototype.$watch = function(watchFn,listenerFn) {
        var watcher = {
            watchFn: watchFn,
            listenerFn: listenerFn || function() {}
        }
        this.$$watchers.push(watcher);
    }

    //检测并渲染；
    Scope.prototype.$$digestOnce  = function() {
        var self = this;
        var dirty;
        _.forEach(this.$$watchers, function(watch) {
            var newValue = watch.watchFn(self);
            var oldValue = watch.last;
            if(newValue !== oldValue){
                watch.listenerFn(newValue,oldValue,self);
                dirty = true;
            }
            watch.last = newValue;
        }); 
        return  dirty
    };

    //只有当dirty稳定后再执行操作。确保第二次数据变化不会导致上一次数据的变更；
    Scope.prototype.$digest = function() {
        var dirty
        do {
            dirty = this.$$digestOnce();
        } while (dirty);
    };
    
   
    var scope = new Scope();
    scope.firstName = 'joe';
    scope.counter = 0;

    //模拟给数据绑定添加watch
    scope.$watch(
        function(scope){
            return scope.firstName;
        },
        function(newVlaue,oldValue,scope){
            scope.counter++;
        }
    );

    scope.$watch(
        function(scope){
            return scope.counter;
        },
        function(newValue,oldValue,scope){
            scope.counterTow = (newValue === 2);
        }
    );
    scope.$digest();
    scope.firstName = 'Jane';
    scope.$digest();
    
    console.log(scope.counter);
    console.log(scope.counterTow);
    console.log(scope)
```

### digest和apply区别、使用场景
**区别:**
1.`$apply()`可以带参数，可以接受一个函数，就可以把非集成angular的代码写在这个里面进行调用；
`$digest()`,只会触发当前作用域和子作用域的监控，而$apply()会触发作用域数上的所有监控。
**优化**
在调用外部框架的时候采用`$apply()`触发作用域上所有监控，如果用`$digest()`可能会丢失数据变更；
不涉及到向上检测可用`$digest()`，减少脏检查范围。
```javascript
var app = angular.module("test", []);

app.directive("increasea", function() {
    return function (scope, element, attr) {
        element.on("click", function() {
            scope.a++;
            scope.$digest();//父级
        });
    };
});

app.directive("increaseb", function() {
    return function (scope, element, attr) {
        element.on("click", function() {
            scope.b++;
            scope.$digest();// <span ng-bind="a"></span> a值未变
            //采用scope.$apply();即可
            //子级，调用$digest()只会检测increaseb子级，increasea的变更会检测不到
        });
    };
});

app.controller("OuterCtrl", ["$scope", function($scope) {
    $scope.a = 1;

    $scope.$watch("a", function(newVal) {
        console.log("a:" + newVal);
    });

    $scope.$on("test", function(evt) {
        $scope.a++;
    });
}]);

app.controller("InnerCtrl", ["$scope", function($scope) {
    $scope.b = 2;

    $scope.$watch("b", function(newVal) {
        console.log("b:" + newVal);
        $scope.$emit("test", newVal);
    });
}]);
```

```html
<div ng-app="test">
    <div ng-controller="OuterCtrl">
        <div ng-controller="InnerCtrl">
            <button increaseb>increase b</button>
            <span ng-bind="b"></span>
        </div>
        <button increasea>increase a</button>
        <span ng-bind="a"></span>
    </div>
</div> 
```

**合并http请求**
每次调用接口，值返回后angular就会触发脏检查，刚进页面不可能只掉用一个接口，所以可以等几个接口返回后在触发脏值检查，这时候就可以`$httpProvider` 的 `useApplyAsync` 方法，他通过`$rootScope.$applyAsync`吧同一时间（10s左右）的返回值合到一起处理；
```typescript
app.config(function ($httpProvider) {
  $httpProvider.useApplyAsync(true)
})
```

**脏值检查的利弊。**

    <span>{{num}}</span>
```javascript
$scope.nun = 0;
var data = [];
for(var i=0,i<1000,i++){
  data.push({
    index:i;
    select:false;
  });
}

$scope.selectChecked = funtion(flag){
  for(var i=0,i<data.length;i++){
   data[i] = flag;
   $scope.nun++;
  }
}
```
这种情况批量操作DOM下，脏值检测没有任何压力，因为angular会使用DocumentFragment，会先在文档碎片中把DOM结构建好，然后最后整体的添加文档中，DOM变更就会一次完成，性能提高很多，如果用setter就会每次变更触发一次DOM渲染，性能就很低。
Vue就是采用setter的但是他也有这种机制，批量操作后再延迟，然后一次更新。



## angular坑及优化

### 1.触发脏检测
angularjs采用脏检查当然angular也继续使用(**变换检测**-只是换了名字)，只是做了性能的优化，
都是通过某些事件来知道数据可能发生了变化。(click……事件)，angularjs就封装一些内置的服务和指令(ng-click、$http、$timeout)来触发脏检查。
而这些事件都是异步的，异步操作就是可能改变值变化的地方。所在在angular中引入了[zone.js][2](猴子补丁)。zone.js重写了所有异步webapi，然后通知angular数据可能发生变化，需要进行检查。
angularjs在没有zonejs的情况下，在使用第三方时间或者ajax都要用$apply()手动触发下脏检查，使用angular情况下再也不需要手动触发了，可以随意的使用原生对象（setTimeout、setInterval……）

### 2.改善的脏检测
angular优化了angularjs中的脏检测，改为变换检测；
angular的核心是组件化，其实就是angularjs中的`directive + controller`，而变化检测也是在组件中进行的，每个`Component`都对应着一个`changeDetector`，多个`Component`组成一个树状结构的树，那么`changeDetector`同样也是树状结构的树。
而且每次发生变化检测都是在树的顶端开始，从顶部往下，单向数据流；
单项数据有助于渲染过程中获得更好的性能、可预测的变化检测。
单向数据流规则是指数据模型发生变化，执行变换检测；（所以会有一个坑后面会提到）
而在angularjs中是双向数据流，数据流杂乱，可能会运行多次检查，使数据不再发生变化。
angular中不但能手动设置变换检测的策略，还能获取到变换检测对象引用的`ChangeDetectorRef`
具体例子可查看[ChangeDetectorRef API][3]
```typescript
class ChangeDetectorRef {
  markForCheck(): void  
  //检查组件树上所有父子组件
  detach():void 
  //将变更检测从变更检测树中分离。分离的变更检测不会进行变更检查。可以配合detectChanges使用，来实现局部变更检查
  detectChanges(): void
  //检查该组件以及子组件。
  checkNoChanges(): void
  //检查该组件以及子组件，如果存在变化就抛出错误，用于开发阶段
  reattach(): void
  //将分离变换检测对象重新连接到变化检测树中
}
```

```typescript
enum ChangeDetectionStrategy {
  OnPush 
  //当检测到子组件绑定的值没法说变化时，变化检测不会进入到子组件中，只有当数据引用发生变化才会进入
  //一般会在组件树比较庞大的时候使用。
  Default 
  //默认行为（如果不设置则为默认策略） 只要值变化，就全部检查
}
```

**坑及优化**

```typescript
//aDemo(父级)
import { Component } from '@angular/core';

@Component({
  selector: 'aDemo',
  template: `
  {{title}}
  <button type="button" class="btn btn-info" (click)="changeA()">changeA</button>
  <br/>
  <h1 class="h1">bDemo</h1>
  <bDemo [data]="data" [title]="title"></bDemo>
  <button type="button" class="btn btn-info" (click)="changeB()">changeB</button>
  `
})
export class ADemoComponent {

  constructor() { }
  title = 'changeABDemo'
  data: any = { name: 'zzz', id: '1' };
  
  changeA() {
    this.data.name = 'yyy';
    this.data.id = '2';
  }

  changeB() {
    this.data = new Object({ name: 'ccc', id: "333" })
  }

}
```
```typescript
//bDemo(子级)
import { Component, Input, ChangeDetectionStrategy, OnChanges, SimpleChanges, DoCheck } from '@angular/core';
@Component({
  selector: 'bDemo',
  template: `
      {{data.name}}{{data.id}}
      {{title}} 
 `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class BDemoComponent implements DoCheck, OnChanges {

  @Input() data: any;
  @Input() title: any;
  constructor() { }

  ngOnChanges(SimpleChanges) {
    console.log(SimpleChanges);
    // 不论检测策略是哪种，只有当数据引用发生变化才会触发；
  }

  ngDoCheck() {
    console.log(this.data);
    // 在变换检测中进行调用，能解决ngOnChanges下不改变ui的情况
    // 通过改变数据引用就也可以解决ngOnChanges
  }
}
```
![aDemo.gif-419.5kB][4]
**坑**
`ngOnChanges`监听输入对象变化，只有当数据引用发生变化时才会触发，
解决方法：

 1. 改变数据对象的应用如new object()或`immutable.js`;
 2. 通过ngDoCheck()变换检测生命周期函数获取值的变化（弊端会执行多次）
 3. 使用订阅对象（使用后在ngOnDestroy钩子中销毁）

**优化**
在bDemo中可以看到我用到了`changeDetection: ChangeDetectionStrategy.OnPush`；在点击changeA按钮时，子组件bDemo中的ui不会发生变化，当点击changeB按钮时，子组件ui发生了变化；
当设置了`OnPush`策略，变更检测发现`data`数据的引用不会发生变化，就不进入子组件中，在点击changeB按钮时，变换检测发现`data`引用发生变化就继续进入bDemo中进行变换检测；
而当我们去掉`OnPush`策略，就会设为默认`Default`策略，不论值是否改变都进入子组件中进行变换检测，如果组件树比较庞大，就可以采用`OnPush`策略，减少变换检测的频率


# chrysalis-ng
>因前端项目以angular居多(tms也可能转向angular)，项目风格也是统一风格，公用的ui组件居多，觉得可以开发属于公司内部的ui组件库，以便后期项目的快速开发；

业余时间写的一个angular ui组件库；预览地址:[https://zchanges.github.io/chrysalis-ng/](https://zchanges.github.io/chrysalis-ng/)

 - 也相应的写了笔记,可以通过[地址][5]查看；
 - 之前巩固基础也做了些js的笔记[Javascript1][6] or[Javascript2][7]

> **暂时编写了这些常用组件**
 
- [x] sidebar
- [x] navMenu
- [x] pager
- [x] pagination
- [x] tabs 
- [x] badge 
- [x] notification 
- [x] chAutoCompleteDemo
 


  [1]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
  [2]: http://www.cnblogs.com/whitewolf/p/zone-js.html
  [3]: https://angular.io/api/core/ChangeDetectorRef
  [4]: http://static.zybuluo.com/zChange/fts5z0tsaqvpsnv3gdvyuclg/aDemo.gif
  [5]: https://github.com/zChanges/chrysalis-ng#components
  [6]: https://github.com/zChanges/z-note/blob/master/Javascript/javaScript.md
  [7]: https://github.com/zChanges/z-note/blob/master/Javascript/javaScript.1.md
