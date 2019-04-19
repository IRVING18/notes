# Rxjava

> 1、Single

## 一、rxjava 操作符

- 1、map ：转换数据类型。 
```java
Single.just(1)
    //将integer转成string，给singleObserver用
    .map(new Function<Integer, String>() {
                    @Override
                    public String apply(Integer integer) {
                        return String.valueOf(integer);
                    }
                })
                .subscribe(new SingleObserver<String>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onSuccess(String s) {
                        textView.setText(s);
                    }

                    @Override
                    public void onError(Throwable e) {

                    }
                });
```
- 2、

## 