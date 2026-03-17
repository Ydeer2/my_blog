---
cover:
title: html,css,javascript学习总结
date: 2025-06-07
categories:
  - 技术
  - 前端
tags:
  - 笔记
---

# html,css,javascript学习总结

## html知识点

### HTML标签的分类

形式上

1.围堵标签

2.自闭和标签

效果上

1.块级标签

2.行内标签

### 标签的使用事项

嵌套标签：一个标签里面嵌套另一个标签

块级标签课嵌套块级标签

块级标签可以嵌套行内标签

行内标签可以嵌套行内标签

行内标签不可以嵌套块级标签

### 布尔属性

在标签中如果不给属性赋值默认这个属性是布尔属性，

如<a disable><a/>

如果省略属性值的引号，这是disable就是布尔属性的，且为false

### 单引号和双引号

单引号和双引号是可以混用的。

可以相互嵌套，但是同类型的不能嵌套

### HTML中存在特殊字符

具体的要可以去这[HTML特殊符号（字符实体)大全-CSDN博客](https://blog.csdn.net/wxl2454497677/article/details/107608884)看看

### 语义化

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/6d30405b7dc248d3b2d764d2f97dd179.png)

意思就是不要因为它的样式而选择这个标签，而是要因为它的语义来使用它。样式可以有专门css的控制。转而言之它的样式并不那么重要，重要的是它的语义。

```html
<strong
  >语义化 1.可读性强 2.可以被网络检索到
  3.阅读器，会看见strong标签，改变阅读器的读音 4.以加粗的形式展现强调的效果
  <b>只有一个加粗效果</b></strong
>
```

## css知识点

## 各种选择器

### 属性选择器

```html
<style>
    /*匹配所有带有class属性的li标签*/
    li[class]{
        color:red;
    }
    /*匹配所有带有'a'类的li标签*/
    li[class='a']{
		color:red;
    }
    /*匹配所有带有包含'a'类的li标签*/
    li[class~='a']{
		color:red;
    }
    /*匹配所有带有包含'a-?'(如：a-dfs a-c a-a等)类的li标签*/
    li[class|='a']{
		color:red;
    }
<style>
```

### 子字符串选择器

```html
<style>
    /*匹配所有带有以a开头的字符串属性的li标签*/
    li[class^="a"]{
        color:red;
    }
    /*匹配所有以'a'结尾类的li标签*/
    li[class$='a']{
		color:red;
    }
    /*匹配所有任意位置出现'a'类的li标签*/
    li[class*='a']{
		color:red;
    }
    /*匹配包忽略大小写的以'a'开头类的li标签*/
    li[class^='a' i]{
		color:red;
    }


<style>
```

### 伪类选择器

所谓伪类，就是根据某一特定状态来进行分类。

伪类选择器又分为：

- 普通伪类选择器
- 行为伪类选择器

图解：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/38a9d65cff4d4d26bc1464dca6761637.png)

具体查看官网[CSS 伪类](https://www.w3school.com.cn/css/css_pseudo_classes.asp)

### 交集选择器和并集选择器

```html
<style>
    /*交集选择器
    同时是p标签，和cls类的p标签
    */
    p.cls{
		color:red;
    }


<style>
```

### 后代选择器

```html
<style>
  /*一个标签内的子标签，无论是直接还是间接*/
  /*ul标签里面的li标签*/
  ul li {
    color: red;
  }
</style>
```

### 子代选择器

```html
<style>
  /*直接子代选择器，间接的不行*/
  ul > li {
    color: red;
  }
</style>
```

### 兄弟选择器

```html
<style>
  /*如ul相邻的p标签，不重复*/
  ul + p {
    color: red;
  }
  /*与ul相邻的p标签，可
    重复*/
  ul ~ p {
    color: red;
  }
</style>
```

## 样式选择

选择设置标签的样式，有多种方式：

- 行内样式
- 内部样式
- 外部样式

### 行内样式

优缺点很明显：优点：简单直接。缺点：硬编码，拓展性不强。

```html
<p style="color : red;">这是一个段落</p>
```

### 内部样式

在style块里面定义样式，方便拓展性强，但是只能用到当前页面。

```html
<style>
  p {
    color: red;
  }
</style>
```

### 外部样式

在外部css文件里面定义样式，需要用的时候在引入到页面里。

```css
p {
  color: red;
}
```

引入css文件。

```html
<link rel="stylename" href="文件路径" />
```

## 选择器的优先级

id>类>元素

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/f10da5ef569d4232b2e0b497fedde348.png)

0. !important 优先级高于行内
1. 行内样式优先级大于其他
2. 内部样式和外部样式，会根据加载顺序，加载进文档，形成文档流，

​ 分析这个文档流中的这些选择器优先级，（内部和内部和外部引入来的）

