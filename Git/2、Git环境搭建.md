# 一、gitlab ssh 秘钥配置
#### 1、在本地生成rsa秘钥，默认生成到.ssh目录下
```shell
    ssh-keygen -t rsa -C "your_email@example.com"
```
#### 2、粘贴 .ssh/id_rsa.pub 文字到gitlab上
点头像-> Setting -> SSH Kes 

# 二、设置linux 命令高亮主题，.oh-my-zsh
#### 1、查看当前环境shell
```shell
   echo $SHELL
```
#### 2、查看系统自带哪些shell
```shell
   cat /etc/shells
```
> 如果没有/bin/zsh的话，安装zsh
 ```shell 
    yum install zsh # centos
    brew install zsh # mac
 ```
> 如果有/bin/zsh 直接设置成默认,设置完重启终端
```shell
    chsh -s /bin/zsh
```
#### 3、安装oh-my-zsh
 ```shell
    git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
    cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
 ```
 
#### 4、修改主题，vim ~/.zshrc
```java
    ZSH_THEME="ys"
```
#### 5、安装自动补全插件 zsh-autosuggestions
```java
    cd ~/.oh-my-zsh/custom/plugins/
    git clone https://github.com/zsh-users/zsh-autosuggestions
    vi ~/.zshrc
    
    加上 
    vi ~/.zshrc
```
