---
title: jQuery样式
layout: post
categories: Front End
tags: jQuery
---
# 
## jQuery对象转化成DOM对象
jQuery库本质上还是JavaScript代码，它只是对JavaScript语言进行包装处理，为的是提供更好更方便快捷的DOM处理与开发中经常使用的功能。我们使用jQuery的同时也能混合JavaScript原生代码一起使用。在很多场景中，我们需要jQuery与DOM能够相互的转换，它们都是可以操作的DOM元素，jQuery是一个类数组对象，而DOM对象就是一个单独的DOM元素。
如何把jQuery对象转成DOM对象？jQuery对象自身提供一个.get() 方法允许我们直接访问jQuery对象中相关的DOM节点，get方法中提供一个元素的索引：
```JavaScript
<script>
var $div = $('div') //jQuery对象
var div = $div.get(0) //通过get方法，转化成DOM对象
div.style.color = 'red' //操作dom对象的属性
var $div = $('div'); //jQuery对象
var div = $div.get(0);
div.style.color = 'red'; //操作dom对象的属性
</script>
```
## DOM对象转化成jQuery对象
如果传递给$(DOM)函数的参数是一个DOM对象，jQuery方法会把这个DOM对象给包装成一个新的jQuery对象
通过$(dom)方法将普通的dom对象加工成jQuery对象之后，我们就可以调用jQuery的方法了
```JavaScript
var div = document.getElementsByTagName('div'); //dom对象
var $div = $(div); //jQuery对象
var $first = $div.first(); //找到第一个div元素
$first.css('color', 'red'); //给第一个元素设置颜色
```
## jQuery层级选择器
**子元素 后代元素 兄弟元素 相邻元素**
![层级选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaINLZia4Tm372AwGHicibgKedXC6PBEuW6OwibdQXRjqf8OdxNZTjtBL3DBQ/0?wx_fmt=png "层级选择器")

- 层级选择器都有一个参考节点
- 后代选择器包含子选择器的选择的内容
- 一般兄弟选择器包含相邻兄弟选择的内容
- 相邻兄弟选择器和一般兄弟选择器所选择到的元素，必须在同一个父元素下
## jQuery基本选择器
![基本选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIURPRF5QDZoDaqtn3VZNGsYN98IDJn1X1ibv7urpu4DNeXAhXJrsMjQg/0?wx_fmt=png "基本选择器")
1. eq(), :lt(), :gt(), :even, :odd 用来筛选他们前面的匹配表达式的集合元素，根据之前匹配的元素在进一步筛选，注意jQuery合集都是从0开始索引
2. gt是一个段落筛选，从指定索引的下一个开始，gt(1) 实际从2开始

## jQuery内容选择器
![内容选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIMeKCqmS9tcmjdp9zBbStB2UmZzIicqxKia8IicAhK61D2mL4QdT6sd1uQ/0?wx_fmt=png "内容选择器")
1. :contains与:has都有查找的意思，但是contains查找包含“指定文本”的元素，has查找包含“指定元素”的元素
如果:contains匹配的文本包含在元素的子元素中，同样认为是符合条件的。
2. :parent与:empty是相反的，两者所涉及的子元素，包括文本节点

## jQuery可见性选择器
![可见性选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIf7nFI7R1gUozxK96hpM3WxK7nQmRVVZV4Jcpna7BDwd9TyiavSAy7Ig/0?wx_fmt=png "可见性选择器")
>:hidden选择器，不仅仅包含样式是display="none"的元素，还包括隐藏表单、visibility等等

我们有几种方式可以隐藏一个元素：
- CSS display的值是none。
- type="hidden"的表单元素。
- 宽度和高度都显式设置为0。
- 一个祖先元素是隐藏的，该元素不在页面上显示
- CSS visibility的值是hidden
- CSS opacity的指是0

## jQuery属性选择器
![属性选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIJpnUd1DJobicT0kxJdjORdfPcpp82w2uA7JXIibmO0BZ4KJcatcAHg2w/0?wx_fmt=png "属性选择器")
> [attr="value"]能帮我们定位不同类型的元素，特别是表单form元素的操作，比如说input[type="text"],input[type="checkbox"]等
> [attr*="value"]能在网站中帮助我们匹配不同类型的文件

## jQuery子元素选择器
![子元素选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIo6w00xdvZold3iconBt314hicliayoq0VgKiau2Hcickib1VbAQF7kd16buw/0?wx_fmt=png "子元素选择器")
1. first只匹配一个单独的元素，但是:first-child选择器可以匹配多个：即为每个父级元素匹配第一个子元素。这相当于:nth-child(1)
2. :last 只匹配一个单独的元素， :last-child 选择器可以匹配多个元素：即，为每个父级元素匹配最后一个子元素
3. 如果子元素只有一个的话，:first-child与:last-child是同一个
4. :only-child匹配某个元素是父元素中唯一的子元素，就是说当前子元素是父元素中唯一的元素，则匹配
5. jQuery实现:nth-child(n)是严格来自CSS规范，所以n值是“索引”，也就是说，从1开始计数，:nth-child(index)从1开始的，而eq(index)是从0开始的
6. nth-child(n) 与 :nth-last-child(n) 的区别前者是从前往后计算，后者从后往前计算，也就是倒数第n个

## jQuery表单对象属性筛选选择器
![表单对象属性选择器](https://mmbiz.qpic.cn/mmbiz_png/9kOHib6iaUsPzPLoHoZBA9YK2ZAOrbicBaIZhvVvXZJSqPJnyiaDO6adOvlHruDx4SiaH8TeCS0cIKWQWUicAvFpjSgA/0?wx_fmt=png "表单对象属性选择器")
1. 选择器适用于复选框和单选框，对于下拉框元素, 使用 :selected 选择器
2. 在某些浏览器中，选择器:checked可能会错误选取到<option>元素，所以保险起见换用选择器input:checked，确保只会选取<input>元素

## jQuery this选择器
this是JavaScript中的关键字，指的是当前的上下文对象，简单的说就是方法/属性的所有者
下面例子中，Sylvain是一个对象，拥有name属性与getName方法,在getName中this指向了所属的对象Sylvain
```JavaScript
var Sylvain = {
    name:"SylvainZhao",
    getName:function(){
        //this,就是Sylvain对象
        return this.name;
    }
}
```
Sylvain.getName(); //SylvainZhao
当然在JavaScript中this是动态的，也就是说这个上下文对象都是可以被动态改变的(可以通过call,apply等方法)。同样的在DOM中this就是指向了这个html元素对象，因为this就是DOM元素本身的一个引用。
假如给页面一个P元素绑定一个事件:
```JavaScript
p.addEventListener('click',function(){
    //this === p
    //以下两者的修改都是等价的
    this.style.color = "red";
    p.style.color = "red";
},false);
```
通过addEventListener绑定的事件回调中，this指向的是当前的dom对象，所以再次修改这样对象的样式，只需要通过this获取到引用即可
```JavaScript
this.style.color = "red"
``` 
但是这样的操作其实还是很不方便的，这里面就要涉及一大堆的样式兼容，如果通过jQuery处理就会简单多了，我们只需要把this加工成jQuery对象。
换成jQuery的做法：
```JavaScript
$('p').click(function(){
    $(this).css('color','red')
})
``` 
通过把$()方法传入当前的元素对象的引用this，把这个this加工成jQuery对象，我们就可以用jQuery提供的快捷方法直接处理样式了