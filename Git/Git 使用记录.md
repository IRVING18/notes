## 提交：
- git add 文件名
- git commit -m"备注"
- git pull origin develop --rebase
- git push origin develop

## Merge:
- 1、git pull origin develop --rebase
- 2、（去文件中解决冲突）
- 3、git add -u 
- 4、git rebase --continue  再次merge  
	或  
	git rebase --abort  还原到本地代码  
- 5、git push origin develop

- 还原本地最后一次commit 
- git checkout
- 还原远程git版本：
- git log

- git reset --hard 版本号

- git push orgin :develop

- git push origin develop

## 创建分支
- git checkout -b 分支名

## 切换到远程分支
- git checkout -t origin/feature/develop

## 删除分支
- git branch -d 分支名  //删除本地分支
- git push origin --delete 分支名   //删除远程分支

## 关联远程仓库
- git init
- git config --global user.name "git用户名"
- git config --global user.email 邮箱
- git add .
- git commit -m""
- git remote add origin 链接
- git push 

## 合并分支：
- git checkout develop 切换到develop分支
- git merge master  再develop分支合并master，如果有冲突解决完
- git checkout master 切换到master分支
- git merge develop 再master mergedevelop分支，
- git push origin master  没有问题就提交到mester

## 删除分支
- git branch -D wangzheng 删除本地wangzheng分支
- git push origin :wangzheng 删除远程wangzheng分支

## 创建tag
- git tag -a v1.2.2 //创建tag
- git push origin --tags //把本地tag都推到远程

- git tag -d v1.2.2 //删除本地tag
- git push origin :refs/tags/v1.2.2 //删除远程tag

## 回到某个tag
- git checkout tag1 //回到那次tag
- git checkout -b tag1 //如果需要可以直接创建新的分支，代码就是这个tag

## 回滚、撤销回滚
> 前提是commit都已经push过  

git reset commit号  
git checkout -- .  
等于 =  
git reset --hard commit号  
  
撤销回滚   
git reflog -> 找到commit号  
git reset --hard commit 号   


# 更换远程地址
## 1、切换远程仓库地址
- 方式一：修改远程仓库地址  

【git remote set-url origin URL】 更换远程仓库地址，URL为新地址。

- 方式二：先删除远程仓库地址，然后再添加  

【git remote rm origin】 删除现有远程仓库 
【git remote add origin url】添加新远程仓库

## 2、查看远程仓库地址

git remote -v

## 3、初次设置远程仓库

git init

git remote add origin <你的远程地址>


# 删除git远程提上去的垃圾文件夹
git pull origin master   
git rm -r --cached <你文件夹名字路径>   
git commit -m""    
git push origin master    


# 撤销commit 操作
> 撤销commit，回到add的状态。^1上一次commit ^2前两次commit都回退

```java
git reset --solf HEAD^1
```
说一下个人理解：

HEAD^的意思是上一个版本，也可以写成HEAD~1

如果你进行了2次commit，想都撤回，可以使用HEAD~2

至于这几个参数：

--mixed 

意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
 

--soft  

不删除工作空间改动代码，撤销commit，不撤销git add . 
 
--hard

删除工作空间改动代码，撤销commit，撤销git add . 

注意完成这个操作后，就恢复到了上一次的commit状态。

# 撤销add 操作

```java
git reset
```

# 将某个文件恢复到某个commit号
```java
git reset sa6789fas src/add/controller.java
```


