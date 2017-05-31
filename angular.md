# angular4

## 直接使用DOM API获取页面元素
> 不推荐使用 ElementRef.nativeElement.querySelector()
```javascript
 @Component({
  selector: 'my-comp',
  template: `
    <input type="text" />
    <div> Some other content </div>
  `
})
export class MyComp {
  constructor(el: ElementRef) {
    el.nativeElement.querySelector('input').focus();
  }
}
```
>推荐使用@ViewChild
```javascript
@Component({
  selector: 'my-comp',
  template: `
    <input #myInput type="text" />
    <div> Some other content </div>
  `
})
export class MyComp implements AfterViewInit {
  @ViewChild('myInput') input: ElementRef;

  constructor(private renderer: Renderer) {}

  ngAfterViewInit() {
    this.renderer.invokeElementMethod(
        this.input.nativeElement, 'focus');
    }
}
```

@ViewChild() 属性装饰器，还支持设置返回对象的类型，具体使用方式如下：

```javascript
@ViewChild('myInput') myInput1: ElementRef;
@ViewChild('myInput', {read: ViewContainerRef}) myInput2: ViewContainerRef;
//若未设置 read 属性，则默认返回的是 ElementRef 对象实例。
```