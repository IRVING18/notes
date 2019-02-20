# 一、gradle 3.0 升级变化

|已弃用配置|新配置|行为|
|:------:  |:------:|:-|
|compile|implementation|1、可以缩短编译模块的时间。<br>2、不能传递依赖，比如A依赖B，B依赖C。使用compile/api A就可以直接调用C中的方法。而implementation A就不能直接调用C的方法。<br>3、比如C中只继承okhttp，而B是封装okhttp层，A只调用B中的方法，用不到任何C中的方法，那么，B就可以implementation C,但是A必须api B。可以减少一部分编译时间，但是A不能和C中的任何类有关系，否则报错。|
|compile|api|和compile功能一模一样|
|provided|compileOnly|对所有的build type以及favlors只在编译时使用，类似eclipse中的external-libs,只参与编译，不打包到最终apk|
|apk|runtimeOnly|只会打包到apk文件中，而不参与编译|
|apt|annotationProcessor|注解处理器依赖项配置，目前看来没有依赖传递，必须要在所有build.gradle中加入配置|

# 二、flavorDimensions 风味维度
> 可以定义多个flavorDimensions 维度

```java
  flavorDimensions 'price','nation'
  productFlavors {
      //价格维度
      free {
          dimension 'price'
      }
      paid {
          dimension 'price'
      }
      //国际维度
      china {
          dimension 'nation'
      }
      global {
          dimension 'nation'
      }
  }
```

# 三、构建变体规则
> 可以自定识别debug、release等不同的代码块

#### 1、设置buildType

```java
  buildTypes {
      //调试模式
      debug {
          minifyEnabled false
          //是否zip对齐
          zipAlignEnabled true
          //移除无用的资源文件
          shrinkResources false
      }
      //测试模块
      detest {
          initWith debug
      }
      //预发环境
      beta {
          initWith debug
      }
      //正式环境
      release {
          minifyEnabled false
          //是否zip对齐
          zipAlignEnabled true
          //移除无用的资源文件
          shrinkResources false
      }
  }
```

#### 2、建立debug，detest文件夹可以设置不同环境的代码实现

例如: 
1. src/chinaDebug/（构建变体源集） 
2. src/debug/（构建类型源集） 
3. src/detest/（产品风味源集） 
4. src/main/（主源集）



