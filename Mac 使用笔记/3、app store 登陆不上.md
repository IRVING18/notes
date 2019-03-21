# 注册的apple id 死活登陆不上app store

### 1、注册完appleID 先登陆iTunes。
> 这时可能连iTunes都登陆不上，这个尿性啊apple。   
> 如果真的不幸你就是那个登不上的，那你只能去找个iPhone手机登陆iTunes了。    
> 登陆上之后需要填写一堆没用的信息。
### 2、再来登陆APP store 
> 这时你会不断重复，账号密码输入完，diu~diu~diu~  转几圈，然后啥反应也没有了。    
> 咳咳~文明点。   
> 这时牛逼的来了。
### 3、滚蛋吧APPLE 
```java
  defaults write com.apple.appstore.commerce Storefront -string "$(defaults read com.apple.appstore.commerce Storefront | sed s/,8/,13/)"
```
