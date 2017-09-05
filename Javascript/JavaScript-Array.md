# JavaScript之数组
---

# 数组去重
       var arr = [3, 1, 2, 2, 3, 4, 4, 5, 4, 5, 6, 8, 6, 7,];


### 通过indexOf判断
```javascript
Array.prototype.unique = function(){
    var arr = [];
    for(let i = 0; i < this.length;i++){
        if(arr.indexOf(this[i])== -1){
            arr.push(this[i]);
        }
    }
    return arr;
}
log(arr.unique());
```
>**es5 filter indexOf判断
判断index和arr.indexOf的位置不一致的就是重复的。一致的就是我需要的。**
```javascript
Array.prototype.unique = function () {
    return this.filter((item, index) => {
        log('__值:' + item + '__所在的位置:' + index + '__arr.indexOf的位置:' + this.indexOf(item))
        return this.indexOf(item) === index;
    })
}
log(arr.unique());
```

```javascript
Array.prototype.unique = function() {
    var arr = [];
    this.forEach((item)=>{
        if(arr.indexOf(item) === -1){
            arr.push(item);
        }
    })
    return arr;
}
log(arr.unique());
```

### 通过key值去判断
> **使用key值来去重**
```javascript
Array.prototype.unique = function() {
    var arr = [];
    var len = this.length;
    var key = {};
    for(let i =0; i<len;i++){
        if(!key[this[i]]){
            key[this[i]] = 1;
            arr.push(this[i]);
        }
    }
    return arr;
}
log(arr.unique());
```

> **使用key值来去重，使用es6    
通过new Map，set值，判断存在值就不添加；
可用map.has,或get（用get，set方法必须穿值map.set(this[i],1)）**
```javascript
Array.prototype.unique = function () {
    var map = new Map();
    var arr = [];
    var len = this.length;

    for (let i = 0; i < len; i++) {
        if(!map.has(this[i])){
            map.set(this[i]);
            arr.push(this[i])
        }
    }
    log(map.keys())
    return arr;
}
log(arr.unique());
```

## 排序对比前后值
> **先排序，然后比较前一个值和后一个值是否相同**
```javascript
Array.prototype.unique = function () {
    var sort = this.sort();
    return sort.filter((item,i,context)=>{
        console.log(item,context[i+1],context)
        return item !== context[i+1];
    })
}
log(arr.unique());
```

>**排序后通过reduce对比前后值，reduce会省略最后一个，所以最后push最后一个值**
```javascript
Array.prototype.unique = function () {
    var sort = this.sort();
    var arr = [];
    console.log(sort)
    sort.reduce((iA,iB)=>{
        console.log(iA,iB);
        if(iA !== iB){
            arr.push(iA)
        }
        return iB
    })
    arr.push(this[this.length-1])
    return arr;
}
log(arr.unique());
```
>**排序后比较相邻的值**
```javascript
Array.prototype.unique = function () {
    var arr = [];
    this.sort().forEach((item, index) => {
        if (item !== this[index + 1]) {
            arr.push(item);
        }
    })
    return arr;
}
log(arr.unique());
```

>**先排序，将this的最后一个元素复制给arr,然后对比原数组的i和arr的最后一个对比是否相同**
```javascript
Array.prototype.unique = function () {
    this.sort();
    var arr = [this[0]];
    for(let i =0; i<this.length;i++){
        console.log(this[i],arr[arr.length - 1]);
        if(this[i] !== arr[arr.length - 1]){
            arr.push(this[i])
        }
    }
    return arr;
}
log(arr.unique());
```

## es6 
> **通过includes判断是否存在**
```javascript
Array.prototype.unique = function() {
    var arr = [];
    this.forEach((item)=>{
        if(!arr.includes(item)){
            arr.push(item);
        }
    })
    return arr;
}
log(arr.unique());
```
> **new Set Array.from [...]**
```javascript
Array.prototype.unique =  function() {
    return [...(new Set(this))]
    return Array.from(new Set(this));
}
log(arr.unique());
```

# 浅拷贝数组
```
 var arr = [1,2,3,4];
 //concat
 var Newarr1 = arr.concat();
 console.log(arr === Newarr1)   

 //slice
 var Newarr2 = arr.slice(0);
 console.log(arr == Newarr2)   

 //apply
 var Newarr3 = []; 
 [].push.apply(Newarr3,arr);
 console.log(arr == Newarr3) 

 //join+split  ！！！数据类型会变成字符串
 var Newarr4 = arr.join().split(',');
 console.log(arr == Newarr4)  
```