​ 这和引入顺序无关。

​ 3.拿到文档流，分析文档流中这些选择器的优先级，

id选择器

属性选择器

类选择器

标签选择器

全局选择器

4.在文档流中，有两个优先级相同的选择器，最后一个会生效。

5.可以做到字符继承，父容器的样式会传递到子容器，一旦子容器有自己的样式，那么会覆盖父容器的样式。

## 组合选择器的优先级

```html
<style>
  /*所谓组合选择器就是多种类型的选择器组合在一起，id 元素 类 */
  /*当两个组合选择器选中同一个标签，那么他们是有优先级的
    优先级的计算是按照id 类 元素的顺序那个计算权重，那个高，优先级高
    */
  /*
    权重：1-1-1
    */
  div #id .cls {
  }
  /*
    权重：2-0-1
    这个权重高
    */
  div #id #ids {
  }
</style>
```

## 元素类型

### 行内元素

只站一行的元素

不能控制高度，但可以控制左右边距。

### 块级元素

新元素一般都是从新行开始，高度左右都可以控制。

### 行内块级

具有行内和块级两种特性，一般都是占一行，但也可以设置高度，常见的行内块级标签有，input，img标签。

## 盒子模型

把一个容器之间的关系想象成一个一个的盒子，一个盒子可以套另一个盒子。可以控制盒子的内外边距，控制盒子的位置。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/c0c1745d65944be2a77309d3932ed087.png)

```html
<style>
  p#sb.cls {
    color: red;
  }
  #div1 {
    border: 2px red solid;
  }
  #div2 {
    border: 2px green solid;
    text-align: center;
    margin: 16px 20px 30px 40px;
    padding: 16px;
  }
</style>
<div id="div1">
  <div id="div2">
    <span>span1</span>
    <span>span2</span>
  </div>
</div>
```

## javascript知识点

### 基础语法

JS主要有两大操作：

dom操作：文档操作。

bom操作：游览器操作。

### js的输出

```javascript
console.log("holle world");
```

### js的外部引入

```html
<script src="资源相对路径"></script>
```

### js变量

```javascript
//全局
var a;
a = 10;
//局部
let b;
b = 10;
```

### js数据类型

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/111772e0aa574afc90cd259ba452de55.png)

```javascript
var max = 1 / 0; //无穷大
console.log(max); //Infinity
console.log(typeof max);
```

parseInt()和parseFloat()

```javascript
var s = parseInt("122fds"); //122
var x = parseFloat("234.324fs"); //234.324
```

### 运算符

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/076fe5a4d619454b9e006e3d814a6e58.png)

**注意** ===判断值相同的同时也比较类型是否相同

### 隐式转换

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/ba490aba41b74f40bd1d27dfe5bcdc1e.png)

```javascript
var x = 1 + true;
//x=2;true会隐式转换成1，有意义的会转换成1，其他如false,NULL就是0,NaN undefined特殊 不会转换。
var b = NaN == NaN;
//NaN这个简直就是六亲不认，自己和自己比也是false
```

### 函数

```javascript
function add(a, b) {
  return a + b;
}
var num = add(2, 3);
```

### 函数表达式

```javascript
var x = function (a, b) {
  return a + b;
};
var k = x(a, b);
```

### 函数里面的arguments

arguments表示函数里面的arguments对象。里面封装着参数

arguments.callee表示函数本身，主要用于匿名函数的调用（有名字的压根就不需要）。

```javascript
var factorial = function (n) {
  if (n <= 1) return 1;
  return n * arguments.callee(n - 1);
};
```

### 局部变量的作用范围

var 关键字的局部变量只有在函数里面才算

```javascript
function fn(a, b) {
  var num = 10;
  console.log(num);
}
/*
除了函数num就不存在了
*/
```

### 数组对象

数组的定义，数组里面的类数据可以是任意一种类型。

```javascript
var arr = ["adf", "dfa", 1, 2, "dfs"];
var Arr = new Array(["df", "sdf", 1, 2]);
var arr2 = new Array();
arr2.push("dfsf");
arr2.push("dfsdsf");
```

基本属性

```javascript
var arr = ["adf", "dfa", 1, 2, "dfs"];
arr.length = 10; //可以直接改变数组的长度。
//相当于五个数据，其他位置为空
arr.push(12); //添加到数组末尾
arr.unshift(244); //添加到数据的头部
arr.pop(); //弹出并删除数组末尾的元素
arr.shift(); //弹出并删除数组头部元素
```

### 日期对象

```javascript
var date = new Date();
```

日期对象的设置

