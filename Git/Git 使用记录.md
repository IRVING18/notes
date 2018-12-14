**提交：
- git add 文件名
- git commit -m"备注"
- git pull origin develop --rebase
- git push origin develop

**Merge:
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

**创建分支
- git checkout -b 分支名

**关联远程仓库
- git init
- git config --global user.name "git用户名"
- git config --global user.email 邮箱
- git add .
- git commit -m""
- git remote add origin 链接
- git push 

**合并分支：
- git checkout develop 切换到develop分支
- git merge master  再develop分支合并master，如果有冲突解决完
- git checkout master 切换到master分支
- git merge develop 再master mergedevelop分支，
- git push origin master  没有问题就提交到mester

**删除分支
- git branch -D wangzheng 删除本地wangzheng分支
- git push origin :wangzheng 删除远程wangzheng分支

**创建tag
- git tag -a v1.2.2 //创建tag
- git push origin --tags //把本地tag都推到远程

- git tag -d v1.2.2 //删除本地tag
- git push origin :refs/tags/v1.2.2 //删除远程tag
