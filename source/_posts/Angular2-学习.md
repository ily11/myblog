---
title: Angular2 学习
date: 2017-09-19 11:40:41
tags: Angular
categories: js
---
## TypeScript
### 基本类型
在TypeScript中主要有以下基本数据类型：
* 布尔类型（boolean）
* 数字类型（number）
* 字符串类型（string）
* 数组类型（array）
* 元组类型（tuple）
* 枚举类型（enum）
* 任意值类型（any）
* null和undefined
* void类型
* never类型
其中元组、枚举、任意值、void类型和never类型是TypeScript有别于JavaScript的特有类型。

#### 元组类型
用来表示已知元素数量和类型的数组，各元素的类型不必相同。示例代码如下：
```
let x:[string,number];
x = ['Angular',25]; //运行正确
x = [25,'Angular']; //运行出错
```

#### never类型
是其他类型（包括null和undefined）的子类型，代表从不会出现的值。这意味着声明为never类型的变量只能被never类型所赋值。
### 解构
所谓解构，就是将声明的一组变量与相同结构的数组或者对象的元素数值一一对应，并将变量相对应元素进行赋值。解构可以帮助开发者非常容易地实现多返回值的场景。

#### 数组解构
```
let input = [1,2];
let [first,second] = input;
console.log(first); //相当于input[0]:1
console.log(second); //相当于input[1]:2
```
还可以在数组解构中使用rest参数语法（形式为“...变量名”）创建一个剩余变量列表，“...”三个连续小数点表示展开操作符，用于创建可变长的参数列表，实例代码如下：
```
let [first,...rest] = [1,2,3,4];
console.log(first); //输出1
console.log(rest);  //输出[2,3,4]
```
#### 对象解构
对象解构有趣的地方是一些原本需要多行编写的代码，用对象解构的方式编写一行代码就能完成，代码很简洁，可读性强：
```
let test = {x:0,y:10,width:15,height:20};
let {x,y,width,height} = test;
console.log(x,y,width,height); //输出0，10，15，20
```
### 函数
#### 可选参数
在TypeScript里，被调函数的每个参数都是必传的：
```
function max(x:number,y:number):number{
  return x>y?x:y;
}

let result1 = max(2); //报错
let result2 = max(2,4); // 正确
let result3 = max(2,4,7); // 报错
```
但是经常会遇到要根据实际需要来决定是否传入某个参数的情况，这时候可在参数名旁边加上?来使其变为可选参数：
```
function max(x:number,y?:number):number{
  if(y){
    return x>y?x:y;
  }else{
    return x;
  }
}

let result1 = max(2); //正确
let result2 = max(2,4); // 正确
let result3 = max(2,4,7); // 报错
```
<font color="red">**注：可选参数必须位于必选参数的后面**</font>

#### 默认参数
TypeScript还支持初始化默认参数。如果函数的某个参数设置了默认值，当该函数被调用的时候，如果没有给这个参数传值或者传的值为undefined时，这个参数的值就是设置的默认值：
```
function max(x:number,y=4):number{
  return x>y?x:y;
}

let result1 = max(2); //正确
let result2 = max(2,4); // 正确
let result3 = max(2,4,7); // 报错
let result4 = max(2,undefined);  //正确
```
带默认值的参数不必放在必选参数的后面，但**如果默认值参数放在了必选参数的前面，用户必须显式地传入undefined:**
```
function max(x=2,y:number):number{
  return x>y?x:y;
}

let result1 = max(2); //报错
let result2 = max(2,4); // 正确
let result3 = max(2,4,7); // 报错
let result4 = max(undefined,4); // 正确
```
#### 剩余参数
上面说的可选参数、必选参数和默认参数只能表示某一个参数，当同时需要操作多个参数的时候，就需要用到TypeScript里的剩余参数。在TypeScript里，所有的可选参数都可以放到一个变量里，示例代码如下：
```
function sum(x:number,...restOfNumber:number[]):number{
  let result = x;
  for(let i=0;i<restOfNumber.length;i++){
    result += restOfNumber[i];
  }
  return result;
}
let result = sum(1,2,3,4,5);
console.log(result); //输出15
```
<font color="red">**注：剩余参数可以理解为个数不限的可选参数，即剩余参数包含的参数个数可以为零到多个**</font>
## 组件
### 组件生命周期
组件的生命周期由angular内部管理，从组件的创建、渲染，到数据变动时间的触发，再到组件从DOM移除，Angular都提供来一系列钩子，以下是组件常用的生命周期钩子方法，Angular会按以下的顺序依次调用：
* ngOnChanges
* ngOnInit
* ngDoCheck
* ngAfterContentInit
* ngAfterContentChecked
* ngAfterViewInit
* ngAfterViewChecked
* ngOnDestroy

#### ngOnChanges
它是用来响应组件输入值发生变化时触发的事件。该方法接收一个SimpleChanges对象，包括当前值和变化前的值。该方法在ngOnInit之前，或者当数据绑定输入属性的值发生变化时触发。

只要在组件里定义来ngOnChanges方法，在输入数据发生变化时该方法就会被自动调用，这里的“输入数据”指的是通过@Input装饰器显式指定的那些变量。
#### ngOnInit
ngOnInit钩子用于数据绑定输入属性之后初始化组件。在组件中，经常会使用ngOnInit获取数据。使用ngOnInit有以下两个原因：
* 组件构造后不久就要进行复杂的初始化
* 需要在输入属性设置完成后才构造组件

在第一轮 ngOnChanges 完成之后调用。 ( 译注：也就是说当每个输入属性的值都被触发了一次 ngOnChanges 之后才会调用 ngOnInit ，此时所有输入属性都已经有了正确的初始绑定值 )
#### ngDoCheck
用于变化监测，该方法会在每次变化监测发生时被调用。

绝大多数情况下，ngDoCheck和ngOnChanges不应该一起使用。ngOnChanges能做的事情，ngDoCheck也能做到，而且ngDoCheck监测的粒度更小，可以完成更灵活的变化监测逻辑。
#### ngAfterContentInit
在组件中使用<ng-content>将外部内容嵌入到组件视图后就会调用ngAfterContentInit，它在第一次ngDoCheck执行后调用，且只执行一次。当把内容投影进组件之后调用。
#### ngAfterContentChecked
在组件中使用<ng-content>自定义内容的情况下，Angular在这些外部内容嵌入到组件视图后，或者每次变化监测的时候都会调用ngAfterContentChecked。每次完成被投影组件内容的变更检测之后调用。
#### ngAfterViewInit
初始化完组件视图及其子视图之后调用。

会在Angular创建了组件的视图及其子视图后被调用。
#### ngAfterViewChecked
每次做完组件视图和子视图的变更检测之后调用。

在Angular创建了组件的视图及其子组件视图之后被调用一次，并且在每次子组件变化监测时也会被调用。
#### ngOnDestroy
当 Angular 每次销毁指令 / 组件之前调用。
