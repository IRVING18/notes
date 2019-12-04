# Rxjava

> RxJava 是一个 基于事件流、实现异步操作的库

> 1、创建操作符
 - create()

 - just() 快速创建Observable ：传递数量 < 10
 - fromArray() 将数组[] 快速创建被观察者：Observable
 - fromIterable() 将List 快速创建被观察者：Observable

 - empty()
 - error()
 - never()

 - defer()
 - timer()
 - interval()
 - intervarRange()
 - range()
 - rangeLong()

> 2、额外
```java
    // 下列方法一般用于测试使用

    <-- empty()  -->
    // 该方法创建的被观察者对象发送事件的特点：仅发送Complete事件，直接通知完成
    Observable observable1=Observable.empty(); 
    // 即观察者接收后会直接调用onCompleted（）

    <-- error()  -->
    // 该方法创建的被观察者对象发送事件的特点：仅发送Error事件，直接通知异常
    // 可自定义异常
    Observable observable2=Observable.error(new RuntimeException())
    // 即观察者接收后会直接调用onError（）

    <-- never()  -->
    // 该方法创建的被观察者对象发送事件的特点：不发送任何事件
    Observable observable3=Observable.never();
    // 即观察者接收后什么都不调用
```