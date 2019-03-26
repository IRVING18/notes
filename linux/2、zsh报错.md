# 1.1 安装zsh
```java
$ sudo apt-get install zsh
```
# 1.2 把默认的Shell改成zsh，注意：不要使用sudo
```java
$ chsh -s /bin/zsh 
```

# 1.3 配置密码文件，解决chsh：PAM认证失败的问题
```java
$ sudo vim /etc/passwd
```
## 1.3.1 把第一行的/bin/bash改成/bin/zsh，这个是root用户的
## 1.3.2 把最后一行的/bin/bash改成/bin/zsh，这个应该是每台电脑的登录用户名+计算机名组成的。

# 2.1 安装oh-my-zsh
```java
$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
# 2.2 重启电脑
```java
$ sudo reboot
```
# 2.3 查看效果
```java
$ LANG
```
# 3. 自动补全插件
## 3.1 下载
```java
$ wget http://mimosa-pudica.net/src/incr-0.2.zsh
```
## 3.2 将此插件放到oh-my-zsh目录的插件库下：
```java
$ sudo mkdir ~/.oh-my-zsh/plugins/incr/
$ sudo mv incr-0.2.zsh ~/.oh-my-zsh/plugins/incr/incr-0.2.zsh
$ sudo vim ~/.zshrc 
```
## 3.3 末尾加上如下代码：
source ~/.oh-my-zsh/plugins/incr/incr*.zsh

## 3.4 更新配置
```java
source ~/.zshrc
```

# 4. 与vim的提示相冲突的解决方案
## 4.1 使用自动补全插件可能会与vim的提示功能相冲突，如会报以下错误：
```java
vim t
_arguments:451: _vim_files: function definition file not found
_alternative:69: __git_remotes: function definition file not found
```
## 4.2 解决方法：将~/.zcompdump*删除即可
```java
$ rm -rf ~/.zcompdump*
$ exec zsh
```
