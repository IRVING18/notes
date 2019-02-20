# gradle 3.0 升级变化

|已弃用配置|新配置|行为|
|:------:  |:------:|:------:|
|compile|implementation|1.可以缩短编译模块的时间。
2.不能传递依赖，比如A依赖B，B依赖C。使用compile/api A就可以直接调用C中的方法。而implementation A就不能直接调用C的方法。
3.比如C中只继承okhttp，而B是封装okhttp层，A只调用B中的方法，用不到任何C中的方法，那么，B就可以implementation C,但是A必须api B。可以减少一部分编译时间，但是A不能和C中的任何类有关系，否则报错。|
|compile|api|和compile功能一模一样|
|provided|compileOnly|对所有的build type以及favlors只在编译时使用，类似eclipse中的external-libs,只参与编译，不打包到最终apk|
|apk|runtimeOnly|只会打包到apk文件中，而不参与编译|

> 3.竖线来一条？

4.再来一条[超链接？][1]

5.设置w*h 都是50的图片？ <img width="50" height=“50” src="http://pic27.nipic.com/20130305/9252150_153617685375_2.jpg"></img>

# 6.标题？
## 7.二级标题
### 8.三级标题
- 1、分行1
- 2、分行2
- 3、分行3
- 4、分行4

### 10.另一种小标题？
* 10、小标题
* 10、小标题
* 10、小标题


### 10.省事了？
1. 小标题
2. 小标题
3. 小标题

11. 分类，**加粗！**

12.类似代码块？？
```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
```

## 13、整一下文字样式
**加粗**
_斜体_
~~删除线~~
**_加粗和斜体_


## 14.来整个表格（需要注意的是，第二行中的 - 只要有三个或三个以上即可。没有具体的个数要求。）
|表头一|表头二|
|------|---|
|内容1|内容2|
|内容3|内容4|

## 15.再来整个表格
- 带左中右对其的表格
- 同样对于第二列中的空格数没有要求，但至少要有一个-。


|左对齐|居中对齐|右对齐|
|:-    |:------:|-:|
|左对齐|居中对齐|右对齐|
|1|2|3|

## 简约写法
a  | b | c  
:-:|:- |-:
    居中    |     左对齐      |   右对齐    
============|=================|=============

16、代码块着色
```java
var num = 0;
for (var i = 0; i < 5; i++) {
    num+=i;
}
console.log(num);
```

## 17、保持排版不变
<pre>
hello world 
         hi
  hello world 
</pre>

## 18、字体颜色来一个

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=12 face="黑体">黑体</font>
<font color=#00ffff size=3>null</font>
<font color=gray size=5>gray</font>

<font color=#A52A2A size=4 >Markdwon测试</font>

<font color=#FF0000>  字体改成红色了 </font>   


<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=7 face="黑体">color=#0099ff size=72 face="黑体"</font>
<font color=#00ffff size=72>color=#00ffff</font>
<font color=gray size=72>color=gray</font>


![linear](http://pic27.nipic.com/20130305/9252150_153617685375_2.jpg)





[1]:https://www.baidu.com
