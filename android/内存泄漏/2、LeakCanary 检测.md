# LeakCanary

[LeadkCanary 官网地址](https://github.com/square/leakcanary)

## 最简单使用

> 这两步就可以最简单的使用，打开APP如果有内存泄漏的情况发生，Leak就会提示出来了。
### 1、在`build.gradle`中添加依赖:

```groovy
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.6.3'
  releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.6.3'
  // Optional, if you use support library fragments:
  debugImplementation 'com.squareup.leakcanary:leakcanary-support-fragment:1.6.3'
}
```

### 2、在`Application` 中注册LeakCanary:

```java
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    
    LeakCanary.install(this);

  }
}
```

## 使用RefWatcher 监听Fragment中的内存泄漏情况

### 1、在Application中封装
```java
 public static RefWatcher getRefWatcher(Context context) {
    Application application = (Application) context.getApplicationContext();
    return application.refWatcher;
  }
```
### 2、在Fragment中加入

```java
public abstract class BaseFragment extends Fragment {

  @Override public void onDestroy() {
    super.onDestroy();
    RefWatcher refWatcher = ExampleApplication.getRefWatcher(getActivity());
    refWatcher.watch(this);
  }
}
```