```javascript
//直接创建
var date0 = new Date();
// 根据毫秒值，从1970 1月1日 00:00开始。
var date = new Date(Date.now());
var date2 = new Date(0);
// 根据日期属性
var date1 = new Date(2025, 11, 23, 2, 20, 23, 23);
// 根据字符串
var date3 = new Date("October 11 2000 14:12:12");
console.log(date);
console.log(date1);
console.log(date3);
var year = date3.getFullYear();
var month = date3.getMonth();
var day = date3.getDay();
date3.setFullYear(1230);
console.log(date3);
console.log(year);
console.log(month);
console.log(day);
```

### window对象的各种框

```javascript
//警告框
window.alert("dfsfdgd");
//询问框
var key = window.prompt("请输入key");
// alert("key："+key);
//确定框
var jud = window.confirm("你确定");
if (jud) {
  alert("你确定");
} else {
  alert("你不确定");
}
```

### 定时器

```javascript
function print() {
  console.log("执行了一次");
}
setInterval(print, 2000);
```

## dom对象

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/f66141f4e1564dedba4d6f6687f815c3.png)

### 获取元素对象

```javascript
//根据id返回单个元素对象
var ele = document.getElementById("sb");
console.log(ele);
//根据标签名返回元素数组
var ele = document.getElementsByTagName("p");
for (let x = 0; x < ele.length; x++) {
  console.log(ele[x]);
}
//根据class属性返回元素数组
var ele = document.getElementsByClassName("cls");
for (let x = 0; x < ele.length; x++) {
  console.log(ele[x]);
}
//根据name属性来获取元素对象
var ele = document.getElementsByName("one");
for (let x = 0; x < ele.length; x++) {
  console.log(ele[x]);
}
```

### 插入元素

```javascript
var p = document.createElement("p");
var be = document.getElementById("child2");
p.innerText = "去你妈的";
//插入父节点里面
ele.appendChild(p);
//插入父节点中的子节点的某个元素之前
ele.insertBefore(p, be);
//删除子节点
ele.removeChild(be);
//删除子节点的第二种方式
be.parentNode.removeChild(be);
```

### dom表单操作

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/8f75823e7b2548ada669e7ab11819feb.png)

## 事件

### 事件源和事件绑定

```html
<div id="divx">你好</div>
    <input type="button" id="one" value="改变">改变
<script>
    var button = document.getElementById("one");
    button.onclick = function(){
        var ele = document.getElementById("divx");
        ele.innerHTML = "nihao";
    }
```

### 焦点事件

```html
<html>
  <input type="button" id="one" value="改变" />改变 昵称<input
    type="text"
    id="id"
    value="nihao"
  /><span id="fa"></span>
  <html>
    <script>
      var x = document.getElementById("id");
      //聚焦
      x.onfocus = function () {
        var text = document.getElementById("fa");
        text.innerHTML = "请输入长度为6~12的字符";
      };
      //离焦
      x.onblur = function () {
        var ele = document.getElementById("id");
        //正则表达式
        var reg = /^\w{6,12}$/;
        var flag = reg.test(ele.value);
        if (flag) {
          document.getElementById("fa").innerHTML = "验证通过";
        } else {
          document.getElementById("fa").innerHTML = "验证失败";
        }
      };
    </script>
  </html>
</html>
```

### 改变事件

当标签的值改变的时候触发

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/e7d716af20fb46bdb5b4908cc06af5eb.png)

```javascript
city.onchanger = funciont(){
    //改变后，会将改变的信息封装成一个对象
    console.log("选中的城市是"+this.value);
}
```

### 表单事件

可以设置一个提交按钮，去控制表单的提交

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/07b04156e5f1452c831f25ee407e14ac.png)

## let关键字

- 用let声明的变量不能重复声明

- let声明的变量有代码快的概念

- var声明的变量没有定义会是unfined，let同样也是unfined，但是var是可以无视声明顺序的

  而，let不可以无视变量的声明顺序。

## const关键字

const关键字修饰的变量不可以更改，仅限于引用不能更改，数组对象还是可以更改里面的值，自定义类对象也可以改变里面的值，只是这个引用不能更改了。

## 构造方法创建对象

```JavaScript
        // function 对象(param1,param2...){
        //     this.param1 = param1;
        //     this.param1 = param2;
        //     this.方法 = function (){

        //     }
        // }
        function Person(name,age){
            this.name = name;
            this.age = age;
            this.eat = function(){
                console.log(this.name+"在吃东西");
            }
        }
        var person = new Person("zhangsan",18);
        person.eat();
```

## 继承关系的绑定

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.eat = function () {
    console.log(this.name + "在吃东西");
  };
}

function Student(score) {
  this.score = score;
}

var person = new Person("zhangsan", 18);
//设置继承绑定
//所有类的父类是object，这个继承可java一样是有继承链的。会不断往下找属性			和方法。
Student.prototype = person;
var student = new Student(89);
student.eat();
console.log(student.name);
person.eat();
```
